# 策略回调

主要的策略函数（`populate_indicators()`、`populate_entry_trend()`、`populate_exit_trend()`）应以向量化方式使用，回调则在"需要时"被调用。

## 当前可用的回调

* [`bot_start()`](#bot-start) - 策略加载时调用一次
* [`bot_loop_start()`](#bot-loop-start) - 每次机器人迭代开始时调用
* [`custom_stake_amount()`](#stake-size-management) - 投入金额管理
* [`custom_exit()`](#custom-exit-signal) - 自定义出场信号
* [`custom_stoploss()`](#custom-stoploss) - 自定义止损
* [`custom_roi()`](#custom-roi) - 自定义 ROI
* [`custom_entry_price()` 和 `custom_exit_price()`](#custom-order-price-rules) - 自定义订单价格
* [`check_entry_timeout()` 和 `check_exit_timeout()`](#custom-order-timeout-rules) - 自定义订单超时
* [`confirm_trade_entry()`](#trade-entry-buy-order-confirmation) - 确认交易入场
* [`confirm_trade_exit()`](#trade-exit-sell-order-confirmation) - 确认交易出场
* [`adjust_trade_position()`](#adjust-trade-position) - 调整交易仓位
* [`adjust_entry_price()`](#adjust-entry-price) - 调整入场价格
* [`leverage()`](#leverage-callback) - 杠杆回调
* [`order_filled()`](#order-filled-callback) - 订单成交回调

## Bot start

策略加载时调用一次，可用于执行只需执行一次的操作：

``` python
class AwesomeStrategy(IStrategy):
    def bot_start(self, **kwargs) -> None:
        if self.config["runmode"].value in ("live", "dry_run"):
            self.custom_remote_data = requests.get("https://some_remote_source.example.com")
```

## Bot loop start

在模拟/实盘模式下每次机器人和节流迭代开始时调用，或在回测/超参优化模式下每根 K 线调用一次：

``` python
def bot_loop_start(self, current_time: datetime, **kwargs) -> None:
    if self.config["runmode"].value in ("live", "dry_run"):
        self.remote_data = requests.get("https://some_remote_source.example.com")
```

## 投入金额管理 (`custom_stake_amount`)

在进入交易前调用，允许管理新交易的仓位大小：

```python
def custom_stake_amount(self, pair: str, current_time: datetime, current_rate: float,
                        proposed_stake: float, min_stake: float | None, max_stake: float,
                        leverage: float, entry_tag: str | None, side: str,
                        **kwargs) -> float:
    dataframe, _ = self.dp.get_analyzed_dataframe(pair=pair, timeframe=self.timeframe)
    current_candle = dataframe.iloc[-1].squeeze()
    if current_candle["fastk_rsi_1h"] > current_candle["fastd_rsi_1h"]:
        if self.config["stake_amount"] == "unlimited":
            return max_stake
        else:
            return self.wallets.get_total_stake_amount() / self.config["max_open_trades"]
    return proposed_stake
```

## 自定义出场信号 (`custom_exit`)

每次迭代为未平仓交易调用，直到交易平仓。允许定义自定义出场信号：

``` python
def custom_exit(self, pair: str, trade: Trade, current_time: datetime, current_rate: float,
                current_profit: float, **kwargs):
    dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
    last_candle = dataframe.iloc[-1].squeeze()
    if current_profit > 0.2 and last_candle["rsi"] < 80:
        return "rsi_below_80"
    if 0.02 < current_profit < 0.1 and last_candle["emalong"] > last_candle["emashort"]:
        return "ema_long_below_80"
    if current_profit < 0.0 and (current_time - trade.open_date_utc).days >= 1:
        return "unclog"
```

## 自定义止损 (`custom_stoploss`)

必须通过设置 `use_custom_stoploss=True` 来启用。止损价格只能向上移动。

``` python
use_custom_stoploss = True

def custom_stoploss(self, pair: str, trade: Trade, current_time: datetime,
                    current_rate: float, current_profit: float, after_fill: bool,
                    **kwargs) -> float | None:
    return -0.04 * trade.leverage
```

## 调整交易仓位 (`adjust_trade_position`)

允许在交易期间进行额外的买入或卖出（DCA）。必须通过设置 `position_adjustment_enable=True` 启用。

## 杠杆回调 (`leverage`)

允许按交易对动态设置杠杆：

```python
def leverage(self, pair: str, current_time: datetime, current_rate: float,
             proposed_leverage: float, max_leverage: float, entry_tag: str | None,
             side: str, **kwargs) -> float:
    return 3.0
```

## 确认交易入场/出场

`confirm_trade_entry()` 和 `confirm_trade_exit()` 允许在订单实际下达之前对交易进行最后的拒绝/确认。

## 自定义订单价格规则

`custom_entry_price()` 和 `custom_exit_price()` 允许指定自定义的入场/出场价格。