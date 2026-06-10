# Freqtrade 策略 101：策略开发快速入门

在本快速入门中，我们假设你熟悉交易的基础知识，并已阅读 [Freqtrade 基础](bot-basics.md)页面。

## 必备知识

Freqtrade 中的策略是一个 Python 类，定义了买卖加密货币`资产`的逻辑。

资产被定义为`交易对`，表示`币种`和`投入货币`。币种是你使用另一种货币作为投入进行交易的资产。

数据由交易所以`K 线`的形式提供，由六个值组成：`日期`、`开盘`、`最高`、`最低`、`收盘`和`成交量`。

`技术分析`函数使用各种计算和统计公式分析 K 线数据，并产生称为`指标`的二级值。

指标在资产交易对的 K 线上进行分析以生成`信号`。

信号在加密货币`交易所`上转化为`订单`，即`交易`。

我们使用术语`入场`和`出场`而不是`买入`和`卖出`，因为 Freqtrade 同时支持`多头`和`空头`交易。

- **多头**：你基于投入货币买入币种，例如使用 USDT 作为投入买入 BTC，通过以高于支付的价格卖出币种来获利。在多头交易中，利润来自币种相对于投入货币的价值上涨。
- **空头**：你从交易所以币种形式借入资本，随后偿还该币种的投入货币价值。在空头交易中，利润来自币种相对于投入货币的价值下跌（你以更低的价格偿还贷款）。

虽然 Freqtrade 支持某些交易所的现货和合约市场，为简单起见，我们将仅关注现货（多头）交易。

## 基本策略的结构

### 主数据框

Freqtrade 策略使用称为`数据框`的行列式表格数据结构来生成入场和出场交易的信号。

你配置的交易对列表中的每个交易对都有自己的数据框。数据框按`日期`列索引，例如 `2024-06-31 12:00`。
接下来的 5 列表示`开盘`、`最高`、`最低`、`收盘`和`成交量`（OHLCV）数据。

### 填充指标值

`populate_indicators` 函数向数据框添加代表技术分析指标值的列。

常见指标的例子包括相对强弱指数、布林带、资金流量指数、移动平均线和平均真实波幅。

通过调用技术分析函数（如 ta-lib 的 RSI 函数 `ta.RSI()`）并将其赋值给列名（如 `rsi`）来将列添加到数据框中。

```python
dataframe['rsi'] = ta.RSI(dataframe)
```

