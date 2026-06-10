# Freqtrade 基础

本页介绍 Freqtrade 如何工作和运行的一些基本概念。

## Freqtrade 术语

* **Strategy（策略）**：你的交易策略，告诉机器人该做什么。
* **Trade（交易）**：已开仓位。
* **Open Order（挂单）**：当前已在交易所下单但尚未成交的订单。
* **Pair（交易对）**：可交易的交易对，通常格式为 基础货币/计价货币（例如现货为 `XRP/USDT`，合约为 `XRP/USDT:USDT`）。
* **Timeframe（时间周期）**：使用的 K 线周期（例如 `"5m"`、`"1h"` 等）。
* **Indicators（指标）**：技术指标（SMA、EMA、RSI 等）。
* **Limit order（限价单）**：以指定限价或更优价格成交的订单。
* **Market order（市价单）**：保证成交，可能因订单大小影响价格。
* **Current Profit（当前利润）**：该笔交易当前待定（未实现）的利润。主要在机器人和 UI 中使用。
* **Realized Profit（已实现利润）**：已经实现的利润。仅与[部分平仓](strategy-callbacks.md#adjust-trade-position)相关——其中也解释了此指标的计算逻辑。
* **Total Profit（总利润）**：已实现和未实现利润的总和。相对数值（%）按该笔交易的总投入计算。

## 手续费处理

Freqtrade 所有利润计算均包含手续费。回测 / 超参优化 / 模拟模式下使用交易所默认手续费（交易所最低费率档位）。实盘操作时使用交易所实际扣收的手续费（包括 BNB 返佣等）。

## 交易对命名

Freqtrade 遵循 [ccxt 命名约定](https://docs.ccxt.com/#/README?id=consistency-of-base-and-quote-currencies) 来命名货币。
在错误的市场上使用错误的命名约定通常会导致机器人无法识别交易对，通常表现为 "this pair is not available" 之类的错误。

### 现货交易对命名

现货交易对的命名为 `基础货币/计价货币`（例如 `ETH/USDT`）。

### 合约交易对命名

合约交易对的命名为 `基础货币/计价货币:结算货币`（例如 `ETH/USDT:USDT`）。

## 机器人执行逻辑

以模拟或实盘模式启动 freqtrade（使用 `freqtrade trade`）将启动机器人并进入机器人迭代循环。
同时也会执行 `bot_start()` 回调。

默认情况下，机器人循环每隔几秒（`internals.process_throttle_secs`）运行一次，并执行以下操作：

* 从持久化存储中获取未平仓交易。
* 计算当前可交易交易对列表。
* 为交易对列表下载 OHLCV 数据，包括所有[信息性交易对](strategy-customization.md#get-data-for-non-tradeable-pairs)。
  此步骤每根 K 线仅执行一次，以避免不必要的网络流量。
* 调用 `bot_loop_start()` 策略回调。
* 按交易对分析策略。
  * 调用 `populate_indicators()`
  * 调用 `populate_entry_trend()`
  * 调用 `populate_exit_trend()`
* 从交易所更新交易挂单状态。
  * 对已成交订单调用 `order_filled()` 策略回调。
  * 检查挂单超时。
    * 对未成交的入场挂单调用 `check_entry_timeout()` 策略回调。
    * 对未成交的出场挂单调用 `check_exit_timeout()` 策略回调。
    * 对挂单调用 `adjust_order_price()` 策略回调。
      * 对未成交的入场挂单调用 `adjust_entry_price()` 策略回调。*仅在未实现 `adjust_order_price()` 时调用*
      * 对未成交的出场挂单调用 `adjust_exit_price()` 策略回调。*仅在未实现 `adjust_order_price()` 时调用*
* 验证现有仓位并适时下出场单。
  * 考虑止损、ROI 和出场信号、`custom_exit()` 以及 `custom_stoploss()`。
  * 根据 `exit_pricing` 配置或通过 `custom_exit_price()` 回调确定出场价。
  * 在下出场单之前，调用 `confirm_trade_exit()` 策略回调。
* 如果启用了仓位调整功能，检查已开交易的仓位调整，调用 `adjust_trade_position()` 并按需下加仓单。
* 检查是否还有可用交易槽位（是否已达到 `max_open_trades`）。
* 验证入场信号，尝试开新仓位。
  * 根据 `entry_pricing` 配置或通过 `custom_entry_price()` 回调确定入场价。
  * 在保证金和合约模式下，调用 `leverage()` 策略回调确定所需杠杆。
  * 通过调用 `custom_stake_amount()` 回调确定投入金额。
  * 在下入场单之前，调用 `confirm_trade_entry()` 策略回调。

此循环将不断重复，直到机器人停止。

## 回测 / 超参优化执行逻辑

[回测](backtesting.md) 或 [超参优化](hyperopt.md) 仅执行上述逻辑的一部分，因为大部分交易操作是完全模拟的。

* 加载已配置交易对列表的历史数据。
* 调用一次 `bot_start()`。
* 计算指标（每个交易对调用一次 `populate_indicators()`）。
* 计算入场 / 出场信号（每个交易对调用一次 `populate_entry_trend()` 和 `populate_exit_trend()`）。
* 按 K 线循环模拟入场和出场点位。
  * 调用 `bot_loop_start()` 策略回调。
  * 检查订单超时，通过 `unfilledtimeout` 配置或 `check_entry_timeout()` / `check_exit_timeout()` 策略回调。
  * 对挂单调用 `adjust_order_price()` 策略回调。
    * 对未成交的入场挂单调用 `adjust_entry_price()` 策略回调。*仅在未实现 `adjust_order_price()` 时调用！*
    * 对未成交的出场挂单调用 `adjust_exit_price()` 策略回调。*仅在未实现 `adjust_order_price()` 时调用！*
  * 检查交易入场信号（`enter_long` / `enter_short` 列）。
  * 确认交易入场 / 出场（如果策略中实现了 `confirm_trade_entry()` 和 `confirm_trade_exit()` 则调用）。
  * 调用 `custom_entry_price()`（如果策略中实现）确定入场价（价格会被调整到开盘 K 线范围内）。
  * 在保证金和合约模式下，调用 `leverage()` 策略回调确定所需杠杆。
  * 通过调用 `custom_stake_amount()` 回调确定投入金额。
  * 如果启用了仓位调整功能，检查已开交易的仓位调整并调用 `adjust_trade_position()` 来确定是否需要加仓。
  * 对已成交的入场订单调用 `order_filled()` 策略回调。
  * 调用 `custom_stoploss()` 和 `custom_exit()` 来寻找自定义出场点。
  * 对于基于出场信号、自定义出场和部分平仓的出场：调用 `custom_exit_price()` 确定出场价（价格会被调整到收盘 K 线范围内）。
  * 对已成交的出场订单调用 `order_filled()` 策略回调。
* 生成回测报告输出

!!! Note
    回测和超参优化的计算中均包含交易所默认手续费。可以通过指定 `--fee` 参数向回测 / 超参优化传入自定义手续费。

!!! Warning "回调调用频率"
    回测中每个回调最多每根 K 线调用一次（`--timeframe-detail` 会将此行为修改为每根细节 K 线调用一次）。
    在实盘中，大多数回调每次迭代调用一次（通常约每 5 秒一次）——这可能导致回测与实盘之间的差异。