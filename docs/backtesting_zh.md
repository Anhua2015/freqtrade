# 回测

本页介绍如何使用回测来验证你的策略表现。

回测需要历史数据可用。
要了解如何获取你感兴趣的交易对和交易所的数据，请前往文档的 [数据下载](data-download.md) 部分。

回测也可在 [Web 服务器模式](freq-ui.md#backtesting) 下使用，允许你通过 Web 界面运行回测。

## 回测命令参考

--8<-- "commands/backtesting.md"

## 用回测测试你的策略

现在你已经有了良好的入场和出场策略以及一些历史数据，你想用真实数据来测试它。这就是我们所说的[回测](https://en.wikipedia.org/wiki/Backtesting)。

回测将使用你配置文件中的加密货币（交易对），并默认从 `user_data/data/<exchange>` 加载历史 K 线（OHLCV）数据。
如果某个交易所 / 交易对 / 时间周期组合没有可用数据，回测会提示你先使用 `freqtrade download-data` 下载数据。
有关下载的详细信息，请参阅文档中的 [数据下载](data-download.md) 部分。

回测的结果将确认你的机器人是否有更高的盈利概率。

所有利润计算都包含手续费，freqtrade 将使用交易所的默认手续费进行计算。

!!! Warning "在回测中使用动态交易对列表"
    使用动态交易对列表是可行的（并非所有处理器都允许在回测模式下使用），但它依赖于当前市场状况——这不会反映交易对列表的历史状态。
    此外，当使用 StaticPairlist 以外的交易对列表时，回测结果的复现性无法保证。
    请阅读[交易对列表文档](plugins.md#pairlists)了解更多信息。

    要获得可复现的结果，最好通过 [`test-pairlist`](utils.md#test-pairlist) 命令生成一个交易对列表，并将其用作静态交易对列表。

!!! Note
    默认情况下，Freqtrade 会将回测结果导出到 `user_data/backtest_results`。
    导出的交易记录可用于[进一步分析](#further-backtest-result-analysis)，或供 scripts 目录下的 [plotting 子命令](plotting.md#plot-price-and-indicators)（`freqtrade plot-dataframe`）使用。

### 起始余额

回测需要一个起始余额，可以通过 `--dry-run-wallet <balance>` 或 `--starting-balance <balance>` 命令行参数提供，也可以通过 `dry_run_wallet` 配置项来设置。
此金额必须高于 `stake_amount`，否则机器人无法模拟任何交易。

### 动态投入金额

回测支持[动态投入金额](configuration.md#dynamic-stake-amount)，将 `stake_amount` 配置为 `"unlimited"` 即可，这样会按 `max_open_trades` 的份数拆分起始余额。
早期交易产生的利润会使后续的投入金额更高，从而在回测期间实现利润的复利效应。

### 回测命令示例

使用 5 分钟 K 线（OHLCV）数据（默认）

```bash
freqtrade backtesting --strategy AwesomeStrategy
```

其中 `--strategy AwesomeStrategy` / `-s AwesomeStrategy` 指的是策略的类名，该类位于 `user_data/strategies` 目录下的某个 Python 文件中。

---

使用 1 分钟 K 线（OHLCV）数据

```bash
freqtrade backtesting --strategy AwesomeStrategy --timeframe 1m
```

---

提供 1000（投入货币单位）的自定义起始余额

```bash
freqtrade backtesting --strategy AwesomeStrategy --dry-run-wallet 1000
```

---

使用磁盘上不同的历史 K 线（OHLCV）数据源

假设你从 Binance 交易所下载了历史数据并保存在 `user_data/data/binance-20180101` 目录中。
你可以按如下方式使用这些数据进行回测：

```bash
freqtrade backtesting --strategy AwesomeStrategy --datadir user_data/data/binance-20180101 
```

---

比较多个策略

```bash
freqtrade backtesting --strategy-list SampleStrategy1 AwesomeStrategy --timeframe 5m
```

其中 `SampleStrategy1` 和 `AwesomeStrategy` 指的是策略的类名。

---

阻止将交易导出到文件

```bash
freqtrade backtesting --strategy backtesting --export none --config config.json 
```

只有在你确定不需要进一步绘图或分析结果时才使用此选项。

---

导出交易到文件并指定自定义目录

```bash
freqtrade backtesting --strategy backtesting --export trades --backtest-directory=user_data/custom-backtest-results
```

---

另请阅读关于[策略启动期](strategy-customization.md#strategy-startup-period)的内容。

---

提供自定义手续费值

有时你的账户享有某些手续费折扣（达到一定账户规模或月度交易量后的手续费减免），而这些信息对 ccxt 不可见。
要在回测中体现这一点，你可以使用 `--fee` 命令行选项向回测提供此值。
此手续费必须是一个比率，且会被应用两次（一次用于入场，一次用于出场）。

例如，如果每笔订单的佣金费率为 0.1%（即写作比率 0.001），则按以下方式运行回测：

```bash
freqtrade backtesting --fee 0.001
```

!!! Note
    仅在你想要试验不同手续费值时才提供此选项（或相应的配置参数）。默认情况下，回测会从交易所交易对/市场信息中获取默认手续费。

---

使用时间范围以较小的测试集运行回测

使用 `--timerange` 参数来更改你想使用的测试集时间范围。

例如，使用 `--timerange=20190501-` 选项运行回测将使用输入数据中自 2019 年 5 月 1 日起的所有可用数据。

```bash
freqtrade backtesting --timerange=20190501-
```

你也可以指定特定的日期范围。

完整的时间范围规范：

- 使用截至 2018/01/31 的数据：`--timerange=-20180131`
- 使用自 2018/01/31 起的数据：`--timerange=20180131-`
- 使用自 2018/01/31 至 2018/03/01 的数据：`--timerange=20180131-20180301`
- 使用 POSIX / 纪元时间戳 1527595200 至 1527618600 之间的数据：`--timerange=1527595200-1527618600`

## 理解回测结果

回测中最重要的是理解结果。

回测结果如下所示：

```
                                               BACKTESTING REPORT                                                
┏━━━━━━━━━━━━━━━┳━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃          Pair ┃ Trades ┃ Avg Profit % ┃  Tot Profit ┃ Tot Profit % ┃    Avg Duration ┃  Win  Draw  Loss  Win% ┃
┡━━━━━━━━━━━━━━━╇━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│ LTC/USDT:USDT │     16 │         1.01 │      56.882 │         5.69 │        16:16:00 │   16     0     0   100 │
│ ETC/USDT:USDT │     12 │         0.73 │      31.513 │         3.15 │         9:55:00 │   11     0     1  91.7 │
│ ETH/USDT:USDT │      8 │         0.69 │      18.659 │         1.87 │ 1 day, 13:55:00 │    7     0     1  87.5 │
│ XLM/USDT:USDT │     10 │          0.3 │      10.694 │         1.07 │        12:08:00 │    9     0     1  90.0 │
│ BTC/USDT:USDT │      8 │         0.22 │       7.502 │         0.75 │ 3 days, 1:24:00 │    6     0     2  75.0 │
│ XRP/USDT:USDT │      9 │        -0.13 │      -6.837 │        -0.68 │        21:18:00 │    8     0     1  88.9 │
│ DOT/USDT:USDT │      6 │        -0.39 │      -9.169 │        -0.92 │         5:35:00 │    4     0     2  66.7 │
│ ADA/USDT:USDT │      8 │        -1.75 │     -52.089 │        -5.21 │        11:38:00 │    6     0     2  75.0 │
│         TOTAL │     77 │         0.23 │      57.157 │         5.72 │        22:12:00 │   67     0    10  87.0 │
└───────────────┴────────┴──────────────┴─────────────┴──────────────┴─────────────────┴────────────────────────┘
                                             LEFT OPEN TRADES REPORT                                              
┏━━━━━━━━━━━━━━━┳━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃          Pair ┃ Trades ┃ Avg Profit % ┃  Tot Profit ┃ Tot Profit % ┃     Avg Duration ┃  Win  Draw  Loss  Win% ┃
┡━━━━━━━━━━━━━━━╇━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│ BTC/USDT:USDT │      1 │        -4.14 │      -9.930 │        -0.99 │ 17 days, 8:00:00 │    0     0     1     0 │
│ ETC/USDT:USDT │      1 │        -4.24 │     -15.365 │        -1.54 │         10:40:00 │    0     0     1     0 │
│ DOT/USDT:USDT │      1 │        -5.29 │     -19.166 │        -1.92 │         11:30:00 │    0     0     1     0 │
│         TOTAL │      3 │        -4.56 │     -44.461 │        -4.45 │  6 days, 2:03:00 │    0     0     3     0 │
└───────────────┴────────┴──────────────┴─────────────┴──────────────┴──────────────────┴────────────────────────┘
                                              ENTER TAG STATS                                              
┏━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Enter Tag ┃ Entries ┃ Avg Profit % ┃  Tot Profit ┃ Tot Profit % ┃ Avg Duration ┃  Win  Draw  Loss  Win% ┃
┡━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│     OTHER │      77 │         0.23 │      57.157 │         5.72 │     22:12:00 │   67     0    10  87.0 │
│     TOTAL │      77 │         0.23 │      57.157 │         5.72 │     22:12:00 │   67     0    10  87.0 │
└───────────┴─────────┴──────────────┴─────────────┴──────────────┴──────────────┴────────────────────────┘
                                              EXIT REASON STATS                                               
┏━━━━━━━━━━━━━┳━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Exit Reason ┃ Exits ┃ Avg Profit % ┃  Tot Profit ┃ Tot Profit % ┃    Avg Duration ┃  Win  Draw  Loss  Win% ┃
┡━━━━━━━━━━━━━╇━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│         roi │    67 │         1.06 │     245.117 │        24.51 │        15:49:00 │   67     0     0   100 │
│ exit_signal │     4 │        -2.23 │     -31.226 │        -3.12 │  1 day, 8:38:00 │    0     0     4     0 │
│  force_exit │     3 │        -4.56 │     -44.461 │        -4.45 │ 6 days, 2:03:00 │    0     0     3     0 │
│   stop_loss │     3 │       -10.14 │    -112.273 │       -11.23 │  1 day, 3:05:00 │    0     0     3     0 │
│       TOTAL │    77 │         0.23 │      57.157 │         5.72 │        22:12:00 │   67     0    10  87.0 │
└─────────────┴───────┴──────────────┴─────────────┴──────────────┴─────────────────┴────────────────────────┘
                                                      MIXED TAG STATS                                                      
┏━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Enter Tag ┃ Exit Reason ┃ Trades ┃ Avg Profit % ┃  Tot Profit ┃ Tot Profit % ┃    Avg Duration ┃  Win  Draw  Loss  Win% ┃
┡━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│           │         roi │     67 │         1.06 │     245.117 │        24.51 │        15:49:00 │   67     0     0   100 │
│           │ exit_signal │      4 │        -2.23 │     -31.226 │        -3.12 │  1 day, 8:38:00 │    0     0     4     0 │
│           │  force_exit │      3 │        -4.56 │     -44.461 │        -4.45 │ 6 days, 2:03:00 │    0     0     3     0 │
│           │   stop_loss │      3 │       -10.14 │    -112.273 │       -11.23 │  1 day, 3:05:00 │    0     0     3     0 │
│     TOTAL │             │     77 │         0.23 │      57.157 │         5.72 │        22:12:00 │   67     0    10  87.0 │
└───────────┴─────────────┴────────┴──────────────┴─────────────┴──────────────┴─────────────────┴────────────────────────┘
                                   SUMMARY METRICS                                    
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Metric                                 ┃ Value                                     ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ Backtesting from                       │ 2025-07-01 00:00:00                       │
│ Backtesting to                         │ 2025-08-01 00:00:00                       │
│ Trading Mode                           │ Isolated Futures                          │
│ Max open trades                        │ 3                                         │
│                                        │                                           │
│ Total/Daily Avg Trades                 │ 77 / 2.48                                 │
│ Starting balance                       │ 1000 USDT                                 │
│ Final balance                          │ 1057.157 USDT                             │
│ Absolute profit                        │ 57.157 USDT                               │
│ Total profit %                         │ 5.72%                                     │
│ CAGR %                                 │ 92.41%                                    │
│ Sharpe (closed trades)                 │ 3.89                                      │
│ Sortino (closed trades)                │ 2.57                                      │
│ Calmar (closed trades)                 │ 43.03                                     │
│ SQN                                    │ 0.71                                      │
│ Profit factor                          │ 1.30                                      │
│ Expectancy (Ratio)                     │ 0.74 (0.04)                               │
│ Avg. daily profit                      │ 1.844 USDT                                │
│ Avg. stake amount                      │ 345.478 USDT                              │
│ Market change                          │ 30.51%                                    │
│ Total trade volume                     │ 53390.788 USDT                            │
│                                        │                                           │
│ Long / Short trades                    │ 67 / 10                                   │
│ Long / Short profit %                  │ 9.19% / -3.48%                            │
│ Long / Short profit USDT               │ 91.940 / -34.783                          │
│                                        │                                           │
│ Best Pair                              │ LTC/USDT:USDT 5.69%                       │
│ Worst Pair                             │ ADA/USDT:USDT -5.21%                      │
│ Best trade                             │ XRP/USDT:USDT 2.00%                       │
│ Worst trade                            │ ADA/USDT:USDT -10.17%                     │
│ Best day                               │ 27.031 USDT                               │
│ Worst day                              │ -47.826 USDT                              │
│ Days win/draw/lose                     │ 20 / 6 / 5                                │
│ Min/Max/Avg. Duration Winners          │ 0d 00:35 / 5d 18:15 / 0d 15:49            │
│ Min/Max/Avg. Duration Losers           │ 0d 10:40 / 17d 08:00 / 2d 17:00           │
│ Max Consecutive Wins / Loss            │ 36 / 3                                    │
│ Rejected Entry signals                 │ 258                                       │
│ Entry/Exit Timeouts                    │ 0 / 0                                     │
│                                        │                                           │
│ Min/Max balance (closed trades)        │ 1003.205 USDT / 1151.425 USDT             │
│ Max % of account underwater            │ 8.19%                                     │
│ Absolute drawdown                      │ 94.268 USDT (8.19%)                       │
│ Drawdown duration                      │ 9 days 08:50:00                           │
│ Profit at drawdown start               │ 151.425 USDT                              │
│ Profit at drawdown end                 │ 57.157 USDT                               │
│ Drawdown start                         │ 2025-07-22 15:10:00                       │
│ Drawdown end                           │ 2025-08-01 00:00:00                       │
│                                        │                                           │
│ Wallet based Metrics                   │                                           │
│ Min/Max balance (wallet balance)       │ 1000 USDT / 1151.425 USDT                 │
│ Min/Max balance dates (wallet balance) │ 2025-07-01 00:05:00 / 2025-07-22 15:15:00 │
│ Max % of account underwater (balance)  │ 5.01%                                     │
│ Absolute drawdown (wallet balance)     │ 54.76 USDT (4.76%)                        │
│ Drawdown duration                      │ 7 days 20:35:00                           │
│ Profit at drawdown start               │ 151.425 USDT                              │
│ Profit at drawdown end                 │ 96.664 USDT                               │
│ Drawdown start                         │ 2025-07-22 15:15:00                       │
│ Drawdown end                           │ 2025-07-30 11:50:00                       │
│ Sharpe (daily wallet balance)          │ 4.42                                      │
│ Sortino (daily wallet balance)         │ 4.35                                      │
│ Calmar (daily wallet balance)          │ 136.07                                    │
└────────────────────────────────────────┴───────────────────────────────────────────┘

Backtested 2025-07-01 00:00:00 -> 2025-08-01 00:00:00 | Max open trades : 3
                                                        STRATEGY SUMMARY                                                        
┏━━━━━━━━━━━━━━━━┳━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━┓
┃       Strategy ┃ Trades ┃ Avg Profit % ┃  Tot Profit ┃ Tot Profit % ┃ Avg Duration ┃  Win  Draw  Loss  Win% ┃       Drawdown ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━┩
│ SampleStrategy │     77 │         0.23 │      57.157 │         5.72 │     22:12:00 │   67     0    10  87.0 │ 94.268   8.19% │
└────────────────┴────────┴──────────────┴─────────────┴──────────────┴──────────────┴────────────────────────┴────────────────┘

```

### 回测报告表

第一张表包含机器人进行的所有交易，包括"未平仓交易"。

最后一行会给出你策略的整体表现，以上表为例：

```
│         TOTAL │     77 │         0.22 │          54.774 │         5.48 │        22:12:00 │   67     0    10  87.0 │
```

机器人进行了 `77` 笔交易，平均持续时间为 `22:12:00`，表现为 `5.48%`（盈利），这意味着以 1000 USDT 的起始资金赚取了总计 `54.774 USDT`。
列 `Avg Profit %` 显示所有已成交交易的平均利润。
列 `Tot Profit %` 则显示总利润相对于起始余额的百分比。

在上述结果中，起始余额为 1000 USDT，绝对利润为 54.774 USDT——因此 `Tot Profit %` 将是 `(54.774 / 1000) * 100 ~= 5.48%`。

你的策略表现受入场策略、出场策略以及你设置的 `minimal_roi` 和 `stop_loss` 影响。

例如，如果你的 `minimal_roi` 仅为 `"0":  0.01`，你不能指望机器人赚取超过 1% 的利润（因为每笔交易达到 1% 时它都会出场）。

```json
"minimal_roi": {
    "0":  0.01
},
```

另一方面，如果你设置了过高的 `minimal_roi`，如 `"0":  0.55`（55%），机器人几乎不可能达到这个利润。
因此，请记住，你的表现是你策略各个元素、配置以及你所设置的加密货币交易对的综合结果。

### 未平仓交易表

第二张表包含机器人在回测期结束时必须 `force_exit`（强制平仓）的所有交易，以便向你呈现完整画面。
这是模拟真实行为的必要做法，因为回测期必须在某个时间点结束，而现实中你可以让机器人一直运行下去。
这些交易也已包含在第一张表中，但为了清晰起见在此表中单独显示。

### 入场标签统计表

第三张表按入场标签（如 `enter_long`、`enter_short`）对交易进行了细分，显示每个标签的入场次数、平均利润百分比、以投入货币计的总利润、总利润百分比、平均持续时间以及盈亏平次数。

### 出场原因统计表

第四张表包含出场原因的汇总（如 `exit_signal`、`roi`、`stop_loss`、`force_exit`）。此表可以告诉你哪些方面需要改进（例如，如果许多 `exit_signal` 交易是亏损的，你应该改进出场信号或考虑禁用它）。

### 混合标签统计表

第五张表结合了入场标签和出场原因，提供了不同入场标签与特定出场原因组合表现的详细视图。这有助于识别哪些入场和出场策略的组合最有效。

### 摘要指标

回测报告的最后一个部分是摘要指标表。
它包含关于你的策略在回测数据上表现的关键指标。

```
                                   SUMMARY METRICS                                    
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Metric                                 ┃ Value                                     ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ Backtesting from                       │ 2025-07-01 00:00:00                       │
│ Backtesting to                         │ 2025-08-01 00:00:00                       │
│ Trading Mode                           │ Isolated Futures                          │
│ Max open trades                        │ 3                                         │
│                                        │                                           │
│ Total/Daily Avg Trades                 │ 77 / 2.48                                 │
│ Starting balance                       │ 1000 USDT                                 │
│ Final balance                          │ 1057.157 USDT                             │
│ Absolute profit                        │ 57.157 USDT                               │
│ Total profit %                         │ 5.72%                                     │
│ CAGR %                                 │ 92.41%                                    │
│ Sharpe (closed trades)                 │ 3.89                                      │
│ Sortino (closed trades)                │ 2.57                                      │
│ Calmar (closed trades)                 │ 43.03                                     │
│ SQN                                    │ 0.71                                      │
│ Profit factor                          │ 1.30                                      │
│ Expectancy (Ratio)                     │ 0.74 (0.04)                               │
│ Avg. daily profit                      │ 1.844 USDT                                │
│ Avg. stake amount                      │ 345.478 USDT                              │
│ Market change                          │ 30.51%                                    │
│ Total trade volume                     │ 53390.788 USDT                            │
│                                        │                                           │
│ Long / Short trades                    │ 67 / 10                                   │
│ Long / Short profit %                  │ 9.19% / -3.48%                            │
│ Long / Short profit USDT               │ 91.940 / -34.783                          │
│                                        │                                           │
│ Best Pair                              │ LTC/USDT:USDT 5.69%                       │
│ Worst Pair                             │ ADA/USDT:USDT -5.21%                      │
│ Best trade                             │ XRP/USDT:USDT 2.00%                       │
│ Worst trade                            │ ADA/USDT:USDT -10.17%                     │
│ Best day                               │ 27.031 USDT                               │
│ Worst day                              │ -47.826 USDT                              │
│ Days win/draw/lose                     │ 20 / 6 / 5                                │
│ Min/Max/Avg. Duration Winners          │ 0d 00:35 / 5d 18:15 / 0d 15:49            │
│ Min/Max/Avg. Duration Losers           │ 0d 10:40 / 17d 08:00 / 2d 17:00           │
│ Max Consecutive Wins / Loss            │ 36 / 3                                    │
│ Rejected Entry signals                 │ 258                                       │
│ Entry/Exit Timeouts                    │ 0 / 0                                     │
│                                        │                                           │
│ Min/Max balance (closed trades)        │ 1003.205 USDT / 1151.425 USDT             │
│ Max % of account underwater            │ 8.19%                                     │
│ Absolute drawdown                      │ 94.268 USDT (8.19%)                       │
│ Drawdown duration                      │ 9 days 08:50:00                           │
│ Profit at drawdown start               │ 151.425 USDT                              │
│ Profit at drawdown end                 │ 57.157 USDT                               │
│ Drawdown start                         │ 2025-07-22 15:10:00                       │
│ Drawdown end                           │ 2025-08-01 00:00:00                       │
│                                        │                                           │
│ Wallet based Metrics                   │                                           │
│ Min/Max balance (wallet balance)       │ 1000 USDT / 1151.425 USDT                 │
│ Min/Max balance dates (wallet balance) │ 2025-07-01 00:05:00 / 2025-07-22 15:15:00 │
│ Max % of account underwater (balance)  │ 5.01%                                     │
│ Absolute drawdown (wallet balance)     │ 54.76 USDT (4.76%)                        │
│ Drawdown duration                      │ 7 days 20:35:00                           │
│ Profit at drawdown start               │ 151.425 USDT                              │
│ Profit at drawdown end                 │ 96.664 USDT                               │
│ Drawdown start                         │ 2025-07-22 15:15:00                       │
│ Drawdown end                           │ 2025-07-30 11:50:00                       │
│ Sharpe (daily wallet balance)          │ 4.42                                      │
│ Sortino (daily wallet balance)         │ 4.35                                      │
│ Calmar (daily wallet balance)          │ 136.07                                    │
└────────────────────────────────────────┴───────────────────────────────────────────┘
```

- `Backtesting from` / `Backtesting to`：回测时间范围（通常通过 `--timerange` 选项定义）。
- `Trading Mode`：现货或合约交易。
- `Max open trades`：`max_open_trades`（或 `--max-open-trades`）的设置——或交易对列表中的交易对数量（取较低者）。
- `Total/Daily Avg Trades`：与回测输出表中的总交易数相同 / 总交易数除以回测天数（这可以让你了解该策略预期会产生多少笔交易）。
- `Starting balance`：起始余额——由 dry-run-wallet（配置或命令行）给出。
- `Final balance`：最终余额——起始余额 + 绝对利润。
- `Absolute profit`：以投入货币计的利润。
- `Total profit %`：总利润百分比。与第一张表 `TOTAL` 行的 `Tot Profit %` 对齐。计算方式为 `(最终资金 − 起始资金) / 起始资金`。
- `CAGR %`：年化复合增长率。
- `Sharpe (closed trades)`：年化夏普比率，仅包含已成交交易（忽略未平仓交易的盈亏）。
- `Sortino (closed trades)`：年化索提诺比率，仅包含已成交交易（忽略未平仓交易的盈亏）。
- `Calmar (closed trades)`：年化卡尔玛比率，仅包含已成交交易（忽略未平仓交易的盈亏）。
- `SQN`：系统质量指数（SQN）——由 Van Tharp 提出。
- `Profit factor`：所有盈利交易利润之和除以所有亏损交易亏损之和。
- `Expectancy (Ratio)`：期望值比率，即每笔交易的平均盈亏。负期望值比率表示你的策略不盈利。
- `Avg. daily profit`：日均利润，计算为 `(总利润 / 回测天数)`。
- `Avg. stake amount`：平均投入金额，即 `stake_amount` 或使用动态投入金额时的平均值。
- `Market change`：回测期间市场的变化。按所有交易对从第一根到最后一根 K 线的收盘价变化取平均值计算。
- `Total trade volume`：为达到上述利润在交易所产生的成交量。
- `Long / Short trades`：多头/空头交易数量拆分（仅当有空头交易时显示）。
- `Long / Short profit %`：多头和空头交易的利润百分比（仅当有空头交易时显示）。
- `Long / Short profit USDT`：多头和空头交易以投入货币计的利润（仅当有空头交易时显示）。
- `Best Pair` / `Worst Pair`：表现最好和最差的交易对（按总利润百分比计），及其对应的 `Tot Profit %`。
- `Best trade` / `Worst trade`：单笔最大盈利交易和单笔最大亏损交易。
- `Best day` / `Worst day`：按日利润计的最好和最差交易日。
- `Days win/draw/lose`：盈利/持平/亏损天数（持平通常是没有已成交交易的天数）。
- `Min/Max/Avg. Duration Winners`：盈利交易的最短、最长和平均持续时间。
- `Min/Max/Avg. Duration Losers`：亏损交易的最短、最长和平均持续时间。
- `Max Consecutive Wins / Loss`：最大连续盈利/亏损次数。
- `Rejected Entry signals`：因已达到 `max_open_trades` 而无法执行的入场信号。
- `Entry/Exit Timeouts`：未成交的入场/出场订单（仅在使用自定义定价时适用）。
- `Min/Max balance (closed trades)`：回测期间基于已成交交易的最低和最高钱包余额。
- `Max % of account underwater`：自模拟开始以来账户从最高点下降的最大百分比。计算为 `(最高余额 - 当前余额) / (最高余额)` 的最大值。
- `Absolute drawdown`：经历的最大绝对回撤，含相对于账户的百分比，计算方式为 `(绝对回撤) / (回撤高点 + 起始余额)`。
- `Absolute drawdown (wallet balance)`：基于未实现余额的最大绝对回撤，含相对于账户的百分比，计算方式同上。
- `Drawdown duration`：最大回撤周期的持续时间。
- `Profit at drawdown start` / `Profit at drawdown end`：最大回撤周期开始和结束时的利润。
- `Drawdown start` / `Drawdown end`：最大回撤的开始和结束时间（也可通过 `plot-dataframe` 子命令可视化）。
- `Min/Max balance (wallet balance)`：回测期间的最低和最高钱包余额——包括未平仓交易中占用的资金。
- `Min/Max balance dates (wallet balance)`：最低和最高未实现余额发生的日期。
- `Sharpe (wallet balance)`：包含未实现利润的年化夏普比率。
- `Sortino (wallet balance)`：包含未实现利润的年化索提诺比率。
- `Calmar (wallet balance)`：包含未实现利润的年化卡尔玛比率。

!!! Tip "基于钱包的指标"
    "Wallet based Metrics" 部分的指标基于未实现余额计算，其中包含未平仓交易中占用的资金。这提供了策略表现的更全面视图，因为它同时考虑了已实现和未实现的盈亏。

### 日 / 周 / 月 / 年细分

你可以使用 `--breakdown <>` 开关获得日、周、月或年度的结果概览。

要可视化月度和年度细分，可以使用以下命令：

``` bash
freqtrade backtesting --strategy MyAwesomeStrategy --breakdown month year
```

``` output
                                 MONTH BREAKDOWN
┏━━━━━━━━━━━━┳━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃      Month ┃ Trades ┃ Tot Profit USDT ┃ Profit Factor ┃  Win  Draw  Loss  Win% ┃
┡━━━━━━━━━━━━╇━━━━━━━━╇━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│ 31/01/2020 │     12 │          44.451 │          7.28 │   10     0     2  83.3 │
│ 29/02/2020 │     30 │           45.41 │          2.36 │   17     0    13  56.7 │
│ 31/03/2020 │     35 │         142.024 │          2.42 │   14     0    21  40.0 │
│ 30/04/2020 │     67 │         -23.692 │          0.81 │   24     0    43  35.8 │
...
...
│ 30/04/2025 │    203 │          -63.43 │          0.81 │   73     0   130  36.0 │
│ 31/05/2025 │    142 │         104.675 │          1.28 │   59     0    83  41.5 │
│ 30/06/2025 │    177 │          -1.014 │           1.0 │   85     0    92  48.0 │
│ 31/07/2025 │    155 │         232.762 │           1.6 │   63     0    92  40.6 │
└────────────┴────────┴─────────────────┴───────────────┴────────────────────────┘
                                  YEAR BREAKDOWN
┏━━━━━━━━━━━━┳━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃       Year ┃ Trades ┃ Tot Profit USDT ┃ Profit Factor ┃  Win  Draw  Loss  Win% ┃
┡━━━━━━━━━━━━╇━━━━━━━━╇━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│ 31/12/2020 │    896 │         868.889 │          1.46 │  351     0   545  39.2 │
│ 31/12/2021 │   1778 │        4487.163 │          1.93 │  745     0  1033  41.9 │
│ 31/12/2022 │   1736 │          938.27 │          1.27 │  698     0  1038  40.2 │
│ 31/12/2023 │   1712 │        1677.126 │          1.68 │  670     0  1042  39.1 │
│ 31/12/2024 │   1609 │        3198.424 │          2.22 │  773     0   836  48.0 │
│ 31/12/2025 │   1042 │         716.174 │          1.33 │  420     0   622  40.3 │
└────────────┴────────┴─────────────────┴───────────────┴────────────────────────┘
```

输出将显示所选时间段内已实现的绝对利润（以投入货币计）的表格，以及额外的统计数据，如交易数量、利润因子，以及在该期间内实现（平仓）的盈亏平次数分布。

### 回测结果缓存

为节省时间，默认情况下，当回测的策略和配置与之前的回测一致时，回测会复用最近一天内的缓存结果。要强制进行新的回测，即使存在相同运行的已有结果，请指定 `--cache none` 参数。

!!! Warning
    对于开放式时间范围（`--timerange 20210101-`），缓存会自动禁用，因为 freqtrade 无法可靠地确保底层数据没有变化。如果原始回测在末尾缺少数据（后来通过下载更多数据修复），也可能在本不应使用缓存的情况下使用缓存结果。
    在这种情况下，请使用 `--cache none` 强制执行一次全新的回测。

### 进一步的回测结果分析

要进一步分析你的回测结果，freqtrade 默认会将交易导出到文件。
然后你可以加载这些交易进行进一步分析，如回测部分的[数据分析](strategy_analysis_example.md#load-backtest-results-to-pandas-dataframe)所示。

此外，你可以在 [Web 服务器模式](freq-ui.md#backtesting) 下使用 freqtrade 在 Web 界面中可视化回测结果。
此模式还允许你加载已有的回测结果，因此你可以在不重新运行回测的情况下分析它们。
在此模式下，可以使用 `--notes "<notes>"` 为回测结果添加备注，这些备注将显示在 Web 界面中。

### 回测输出文件

freqtrade 生成的输出文件是一个 zip 文件，包含以下内容：

- JSON 格式的回测报告
- Feather 格式的市场变化数据
- 策略文件的副本
- 策略参数副本（如果使用了参数文件）
- 脱敏后的配置文件副本

这确保了在相同数据可用的假设下结果可复现。

zip 文件中仅包含策略文件和配置文件，不包含可能的依赖项。

## 回测所做的假设

由于回测缺少 K 线内部发生的详细信息，它需要进行一些假设：

- 交易所[交易限制](#trading-limits-in-backtesting)被遵守
- 入场发生在开盘价，除非指定了自定义价格逻辑
- 所有订单以请求价格成交（无滑点），只要价格在 K 线的最高/最低范围内
- 出场信号出场发生在下一根 K 线的开盘价
- 出场释放其交易槽位，以便用不同的交易对进行新的交易
- 出场信号优先于止损，因为出场信号假定在 K 线开盘时触发
- ROI
  - 出场与最高价比较——但使用 ROI 值（例如 ROI = 2%，最高价 = 5%——因此出场将在 2% 处）
  - 出场永远不会"低于 K 线"，因此 2% 的 ROI 可能在 2.4% 处出场（如果最低价在 2.4% 利润处）
  - 在触发 K 线上生效的 ROI 条目（例如 1h K 线的 `120: 0.02`，来自 `60: 0.05`）将使用该 K 线的开盘价作为出场价
  - 由 `<N>=-1` ROI 条目导致的强制出场使用最低价作为出场值，除非 N 落在 K 线开盘时（例如 1h K 线的 `120: -1`）
- 止损出场精确在止损价发生，即使最低价更低，但损失将比止损价高 `2 * fees`
- 在一根 K 线内，止损在 ROI 之前评估。因此与模拟/实盘模式下相同策略的结果相比，你经常会看到更多以 `stoploss` 为出场原因的交易
- 对于止损，最低价发生在最高价之前，优先保护资金
- 跟踪止损
  - 跟踪止损仅在低于 K 线最低价时调整（否则会被触发）
  - 在触发跟踪止损的交易入场 K 线上，假定使用"最小偏移"（`stop_positive_offset`）来计算止损——而不是最高价。此规则不适用于自定义止损场景，因为没有关于止损逻辑的信息可用。
  - 最高价先发生——调整止损
  - 最低价使用调整后的止损（因此高低价差较大的出场被正确回测）
  - ROI 在跟踪止损之前应用，确保在 ROI 和跟踪止损同时适用时利润在 ROI 处"封顶"
- 出场原因不解释一笔交易是盈利还是亏损，只说明是什么触发了出场（如果使用负 ROI 值，这可能看起来奇怪）
- 评估顺序（如果同一根 K 线上发生多个信号）
  - 出场信号
  - 止损
  - ROI
  - 跟踪止损
- 仓位反转（仅合约）发生在与平仓交易方向相反的入场信号在现有交易平仓的同一根 K 线上触发时。

基于这些假设，回测尽量接近真实交易。然而，回测**永远无法**替代在模拟模式下运行策略。
此外，请记住过去的结果不能保证未来的成功。

除了上述假设外，策略作者应仔细阅读[常见错误](strategy-customization.md#common-mistakes-when-developing-strategies)部分，以避免在回测中使用真实市场条件下不可用的数据。

### 回测中的交易限制

交易所有一定的交易限制，比如最小（和最大）基础货币数量，或最小/最大投入（计价）货币数量。
这些限制通常列在交易所文档中，称为"交易规则"或类似名称，不同交易对之间可能有很大差异。

回测（以及实盘和模拟）确实遵守这些限制，并确保止损可以设置在此值以下——因此该值会略高于交易所规定的值。
然而，Freqtrade 没有关于历史限制的信息。

这可能导致交易限制因使用历史价格而被夸大，导致最小金额 > 50$ 的情况。

例如：

BTC 最小可交易数量为 0.001。
BTC 今日交易价格为 22.000$（0.001 BTC 与此相关）——但回测期间包含高达 50.000$ 的价格。
今日的最小值将是 `0.001 * 22_000`——即 22$。
然而，在某些历史设置中，限制也可能是 50$——基于 `0.001 * 50_000`。

#### 交易精度限制

大多数交易所对价格和数量都有精度限制，因此你不能以 1.0020401 的数量或 1.24567123123 的价格购买一个交易对。
相反，这些价格和数量将被四舍五入或截断（基于交易所的定义）到定义的交易精度。
例如，上述值可能被舍入为数量 1.002 和价格 1.24567。

这些精度值基于当前交易所限制（如[上一节](#trading-limits-in-backtesting)所述），因为历史精度限制不可用。

## 提高回测精度

回测的一个重大局限是无法知道 K 线内部的价格变动方式（是先到最高价再收盘，还是反过来？）。
因此，假设你用 1h 时间周期运行回测，该 K 线将有 4 个价格（开盘、最高、最低、收盘）。

虽然回测确实对此做了一些假设（见上文）——但这永远不可能完美，且总会偏向某一方面。
为了缓解这个问题，freqtrade 可以使用更低（更快）的时间周期来模拟 K 线内部的价格变动。

要使用此功能，你可以在常规回测命令后追加 `--timeframe-detail 5m`。

``` bash
freqtrade backtesting --strategy AwesomeStrategy --timeframe 1h --timeframe-detail 5m
```

这将为所选时间范围加载 1h 数据（主时间周期）以及 5m 数据（细节时间周期）。
策略将以 1h 时间周期进行分析。
可能发生活动的 K 线（有活跃信号、交易对处于交易中）以 5m 时间周期进行评估。
这将允许对 K 线内部价格变动进行更精确的模拟——并且可能导致不同的结果，尤其是在较长时间周期上。

入场通常仍然发生在主 K 线的开盘价，但释放的交易槽位可能更早释放（如果出场信号在 5m K 线上触发），然后可用于不同交易对的新交易。

一旦交易开仓，所有回调函数（`custom_exit()`、`custom_stoploss()` 等）将在每根 5m K 线上运行（因此在上例的 1h 时间周期和 5m 细节时间周期下为 12 次）。

`--timeframe-detail` 必须小于原始时间周期，否则回测将无法启动。

显然，这将需要更多内存（5m 数据比 1h 数据大），也会影响运行时间（取决于交易数量和交易持续时间）。
此外，数据必须已经可用 / 已下载。

!!! Tip
    你可以将此功能用作策略开发的最后一步，确保你的策略没有利用回测[假设](#assumptions-made-by-backtesting)中的某一项。在此模式下表现同样良好的策略在模拟/实盘模式下也有很好的表现机会（尽管只有前向测试（模拟模式）才能真正确认一个策略）。

??? Sample "极端差异示例"
    在极端示例中使用 `--timeframe-detail`（以下所有交易对在 10:00 K 线上都有入场信号）且 `max_open_trades` 为 1 时，可能导致以下回测交易序列：

    | Pair | Entry Time | Exit Time | Duration |
    |------|------------|-----------| -------- |
    | BTC/USDT | 2024-01-01 10:00:00 | 2021-01-01 10:05:00 | 5m |
    | ETH/USDT | 2024-01-01 10:05:00 | 2021-01-01 10:15:00 | 10m |
    | XRP/USDT | 2024-01-01 10:15:00 | 2021-01-01 10:30:00 | 15m |
    | SOL/USDT | 2024-01-01 10:15:00 | 2021-01-01 11:05:00 | 50m |
    | BTC/USDT | 2024-01-01 11:05:00 | 2021-01-01 12:00:00 | 55m |

    不使用 timeframe-detail 时，会看起来像：

    | Pair | Entry Time | Exit Time | Duration |
    |------|------------|-----------| -------- |
    | BTC/USDT | 2024-01-01 10:00:00 | 2021-01-01 11:00:00 | 1h |
    | BTC/USDT | 2024-01-01 11:00:00 | 2021-01-01 12:00:00 | 1h |

    差异显著，因为在不使用细节数据时，每根 K 线只评估前 `max_open_trades` 个信号，且交易槽位仅在 K 线结束时释放，允许在下一根 K 线开新交易。

## 回测多个策略

要比较多个策略，可以向回测提供一个策略列表。

每次运行限制为 1 个时间周期值。然而，数据仅从磁盘加载一次，因此如果你有多个策略想要比较，这将带来不错的运行速度提升。

所有列出的策略必须在同一目录中，除非还指定了 `--recursive-strategy-search`，此时策略目录下的子目录也会被考虑。

``` bash
freqtrade backtesting --timerange 20180401-20180410 --timeframe 5m --strategy-list Strategy001 Strategy002 --export trades
```

这将把结果保存到 `user_data/backtest_results/backtest-result-<datetime>.json`，包含 `Strategy001` 和 `Strategy002` 两个策略的结果。
会有一个额外的表比较不同策略的盈亏情况（与第一张表中的 "Total" 行相同）。
每个策略的详细输出将依次显示，请确保向上滚动查看每个策略的详细信息。

```
================================================== STRATEGY SUMMARY ===================================================================
| Strategy    |  Trades |   Avg Profit % |   Tot Profit BTC |   Tot Profit % | Avg Duration   |  Wins |  Draws | Losses | Drawdown % |
|-------------+---------+----------------+------------------+----------------+----------------+-------+--------+--------+------------|
| Strategy1   |     429 |           0.36 |       0.00762792 |          76.20 | 4:12:00        |   186 |      0 |    243 |       45.2 |
| Strategy2   |    1487 |          -0.13 |      -0.00988917 |         -98.79 | 4:43:00        |   662 |      0 |    825 |     241.68 |
```

## 下一步

太好了，你的策略是盈利的。如果机器人能给你策略的最优参数呢？
你的下一步是学习[如何使用超参优化找到最优参数](hyperopt.md)。