# 超参优化

本页介绍如何通过寻找最优参数来调优你的策略，这个过程称为超参数优化。机器人使用 `optuna` 包中的算法来完成此任务。

超参优化需要历史数据可用，就像回测一样（超参优化用不同参数多次运行回测）。

## 安装超参优化依赖

```bash
source .venv/bin/activate
pip install -r requirements-hyperopt.txt
```

## 超参优化命令参考

--8<-- "commands/hyperopt.md"

## 超参优化执行逻辑

超参优化首先将数据加载到内存中，然后对每个交易对运行一次 `populate_indicators()` 生成所有指标。

然后超参优化会派生到不同的进程中，反复运行回测，更改属于定义的 `--spaces` 的参数。

对于每一组新参数，freqtrade 将首先运行 `populate_entry_trend()`，然后运行 `populate_exit_trend()`，然后运行常规的回测过程来模拟交易。回测后，结果传入[损失函数](#loss-functions)，评估此结果是否比之前的结果更好或更差。基于损失函数的结果，超参优化将确定下一轮回测中要尝试的下一组参数。

## 参数类型

* `IntParameter` - 定义具有搜索空间上下边界的整数参数。
* `DecimalParameter` - 定义具有有限小数位数的浮点参数（默认 3 位）。
* `RealParameter` - 定义具有上下边界但没有精度限制的浮点参数。很少使用。
* `CategoricalParameter` - 定义具有预定数量选择的参数。
* `BooleanParameter` - `CategoricalParameter([True, False])` 的简写。

### 参数选项

* `optimize` - 设为 `False` 时，该参数不会被包含在优化过程中（默认：True）。
* `load` - 设为 `False` 时，之前超参优化运行的结果不会被用作后续超参优化的起始值（默认：True）。

### 定义可超参优化的参数

```python
class MyAwesomeStrategy(IStrategy):
    buy_adx = DecimalParameter(20, 40, decimals=1, default=30.1, space="buy")
    buy_rsi = IntParameter(20, 40, default=30, space="buy")
    buy_adx_enabled = BooleanParameter(default=True, space="buy")
    buy_rsi_enabled = CategoricalParameter([True, False], default=False, space="buy")
    buy_trigger = CategoricalParameter(["bb_lower", "macd_cross_signal"], default="bb_lower", space="buy")
```

在 `populate_entry_trend()` 中使用这些参数：

```python
    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        conditions = []
        if self.buy_adx_enabled.value:
            conditions.append(dataframe['adx'] > self.buy_adx.value)
        if self.buy_rsi_enabled.value:
            conditions.append(dataframe['rsi'] < self.buy_rsi.value)

        if self.buy_trigger.value == 'bb_lower':
            conditions.append(dataframe['close'] < dataframe['bb_lowerband'])
        if self.buy_trigger.value == 'macd_cross_signal':
            conditions.append(qtpylib.crossed_above(
                dataframe['macd'], dataframe['macdsignal']
            ))

        conditions.append(dataframe['volume'] > 0)

        if conditions:
            dataframe.loc[
                reduce(lambda x, y: x & y, conditions),
                'enter_long'] = 1
        return dataframe
```

## 损失函数

损失函数是超参优化的关键部分。它评估回测结果的质量。Freqtrade 内置了多个损失函数：

* `SharpeHyperOptLoss` - 基于夏普比率
* `SharpeHyperOptLossDaily` - 基于每日夏普比率
* `SortinoHyperOptLoss` - 基于索提诺比率
* `SortinoHyperOptLossDaily` - 基于每日索提诺比率
* `MaxDrawDownHyperOptLoss` - 基于最大回撤
* `CalmarHyperOptLoss` - 基于卡尔玛比率
* `ProfitDrawDownHyperOptLoss` - 基于利润和回撤
* `WinRatioHyperOptLoss` - 基于胜率
* `OnlyProfitHyperOptLoss` - 仅基于利润

## 运行超参优化

```bash
freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --strategy MyAwesomeStrategy --config config.json -e 500
```

## 可复现结果

超参优化使用随机组合开始搜索。为了使结果可复现，可以使用 `--random-state` 参数设置随机种子。

## 超参优化技巧

- 建议反复运行 500-1000 轮，直到达到至少 10000 轮
- 使用 `--spaces` 参数限制优化的空间以加快速度
- 快速优化 ROI、止损和跟踪止损：`--spaces roi stoploss trailing`
- `populate_indicators` 需要创建任何空间可能使用的所有指标