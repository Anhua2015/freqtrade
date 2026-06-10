# 高级回测分析

## 分析买入/入场和卖出/离场标签

了解策略根据用于标记不同买入条件的买入/入场标签如何表现，是很有帮助的。你可能希望看到比默认回测输出提供的更复杂的每个买入和卖出条件的统计数据。你还可能希望查看导致开仓的信号 K 线上的指标值。

!!! Note
    以下买入原因分析仅适用于回测，*不适用于超参优化*。

我们需要使用 `--export` 选项设置为 `signals` 来运行回测，以启用信号**和**交易的导出：

``` bash
freqtrade backtesting -c <config.json> --timeframe <tf> --strategy <strategy_name> --timerange=<timerange> --export=signals
```

这将告诉 freqtrade 输出一个包含策略、交易对以及导致入场和离场信号的 K 线 DataFrame 的 pickle 字典。
根据策略产生的入场数量，这个文件可能会变得非常大，因此请定期检查 `user_data/backtest_results` 文件夹以删除旧的导出文件。

在运行下一次回测之前，请确保要么删除旧的回测结果，要么使用 `--cache none` 选项运行回测，以确保不使用缓存结果。

如果一切顺利，你现在应该在 `user_data/backtest_results` 文件夹中看到 `backtest-result-{timestamp}_signals.pkl` 和 `backtest-result-{timestamp}_exited.pkl` 文件。

要分析入场/离场标签，我们需要使用 `freqtrade backtesting-analysis` 命令，并使用 `--analysis-groups` 选项提供空格分隔的参数：

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-groups 0 1 2 3 4 5
```

此命令将读取最近的回测结果。`--analysis-groups` 选项用于指定各种表格输出，显示每个组或每笔交易的利润，从最简单（0）到最详细（按交易对、按买入和卖出标签）（4）：

* 0：按入场标签的整体胜率和利润摘要
* 1：按入场标签分组的利润摘要
* 2：按入场标签和离场标签分组的利润摘要
* 3：按交易对和入场标签分组的利润摘要
* 4：按交易对、入场标签和离场标签分组的利润摘要（这可能非常大）
* 5：按离场标签分组的利润摘要

更多选项可通过 `-h` 选项查看。

### 使用 backtest-filename

默认情况下，`backtesting-analysis` 处理 `user_data/backtest_results` 目录中最新的回测结果。
如果你想分析更早的回测结果，使用 `--backtest-filename` 选项指定所需的文件。这允许你随时重新访问和重新分析历史回测输出：

``` bash
freqtrade backtesting -c <config.json> --strategy <strategy_name> --timerange <timerange> --export signals --backtest-filename backtest-result-2025-03-05_20-38-34.zip
```

你应该在日志中看到如下输出，其中包含导出的带时间戳的文件名：

```
2022-06-14 16:28:32,698 - freqtrade.misc - INFO - dumping json to "mystrat_backtest-2022-06-14_16-28-32.json"
```

然后你可以在 `backtesting-analysis` 中使用该文件名：

``` bash
freqtrade backtesting-analysis -c <config.json> --backtest-filename=backtest-result-2025-03-05_20-38-34.zip
```

要使用不同结果目录中的结果，你可以使用 `--backtest-directory` 指定目录：

``` bash
freqtrade backtesting-analysis -c <config.json> --backtest-directory custom_results/ --backtest-filename backtest-result-2025-03-05_20-38-34.zip
```

### 调整显示的买入标签和卖出标签

要仅在输出中显示某些买入和卖出标签，使用以下两个选项：

```
--enter-reason-list : 要分析的入场信号列表，空格分隔。默认："all"
--exit-reason-list : 要分析的离场信号列表，空格分隔。默认："all"
```

例如：

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-groups 0 2 --enter-reason-list enter_tag_a enter_tag_b --exit-reason-list roi custom_exit_tag_a stop_loss
```

### 输出信号 K 线指标值