??? Hint "技术分析库"
    不同的库以不同的方式生成指标值。请查看每个库的文档以了解如何将其集成到你的策略中。你也可以查看 [Freqtrade 示例策略](https://github.com/freqtrade/freqtrade-strategies)来获取灵感。

### 填充入场信号

`populate_entry_trend` 函数定义入场信号的条件。

数据框列 `enter_long` 被添加到数据框中，当此列的值为 `1` 时，Freqtrade 看到一个入场信号。

??? Hint "做空"
    要进入空头交易，使用 `enter_short` 列。

### 填充出场信号

`populate_exit_trend` 函数定义出场信号的条件。

数据框列 `exit_long` 被添加到数据框中，当此列的值为 `1` 时，Freqtrade 看到一个出场信号。

??? Hint "做空"
    要退出空头交易，使用 `exit_short` 列。

## 一个简单策略

以下是一个 Freqtrade 策略的最小示例：

```python
from freqtrade.strategy import IStrategy
from pandas import DataFrame
import talib.abstract as ta

class MyStrategy(IStrategy):

    timeframe = '15m'

    # 设置初始止损为 -10%
    stoploss = -0.10

    # 当利润大于 1% 时随时退出盈利仓位
    minimal_roi = {"0": 0.01}

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # 生成技术分析指标的值
        dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)

        return dataframe

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # 基于指标值生成入场信号
        dataframe.loc[
            (dataframe['rsi'] < 30),
            'enter_long'] = 1

        return dataframe

    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # 基于指标值生成出场信号
        dataframe.loc[
            (dataframe['rsi'] > 70),
            'exit_long'] = 1

        return dataframe
```

## 进行交易

当发现信号时（入场或出场列中的 `1`），Freqtrade 将尝试下订单，即进行`交易`或建立`仓位`。

每个新的交易仓位占用一个`槽位`。槽位表示可以同时开立的最大新交易数量。
槽位数量由 `max_open_trades` [配置](configuration.md)选项定义。

然而，有多种情况可能导致生成信号并不总是创建交易订单。这些包括：

- 剩余投入不足以买入资产，或钱包中资金不足以卖出资产（包括任何手续费）
- 没有足够的空闲槽位来开新交易（你已开仓位数量等于 `max_open_trades` 选项）
- 该交易对已有未平仓交易（Freqtrade 不能叠加仓位——然而它可以[调整现有仓位](strategy-callbacks.md#adjust-trade-position)）
- 如果同一根 K 线上同时存在入场和出场信号，它们被视为[冲突](strategy-customization.md#colliding-signals)，不会发起订单
- 策略通过使用相关的[入场](strategy-callbacks.md#trade-entry-buy-order-confirmation)或[出场](strategy-callbacks.md#trade-exit-sell-order-confirmation)回调主动拒绝交易订单

阅读[策略定制](strategy-customization.md)文档了解更多细节。

## 回测和前向测试

策略开发可能是一个漫长且令人沮丧的过程，因为将我们人类的"直觉"转化为计算机控制的（"算法"）策略并不总是一帆风顺。

因此，策略应经过测试以验证它将按预期工作。

Freqtrade 有两种测试模式：

- **回测**：使用你[从交易所下载](data-download.md)的历史数据，回测是快速评估策略表现的方法。然而，很容易扭曲结果使策略看起来比实际更盈利。请查看[回测文档](backtesting.md)了解更多信息。
- **模拟运行**：通常被称为_前向测试_，模拟运行使用交易所的实时数据。然而，任何会导致交易的信号都由 Freqtrade 正常跟踪，但不会在交易所本身上开立任何交易。前向测试是实时运行的，因此虽然需要更长时间才能获得结果，但它是比回测更可靠的**潜在**表现指标。

模拟运行通过在你的[配置](configuration.md#using-dry-run-mode)中将 `dry_run` 设置为 true 来启用。

!!! Warning "回测可能非常不准确"
    回测结果可能与现实不匹配的原因有很多。请查看[回测假设](backtesting.md#assumptions-made-by-backtesting)和[常见策略错误](strategy-customization.md#common-mistakes-when-developing-strategies)文档。
    一些列出和排名 Freqtrade 策略的网站展示了令人印象深刻的回测结果。不要假设这些结果是可实现的或现实的。

??? Hint "有用的命令"
    Freqtrade 包含两个有用的命令来检查策略中的基本缺陷：[lookahead-analysis](lookahead-analysis.md) 和 [recursive-analysis](recursive-analysis.md)。

### 评估回测和模拟运行结果

回测后始终模拟运行你的策略，以查看回测和模拟运行结果是否足够相似。

如果存在任何显著差异，请验证你的入场和出场信号是否一致，并在两种模式之间出现在相同的 K 线上。然而，模拟运行和回测之间总会存在差异：

- 回测假设所有订单成交。在模拟运行中，如果使用限价单或交易所没有成交量，则可能不是这种情况。
- 在 K 线收盘时跟随入场信号，回测假设交易在下一根 K 线的开盘价入场（除非你的策略中有自定义定价回调）。在模拟运行中，信号和交易开立之间通常存在延迟。这是因为当新 K 线在你的主时间周期上到来时（例如每 5 分钟），Freqtrade 需要时间来分析所有交易对的数据框。因此，Freqtrade 将在 K 线开盘后几秒钟（理想情况下尽可能小的延迟）尝试开立交易。
- 由于模拟运行中的入场价格可能与回测不匹配，这意味着利润计算也会不同。因此，ROI、止损、跟踪止损和回调出场不完全相同是正常的。
- 新 K 线到来与你的信号被触发和交易被开立之间的计算"延迟"越大，价格不可预测性就越大。确保你的计算机足够强大，能够在合理时间内处理你的交易对列表中交易对数量的数据。如果有显著的数据处理延迟，Freqtrade 会在日志中警告你。

## 控制或监控运行中的机器人

一旦你的机器人在模拟或实盘模式下运行，Freqtrade 有六种机制来控制或监控运行中的机器人：

- **[FreqUI](freq-ui.md)**：最容易上手，FreqUI 是一个 Web 界面，用于查看和控制机器人的当前活动。
- **[Telegram](telegram-usage.md)**：在移动设备上，Telegram 集成可用于获取关于机器人活动的提醒并控制某些方面。
- **[FTUI](https://github.com/freqtrade/ftui)**：FTUI 是 Freqtrade 的终端（命令行）界面，仅允许监控运行中的机器人。
- **[freqtrade-client](rest-api.md#consuming-the-api)**：REST API 的 Python 实现，使你可以轻松地从 Python 应用或命令行发起请求并使用机器人响应。
- **[REST API 端点](rest-api.md#available-endpoints)**：REST API 允许程序员开发自己的工具来与 Freqtrade 机器人交互。
- **[Webhooks](webhook-config.md)**：Freqtrade 可以通过 webhooks 向其他服务（如 discord）发送信息。

### 日志

Freqtrade 生成大量的调试日志来帮助你理解正在发生的事情。请熟悉你可能会在机器人日志中看到的信息和错误消息。

默认情况下日志输出到标准输出（命令行）。如果你想写入文件，许多 freqtrade 命令（包括 `trade` 命令）都接受 `--logfile` 选项来写入文件。

查看 [FAQ](faq.md#how-do-i-search-the-bot-logs-for-something) 获取示例。

## 最后的思考

算法交易很困难，大多数公开策略表现不佳，因为要使策略在多种场景下盈利需要大量的时间和努力。

因此，拿公开策略并使用回测作为评估表现的方式通常是有问题的。然而，Freqtrade 提供了有用的方法来帮助你做出决策并进行尽职调查。

实现盈利有许多不同的方式，没有一个单一的技巧、窍门或配置选项能修复表现不佳的策略。

Freqtrade 是一个拥有庞大且乐于助人社区的开源平台——请务必访问我们的 [discord 频道](https://discord.gg/p7nuUNVfP7)与他人讨论你的策略！

一如既往，只投入你愿意损失的资金。

## 结语

在 Freqtrade 中开发策略涉及基于技术指标定义入场和出场信号。通过遵循上述结构和方法，你可以创建和测试自己的交易策略。

常见问题和答案可在我们的 [FAQ](faq.md) 中找到。

要继续，请参考更深入的 [Freqtrade 策略定制文档](strategy-customization.md)。