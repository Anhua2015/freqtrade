# 强化学习

!!! Note "安装大小"
    强化学习依赖包括如 `torch` 这样的大包，应在 `./setup.sh -i` 期间对问题 "Do you also want dependencies for freqai-rl (~700mb additional space required) [y/N]?" 回答 "y" 来显式请求。
    偏好 docker 的用户应确保使用带有 `_freqairl` 后缀的 docker 镜像。

## 背景和术语

### 什么是 RL，为什么 FreqAI 需要它？

强化学习涉及两个重要组件：*智能体（agent）*和训练*环境（environment）*。在智能体训练期间，智能体逐根 K 线遍历历史数据，始终从一组动作中做出 1 个选择：多头入场、多头出场、空头入场、空头出场、中性。在此训练过程中，环境跟踪这些动作的表现，并根据用户自定义的 `calculate_reward()` 奖励智能体（此处我们提供了一个默认奖励供用户在此基础上构建，[详情见此处](#creating-a-custom-reward-function)）。奖励用于训练神经网络中的权重。

FreqAI RL 实现的第二个重要组件是使用*状态（state）*信息。状态信息在每一步被输入网络，包括当前利润、当前仓位和当前交易持续时间。这些用于在训练环境中训练智能体，并在模拟/实盘中强化智能体（此功能在回测中不可用）。*FreqAI + Freqtrade 是这种强化机制的完美匹配，因为这些信息在实盘部署中随手可得。*

强化学习是 FreqAI 的自然演进，因为它添加了分类器和回归器无法匹配的新一层自适应性和市场反应能力。然而，分类器和回归器具有 RL 所没有的优势，如稳健的预测。训练不当的 RL 智能体可能会找到"作弊"和"技巧"来最大化奖励，而实际上没有赢得任何交易。因此，RL 更复杂，需要比典型的分类器和回归器更高层次的理解。

### 配置强化学习器

为了配置 `Reinforcement Learner`，以下字典必须存在于 `freqai` 配置中：

```json
        "rl_config": {
            "train_cycles": 25,
            "add_state_info": true,
            "max_trade_duration_candles": 300,
            "max_training_drawdown_pct": 0.02,
            "cpu_count": 8,
            "model_type": "PPO",
            "policy_type": "MlpPolicy",
            "model_reward_parameters": {
                "rr": 1,
                "profit_aim": 0.025
            }
        }
```

参数详情见[此处](freqai-parameter-table.md)，但一般来说 `train_cycles` 决定智能体应在其人工环境中循环遍历 K 线数据多少次来训练模型中的权重。`model_type` 是一个字符串，选择 [stable_baselines](https://stable-baselines3.readthedocs.io/en/master/)（外部链接）中可用的模型之一。

### 重要注意事项

RL 训练环境比真正的 Freqtrade 回测环境简化得多。它不包含任何复杂的策略逻辑，如 `custom_exit`、`custom_stoploss`、杠杆控制等回调。RL 环境是真实市场的一个非常"原始"的表示，智能体有自由学习由 `calculate_reward()` 强制执行的政策（即止损、止盈等）。因此，重要的是要考虑智能体训练环境与真实世界并不相同。

## 运行强化学习

设置和运行强化学习模型与运行回归器或分类器相同。必须在命令行上定义相同的两个标志 `--freqaimodel` 和 `--strategy`：

```bash
freqtrade trade --freqaimodel ReinforcementLearner --strategy MyRLStrategy --config config.json
```

强化学习不需要设置目标标签。相反，FreqAI 要求在动作列中设置一个默认（中性）值：

```python
    def set_freqai_targets(self, dataframe, **kwargs) -> DataFrame:
        # 对于 RL，没有直接的目标要设置。这是填充值（中性）
        # 直到智能体发送动作。
        dataframe["&-action"] = 0
        return dataframe
```

策略必须将原始价格数据传递给智能体，以便它在训练环境中可以访问原始 OHLCV：

```python
    def feature_engineering_standard(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        # 以下特征对 RL 模型是必需的
        dataframe[f"%-raw_close"] = dataframe["close"]
        dataframe[f"%-raw_open"] = dataframe["open"]
        dataframe[f"%-raw_high"] = dataframe["high"]
        dataframe[f"%-raw_low"] = dataframe["low"]
    return dataframe
```

## 创建自定义奖励函数

!!! danger "非生产用途"
    警告！
    Freqtrade 源代码中提供的奖励函数是功能展示，设计用于展示/测试尽可能多的环境控制功能。它也被设计为在小型计算机上快速运行。这是一个基准，*不*适用于实盘生产。请注意，你将需要创建自己的 custom_reward() 函数或使用 Freqtrade 源代码之外的其他用户构建的模板。

在 `MyRLEnv` 类中设置 `calculate_reward()` 函数。一个默认的 `calculate_reward()` 在 `prediction_models/ReinforcementLearner.py` 中提供，用于演示创建奖励的必要构建块，但这*不*是为生产设计的。

```python
from freqtrade.freqai.prediction_models.ReinforcementLearner import ReinforcementLearner
from freqtrade.freqai.RL.Base5ActionRLEnv import Actions, Base5ActionRLEnv, Positions

class MyCoolRLModel(ReinforcementLearner):
    class MyRLEnv(Base5ActionRLEnv):
        def calculate_reward(self, action: int) -> float:
            if not self._is_valid(action):
                return -2
            pnl = self.get_unrealized_profit()
            factor = 100
            # 奖励智能体入场
            if (action in (Actions.Long_enter.value, Actions.Short_enter.value)
                    and self._position == Positions.Neutral):
                return 25 * factor
            # 阻止智能体不入场
            if action == Actions.Neutral.value and self._position == Positions.Neutral:
                return -1
            # 平多头
            if action == Actions.Long_exit.value and self._position == Positions.Long:
                if pnl > self.profit_aim * self.rr:
                    factor *= 2
                return float(pnl * factor)
            # 平空头
            if action == Actions.Short_exit.value and self._position == Positions.Short:
                if pnl > self.profit_aim * self.rr:
                    factor *= 2
                return float(pnl * factor)
            return 0.
```

## 使用 Tensorboard

强化学习模型受益于跟踪训练指标。FreqAI 已集成 Tensorboard，允许用户跨所有币种和所有重新训练跟踪训练和评估性能。

```bash
tensorboard --logdir user_data/models/unique-id
```

## 选择基础环境

FreqAI 提供三个基础环境：`Base3ActionRLEnvironment`、`Base4ActionEnvironment` 和 `Base5ActionEnvironment`。

- `Base3ActionEnvironment`：最简单，智能体可以从持有、多头或空头中选择。也可用于仅多头的机器人。
- `Base4ActionEnvironment`：智能体可以多头入场、空头入场、持有中性或平仓。
- `Base5ActionEnvironment`：与 Base4 相同，但将平多头和平空头分开。

!!! Note
    只有 `Base3ActionRLEnv` 可以进行仅多头训练/交易（设置策略属性 `can_short = False`）。