`freqtrade backtesting-analysis` 真正强大的功能在于能够打印出信号 K 线上的指标值，以便对买入信号指标进行精细的调查和调整。要打印特定指标集的列，使用 `--indicator-list` 选项：

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-groups 0 2 --enter-reason-list enter_tag_a enter_tag_b --exit-reason-list roi custom_exit_tag_a stop_loss --indicator-list rsi rsi_1h bb_lowerband ema_9 macd macdsignal
```

指标必须存在于你策略的主 DataFrame 中（无论是主时间周期还是信息性时间周期），否则它们将在脚本输出中被忽略。

!!! Note "指标列表"
    指标值将同时显示入场点和离场点。如果指定了 `--indicator-list all`，则仅显示入场点的指标值，以避免列表过长（取决于策略）。

分析中包含了一系列 K 线和交易相关字段，通过在 indicator-list 中包含它们即可自动访问，这些包括：

* **open_date     :** 交易开仓时间
* **close_date    :** 交易平仓时间
* **min_rate      :** 持仓期间的最低价格
* **max_rate      :** 持仓期间的最高价格
* **open          :** 信号 K 线开盘价
* **close         :** 信号 K 线收盘价
* **high          :** 信号 K 线最高价
* **low           :** 信号 K 线最低价
* **volume        :** 信号 K 线成交量
* **profit_ratio  :** 交易利润率
* **profit_abs    :** 交易的绝对利润回报

#### 指标值示例输出

``` bash
freqtrade backtesting-analysis -c user_data/config.json --analysis-groups 0 --indicator-list chikou_span tenkan_sen 
```

在此示例中，我们旨在显示入场和离场点处的 `chikou_span` 和 `tenkan_sen` 指标值。

指标值的示例输出可能如下：

| pair      | open_date                 | enter_reason | exit_reason | chikou_span (entry) | tenkan_sen (entry) | chikou_span (exit) | tenkan_sen (exit) |
|-----------|---------------------------|--------------|-------------|---------------------|--------------------|--------------------|-------------------|
| DOGE/USDT | 2024-07-06 00:35:00+00:00 |              | exit_signal | 0.105               | 0.106              | 0.105              | 0.107             |
| BTC/USDT  | 2024-08-05 14:20:00+00:00 |              | roi         | 54643.440           | 51696.400          | 54386.000          | 52072.010         |

如表格所示，`chikou_span (entry)` 表示入场时的指标值，而 `chikou_span (exit)` 反映离场时的值。
这种指标值的详细视图增强了分析能力。
`(entry)` 和 `(exit)` 后缀被添加到指标上，以区分入场点和离场点的值。

!!! Note "交易范围指标"
    某些交易范围指标没有 `(entry)` 或 `(exit)` 后缀。这些指标包括：`pair`, `stake_amount`, `max_stake_amount`, `amount`, `open_date`, `close_date`, `open_rate`, `close_rate`, `fee_open`, `fee_close`, `trade_duration`, `profit_ratio`, `profit_abs`, `exit_reason`, `initial_stop_loss_abs`, `initial_stop_loss_ratio`, `stop_loss_abs`, `stop_loss_ratio`, `min_rate`, `max_rate`, `is_open`, `enter_tag`, `leverage`, `is_short`, `open_timestamp`, `close_timestamp` 和 `orders`

#### 基于入场或离场信号筛选指标

`--indicator-list` 选项默认显示入场和离场信号的指标值。要仅筛选入场信号的指标值，可以使用 `--entry-only` 参数。类似地，要仅显示离场信号的指标值，使用 `--exit-only` 参数。

示例：显示入场信号处的指标值：

``` bash
freqtrade backtesting-analysis -c user_data/config.json --analysis-groups 0 --indicator-list chikou_span tenkan_sen --entry-only
```

示例：显示离场信号处的指标值：

``` bash
freqtrade backtesting-analysis -c user_data/config.json --analysis-groups 0 --indicator-list chikou_span tenkan_sen --exit-only
```

!!! note 
    使用这些筛选器时，指标名称不会带有 `(entry)` 或 `(exit)` 后缀。

### 按日期筛选交易输出

要仅显示回测时间范围内两个日期之间的交易，使用通常的 `timerange` 选项，格式为 `YYYYMMDD-[YYYYMMDD]`：

```
--timerange : 筛选输出交易的时间范围，开始日期包含，结束日期不包含。例如 20220101-20221231
```

例如，如果你的回测时间范围是 `20220101-20221231`，但你只想输出一月份的交易：

``` bash
freqtrade backtesting-analysis -c <config.json> --timerange 20220101-20220201
```

### 打印被拒绝的信号

使用 `--rejected-signals` 选项打印被拒绝的信号。

``` bash
freqtrade backtesting-analysis -c <config.json> --rejected-signals
```

### 将表格写入 CSV

某些表格输出可能非常大，因此将它们打印到终端不太理想。
使用 `--analysis-to-csv` 选项禁用将表格打印到标准输出，而是将它们写入 CSV 文件。

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-to-csv
```

默认情况下，这将为你指定的每个输出表写入一个文件，例如：

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-to-csv --rejected-signals --analysis-groups 0 1
```

这将在 `user_data/backtest_results` 中写入：

* rejected_signals.csv
* group_0.csv
* group_1.csv

要覆盖文件的写入位置，还需指定 `--analysis-csv-path` 选项。

``` bash
freqtrade backtesting-analysis -c <config.json> --analysis-to-csv --analysis-csv-path another/data/path/
```