# 高级超参优化

本页解释了一些高级超参优化主题，这些可能需要比创建普通超参优化类更高的编码技能和 Python 知识。

## 创建和使用自定义损失函数

要使用自定义损失函数类，请确保在你的自定义超参损失类中定义了 `hyperopt_loss_function` 函数。
对于下面的示例，你需要将命令行参数 `--hyperopt-loss SuperDuperHyperOptLoss` 添加到超参优化调用中，以使用此函数。

下面是一个示例，与默认超参优化损失实现相同。完整示例可在 [userdata/hyperopts](https://github.com/freqtrade/freqtrade/blob/develop/freqtrade/templates/sample_hyperopt_loss.py) 中找到。

``` python
from datetime import datetime
from typing import Any, Dict

from pandas import DataFrame

from freqtrade.constants import Config
from freqtrade.optimize.hyperopt import IHyperOptLoss

TARGET_TRADES = 600
EXPECTED_MAX_PROFIT = 3.0
MAX_ACCEPTED_TRADE_DURATION = 300

class SuperDuperHyperOptLoss(IHyperOptLoss):
    """
    定义超参优化的默认损失函数
    """

    @staticmethod
    def hyperopt_loss_function(
        *,
        results: DataFrame,
        trade_count: int,
        min_date: datetime,
        max_date: datetime,
        config: Config,
        processed: dict[str, DataFrame],
        backtest_stats: dict[str, Any],
        starting_balance: float,
        **kwargs,
    ) -> float:
        """
        目标函数，返回更小的数字表示更好的结果
        这是传统算法（freqtrade 至今仍在使用）。
        权重分配如下：
        * 0.4 给交易持续时间
        * 0.25：避免交易亏损
        * 1.0 给总利润，与上述定义的期望值 (`EXPECTED_MAX_PROFIT`) 比较
        """
        total_profit = results['profit_ratio'].sum()
        trade_duration = results['trade_duration'].mean()

        trade_loss = 1 - 0.25 * exp(-(trade_count - TARGET_TRADES) ** 2 / 10 ** 5.8)
        profit_loss = max(0, 1 - total_profit / EXPECTED_MAX_PROFIT)
        duration_loss = 0.4 * min(trade_duration / MAX_ACCEPTED_TRADE_DURATION, 1)
        result = trade_loss + profit_loss + duration_loss
        return result
```

目前，参数包括：

* `results`：包含结果交易的 DataFrame。
    results 中可用的列如下（对应于使用 `--export trades` 时回测的输出文件）：  
    `pair, profit_ratio, profit_abs, open_date, open_rate, fee_open, close_date, close_rate, fee_close, amount, trade_duration, is_open, exit_reason, stake_amount, min_rate, max_rate, stop_loss_ratio, stop_loss_abs`
* `trade_count`：交易数量（等同于 `len(results)`）
* `min_date`：使用的时间范围的开始日期
* `min_date`：使用的时间范围的结束日期
* `config`：使用的 Config 对象（注意：如果某些参数是超参优化空间的一部分，则并非所有策略相关参数都会在此更新）。
* `processed`：以交易对为键的 DataFrame 字典，包含用于回测的数据。
* `backtest_stats`：使用与回测文件 "strategy" 子结构相同格式的回测统计数据。可用字段可在 `optimize_reports.py` 中的 `generate_strategy_stats()` 中查看。
* `starting_balance`：用于回测的起始余额。

此函数需要返回一个浮点数（`float`）。更小的数字将被解释为更好的结果。参数和平衡由你决定。

!!! Note
    此函数每个 epoch 调用一次 — 所以请确保尽可能优化，以免不必要地减慢超参优化速度。

!!! Note "`*args` 和 `**kwargs`"
    请在接口中保留 `*args` 和 `**kwargs` 参数，以便我们将来扩展此接口。

## 覆盖预定义空间

要覆盖预定义空间（`roi_space`、`generate_roi_table`、`stoploss_space`、`trailing_space`、`max_open_trades_space`），定义一个名为 Hyperopt 的嵌套类并按如下方式定义所需空间：

```python
from freqtrade.optimize.space import Categorical, Dimension, Integer, SKDecimal

class MyAwesomeStrategy(IStrategy):
    class HyperOpt:
        # 定义自定义止损空间。
        def stoploss_space():
            return [SKDecimal(-0.05, -0.01, decimals=3, name='stoploss')]

        # 定义自定义 ROI 空间
        def roi_space() -> List[Dimension]:
            return [
                Integer(10, 120, name='roi_t1'),
                Integer(10, 60, name='roi_t2'),
                Integer(10, 40, name='roi_t3'),
                SKDecimal(0.01, 0.04, decimals=3, name='roi_p1'),
                SKDecimal(0.01, 0.07, decimals=3, name='roi_p2'),
                SKDecimal(0.01, 0.20, decimals=3, name='roi_p3'),
            ]

        def generate_roi_table(params: Dict) -> dict[int, float]:

            roi_table = {}
            roi_table[0] = params['roi_p1'] + params['roi_p2'] + params['roi_p3']
            roi_table[params['roi_t3']] = params['roi_p1'] + params['roi_p2']
            roi_table[params['roi_t3'] + params['roi_t2']] = params['roi_p1']
            roi_table[params['roi_t3'] + params['roi_t2'] + params['roi_t1']] = 0

            return roi_table

        def trailing_space() -> List[Dimension]:
            # 这里所有参数都是必需的，你只能修改它们的类型或范围。
            return [
                # 固定为 true，如果优化 trailing_stop，则假定始终使用 trailing stop。
                Categorical([True], name='trailing_stop'),

                SKDecimal(0.01, 0.35, decimals=3, name='trailing_stop_positive'),
                # 'trailing_stop_positive_offset' 应大于 'trailing_stop_positive'，
                # 因此此中间参数用作两者差值。
                # 'trailing_stop_positive_offset' 的值在 generate_trailing_params() 方法中构造。
                # 这类似于用于构造 ROI 表的超空间维度。
                SKDecimal(0.001, 0.1, decimals=3, name='trailing_stop_positive_offset_p1'),

                Categorical([True, False], name='trailing_only_offset_is_reached'),
        ]

        # 定义自定义 max_open_trades 空间
        def max_open_trades_space() -> List[Dimension]:
            return [
                Integer(-1, 10, name='max_open_trades'),
            ]
```

!!! Note
    所有覆盖都是可选的，可以根据需要混合/匹配。

## 动态参数

参数也可以动态定义，但必须在 [`bot_start()` 回调](strategy-callbacks.md#bot-start) 被调用后对实例可用。

``` python

class MyAwesomeStrategy(IStrategy):

    def bot_start(self, **kwargs) -> None:
        self.buy_adx = IntParameter(20, 30, default=30, optimize=True)

    # ...
```

!!! Warning
    以这种方式创建的参数不会显示在 `list-strategies` 参数计数中。

## 覆盖基础估计器

你可以通过在 Hyperopt 子类中实现 `generate_estimator()` 来为超参优化定义自己的 optuna 采样器。

```python
class MyAwesomeStrategy(IStrategy):
    class HyperOpt:
        def generate_estimator(dimensions: List['Dimension'], **kwargs):
            return "NSGAIIISampler"

```

可能的值包括 `"NSGAIISampler"`、`"TPESampler"`、`"GPSampler"`、`"CmaEsSampler"`、`"NSGAIIISampler"`、`"QMCSampler"`（详细信息请参阅 [optuna-samplers 文档](https://optuna.readthedocs.io/en/stable/reference/samplers/index.html)），或者"继承自 `optuna.samplers.BaseSampler` 的类的实例"。

可能需要一些研究来找到其他采样器（例如来自 optunahub）。

!!! Note
    虽然可以提供自定义估计器，但作为用户，你有责任研究可能的参数并分析/理解应使用哪些参数。
    如果你对此不确定，最好使用默认值之一（`"NSGAIIISampler"` 已被证明是最通用的）而不带额外参数。

??? Example "使用来自 Optunahub 的 `AutoSampler`"

    [AutoSampler 文档](https://hub.optuna.org/samplers/auto_sampler/)
    
    安装必要的依赖：
    ``` bash
    pip install optunahub cmaes torch scipy
    ```
    在策略中实现 `generate_estimator()`：

    ``` python
    # ...
    from freqtrade.strategy.interface import IStrategy
    from typing import List
    import optunahub
    # ... 

    class my_strategy(IStrategy):
        class HyperOpt:
            def generate_estimator(dimensions: List["Dimension"], **kwargs):
                if "random_state" in kwargs.keys():
                    return optunahub.load_module("samplers/auto_sampler").AutoSampler(seed=kwargs["random_state"])
                else:
                    return optunahub.load_module("samplers/auto_sampler").AutoSampler()

    ```

    显然，同样的方法适用于 optuna 支持的所有其他采样器。

## 空间选项

对于额外的空间，scikit-optimize（与 Freqtrade 结合）提供以下空间类型：

* `Categorical` - 从类别列表中选择（例如 `Categorical(['a', 'b', 'c'], name="cat")`）
* `Integer` - 从整数范围中选择（例如 `Integer(1, 10, name='rsi')`）
* `SKDecimal` - 从有限精度的小数范围中选择（例如 `SKDecimal(0.1, 0.5, decimals=3, name='adx')`）。*仅适用于 freqtrade*。
* `Real` - 从全精度的小数范围中选择（例如 `Real(0.1, 0.5, name='adx')`）

你可以从 `freqtrade.optimize.space` 导入所有这些，尽管 `Categorical`、`Integer` 和 `Real` 只是相应 scikit-optimize Space 的别名。`SKDecimal` 由 freqtrade 提供，用于更快的优化。

``` python
from freqtrade.optimize.space import Categorical, Dimension, Integer, SKDecimal, Real  # noqa
```

!!! Hint "SKDecimal 与 Real"
    我们建议在几乎所有情况下使用 `SKDecimal` 而不是 `Real` 空间。虽然 Real 空间提供完全精度（最多约 16 位小数）—— 这种精度很少需要，并且会导致不必要的长时间超参优化。

    假设定义了一个相当小的空间（`SKDecimal(0.10, 0.15, decimals=2, name='xxx')`）— SKDecimal 将有 5 种可能性（`[0.10, 0.11, 0.12, 0.13, 0.14, 0.15]`）。

    另一方面，相应的 Real 空间 `Real(0.10, 0.15 name='xxx')` 具有几乎无限的可能性（`[0.10, 0.010000000001, 0.010000000002, ... 0.014999999999, 0.01500000000]`）。