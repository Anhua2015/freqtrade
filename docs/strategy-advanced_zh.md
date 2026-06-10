# 高级策略

本页介绍策略中可用的一些高级概念。
如果你是新手，请先熟悉 [Freqtrade 基础](bot-basics.md)和[策略定制](strategy-customization.md)中描述的方法。

!!! Tip
    通过运行 `freqtrade new-strategy --strategy MyAwesomeStrategy --template advanced`，从包含所有可用回调方法的策略模板开始。

## 存储信息（持久化）

Freqtrade 允许在数据库中存储/检索与特定交易关联的用户自定义信息。

使用交易对象，可以通过 `trade.set_custom_data(key='my_key', value=my_value)` 存储信息，并通过 `trade.get_custom_data(key='my_key')` 检索。

```python
from freqtrade.persistence import Trade
from datetime import timedelta

class AwesomeStrategy(IStrategy):
    def bot_loop_start(self, **kwargs) -> None:
        for trade in Trade.get_open_order_trades():
            fills = trade.select_filled_orders(trade.entry_side)
            if trade.pair == 'ETH/USDT':
                trade_entry_type = trade.get_custom_data(key='entry_type')
                if trade_entry_type is None:
                    trade_entry_type = 'breakout' if 'entry_1' in trade.enter_tag else 'dip'
                elif len(fills) > 1:
                    trade_entry_type = 'buy_up'
                trade.set_custom_data(key='entry_type', value=trade_entry_type)
        return super().bot_loop_start(**kwargs)
```

!!! Note
    建议使用简单数据类型 `[bool, int, float, str]` 以确保序列化数据时不会出现问题。

## 入场标签（Enter Tag）

当你的策略有多个入场信号时，你可以命名触发的信号，然后在 `custom_exit` 中访问入场信号。

```python
def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe["enter_tag"] = ""
    signal_rsi = (qtpylib.crossed_above(dataframe["rsi"], 35))
    signal_bblower = (dataframe["bb_lowerband"] < dataframe["close"])
    dataframe.loc[
        (signal_rsi | signal_bblower) & (dataframe["volume"] > 0), "enter_long"
    ] = 1
    # 连接标签以保留所有信号
    dataframe.loc[signal_rsi, "enter_tag"] += "long_signal_rsi "
    dataframe.loc[signal_bblower, "enter_tag"] += "long_signal_bblower "
    return dataframe
```

!!! Note
    `enter_tag` 限制为 255 个字符，超出部分将被截断。

## 出场标签（Exit Tag）

类似入场标签，你也可以指定出场标签。提供的出场标签随后用作出场原因。

## 策略版本

你可以通过实现 `version` 方法来实现自定义策略版本控制。

``` python
def version(self) -> str:
    return "1.1"
```

## 派生策略

策略可以从其他策略派生。这避免了策略代码的重复。

``` python title="user_data/strategies/MyAwesomeStrategy2.py"
from myawesomestrategy import MyAwesomeStrategy
class MyAwesomeStrategy2(MyAwesomeStrategy):
    # 覆盖某些内容
    stoploss = 0.08
    trailing_stop = True
```

## 数据框访问

你可以在各种策略函数中通过从 dataprovider 查询来访问数据框。

``` python
from freqtrade.exchange import timeframe_to_prev_date

class AwesomeStrategy(IStrategy):
    def confirm_trade_exit(self, pair: str, trade: 'Trade', ...):
        dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
        last_candle = dataframe.iloc[-1].squeeze()
        trade_date = timeframe_to_prev_date(self.timeframe, trade.open_date_utc)
        trade_candle = dataframe.loc[dataframe['date'] == trade_date]
```

## 性能警告

执行策略时，可能会在日志中看到：

> PerformanceWarning: DataFrame is highly fragmented.

这是来自 `pandas` 的警告。建议使用 `pd.concat(axis=1)`。

例如：

```python
frames = [dataframe]
for val in self.buy_ema_short.range:
    frames.append(DataFrame({
        f'ema_short_{val}': ta.EMA(dataframe, timeperiod=val)
    }))
dataframe = pd.concat(frames, axis=1)
```