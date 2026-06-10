# ![freqtrade](https://raw.githubusercontent.com/freqtrade/freqtrade/develop/docs/assets/freqtrade_poweredby.svg)

[![Freqtrade CI](https://github.com/freqtrade/freqtrade/actions/workflows/ci.yml/badge.svg?branch=develop)](https://github.com/freqtrade/freqtrade/actions/workflows/ci.yml)
[![DOI](https://joss.theoj.org/papers/10.21105/joss.04864/status.svg)](https://doi.org/10.21105/joss.04864)
[![codecov](https://codecov.io/gh/freqtrade/freqtrade/branch/develop/graph/badge.svg?token=AD5BG3ATKI)](https://codecov.io/gh/freqtrade/freqtrade)
[![Documentation](https://readthedocs.org/projects/freqtrade/badge/)](https://www.freqtrade.io)
[![Discord Server](https://img.shields.io/badge/Freqtrade_Discord-4E4E4E?logo=discord)](https://discord.gg/p7nuUNVfP7)

Freqtrade 是一个免费开源、用 Python 编写的加密货币交易机器人。它支持所有主流交易所，可通过 Telegram 或 Web UI 进行控制。内置回测、图表和资金管理工具，以及基于机器学习的策略优化。

![freqtrade](https://raw.githubusercontent.com/freqtrade/freqtrade/develop/docs/assets/freqtrade-screenshot.png)

## 免责声明

本软件仅供教育目的使用。不要投入你无法承受损失的资金。使用本软件的风险完全由你自行承担。作者及所有关联方不对你的交易结果承担任何责任。

始终先在模拟运行（Dry-Run）模式下运行交易机器人，在充分理解其工作原理以及预期盈亏之前，不要投入真实资金。

我们强烈建议你具备编程和 Python 知识。不要犹豫去阅读源代码并理解这个机器人的机制。

## 支持的交易所

请阅读[交易所特定说明](https://www.freqtrade.io/en/stable/exchanges/)以了解每个交易所可能需要的特殊配置。

### 支持的现货交易所

- [X] [Binance](https://www.binance.com/)
- [X] [BingX](https://bingx.com/invite/0EM9RX)
- [X] [Bitget](https://www.bitget.com/)
- [X] [Bitmart](https://bitmart.com/)
- [X] [Bybit](https://bybit.com/)
- [X] [Gate.io](https://www.gate.io/ref/6266643)
- [X] [HTX](https://www.htx.com/)
- [X] [Hyperliquid](https://hyperliquid.xyz/) （去中心化交易所，DEX）
- [X] [Kraken](https://kraken.com/)
- [X] [OKX](https://okx.com/)
- [X] [MyOKX](https://okx.com/) （OKX EEA）
- [ ] [可能还有许多其他交易所](https://github.com/ccxt/ccxt/)。_（不保证能正常工作）_

### 支持的合约交易所

- [X] [Binance](https://www.binance.com/)
- [X] [Bitget](https://www.bitget.com/)
- [X] [Gate.io](https://www.gate.io/ref/6266643)
- [X] [Hyperliquid](https://hyperliquid.xyz/) （去中心化交易所，DEX）
- [X] [OKX](https://okx.com/)
- [X] [Bybit](https://bybit.com/)
- [X] [Kraken](https://www.kraken.com/features/futures)

请在开始前务必阅读[交易所特定说明](https://www.freqtrade.io/en/stable/exchanges/)以及[杠杆交易](https://www.freqtrade.io/en/stable/leverage/)文档。

### 社区验证

经社区确认可用的交易所：

- [X] [Bitvavo](https://bitvavo.com/)
- [X] [Kucoin](https://www.kucoin.com/)

## 文档

我们建议你阅读机器人文档，以确保理解其工作原理。

完整文档请访问 [freqtrade 官网](https://www.freqtrade.io)。

## 功能特性

- [x] **基于 Python 3.11+**：可在任何操作系统上运行 — Windows、macOS 和 Linux。
- [x] **持久化存储**：通过 sqlite 实现数据持久化。
- [x] **模拟运行**：无需投入真实资金即可运行机器人。
- [x] **回测**：对买卖策略进行模拟测试。
- [x] **基于机器学习的策略优化**：利用真实交易所数据进行机器学习，优化买卖策略参数。
- [X] **自适应预测建模**：通过 FreqAI 构建智能策略，利用自适应机器学习方法对市场进行自我训练。[了解更多](https://www.freqtrade.io/en/stable/freqai/)
- [x] **加密货币白名单**：选择你想要交易的加密货币，或使用动态白名单。
- [x] **加密货币黑名单**：选择你想要避开的加密货币。
- [x] **内置 Web UI**：内置 Web 界面来管理你的机器人。
- [x] **通过 Telegram 管理**：使用 Telegram 管理机器人。
- [x] **法币显示盈亏**：以法币显示你的盈亏。
- [x] **性能状态报告**：提供当前交易的性能状态报告。

## 快速开始

请参考 [Docker 快速入门文档](https://www.freqtrade.io/en/stable/docker_quickstart/) 了解如何快速上手。

其他（原生）安装方式请参考[安装文档页面](https://www.freqtrade.io/en/stable/installation/)。

## 基本用法

### 机器人命令

```
usage: freqtrade [-h] [-V]
                 {trade,create-userdir,new-config,show-config,new-strategy,download-data,convert-data,convert-trade-data,trades-to-ohlcv,list-data,backtesting,backtesting-show,backtesting-analysis,edge,hyperopt,hyperopt-list,hyperopt-show,list-exchanges,list-markets,list-pairs,list-strategies,list-hyperoptloss,list-freqaimodels,list-timeframes,show-trades,test-pairlist,convert-db,install-ui,plot-dataframe,plot-profit,webserver,strategy-updater,lookahead-analysis,recursive-analysis}
                 ...

免费开源的加密货币交易机器人

位置参数：
  {trade,create-userdir,new-config,show-config,new-strategy,download-data,convert-data,convert-trade-data,trades-to-ohlcv,list-data,backtesting,backtesting-show,backtesting-analysis,edge,hyperopt,hyperopt-list,hyperopt-show,list-exchanges,list-markets,list-pairs,list-strategies,list-hyperoptloss,list-freqaimodels,list-timeframes,show-trades,test-pairlist,convert-db,install-ui,plot-dataframe,plot-profit,webserver,strategy-updater,lookahead-analysis,recursive-analysis}
    trade               交易模块。
    create-userdir      创建用户数据目录。
    new-config          创建新配置
    show-config         显示解析后的配置
    new-strategy        创建新策略
    download-data       下载回测数据。
    convert-data        将 K 线（OHLCV）数据从一种格式转换为另一种格式。
    convert-trade-data  将交易数据从一种格式转换为另一种格式。
    trades-to-ohlcv     将交易数据转换为 OHLCV 数据。
    list-data           列出已下载的数据。
    backtesting         回测模块。
    backtesting-show    显示历史回测结果
    backtesting-analysis 回测分析模块。
    hyperopt            超参优化模块。
    hyperopt-list       列出超参优化结果
    hyperopt-show       显示超参优化结果详情
    list-exchanges      打印可用交易所。
    list-markets        打印交易所上的市场。
    list-pairs          打印交易所上的交易对。
    list-strategies     打印可用策略。
    list-hyperoptloss   打印可用的超参优化损失函数。
    list-freqaimodels   打印可用的 freqAI 模型。
    list-timeframes     打印交易所可用的时间周期。
    show-trades         显示交易记录。
    test-pairlist       测试你的交易对列表配置。
    convert-db          将数据库迁移到不同系统
    install-ui          安装 FreqUI
    plot-dataframe      绘制带指标的 K 线图。
    plot-profit         生成利润图表。
    webserver           Web 服务器模块。
    strategy-updater    将过时的策略文件更新到当前版本
    lookahead-analysis  检查潜在的未来函数偏差。
    recursive-analysis  检查潜在的递归公式问题。

选项：
  -h, --help            显示此帮助信息并退出
  -V, --version         显示程序版本号并退出
```

### Telegram RPC 命令

Telegram 并非必须。但这是控制机器人的绝佳方式。更多详情和完整命令列表请参阅[文档](https://www.freqtrade.io/en/stable/telegram-usage/)。

- `/start`：启动交易者。
- `/stop`：停止交易者。
- `/stopentry`：停止开新仓。
- `/status <trade_id>|[table]`：列出所有或特定的未平仓交易。
- `/profit [<n>]`：列出过去 n 天内所有已完成交易的累计利润。
- `/profit_long [<n>]`：列出过去 n 天内所有已完成做多交易的累计利润。
- `/profit_short [<n>]`：列出过去 n 天内所有已完成做空交易的累计利润。
- `/forceexit <trade_id>|all`：立即退出指定交易（忽略 `minimum_roi`）。
- `/fx <trade_id>|all`：`/forceexit` 的别名
- `/performance`：按交易对分组显示每个已完成交易的表现
- `/balance`：显示各币种账户余额。
- `/daily <n>`：显示过去 n 天每天的盈亏。
- `/help`：显示帮助信息。
- `/version`：显示版本。

## 开发分支

项目目前设有两个主要分支：

- `develop` — 此分支通常包含新功能，但也可能包含破坏性变更。我们尽力保持此分支尽可能稳定。
- `stable` — 此分支包含最新的稳定版本。此分支通常经过充分测试。
- `feat/*` — 这些是功能分支，开发活跃。除非你想测试特定功能，否则请不要使用这些分支。

## 支持

### 帮助 / Discord

对于文档未涵盖的任何问题，或需了解机器人的更多信息，或只是想与志同道合的人交流，欢迎加入 Freqtrade [Discord 服务器](https://discord.gg/p7nuUNVfP7)。

### [Bug / 问题](https://github.com/freqtrade/freqtrade/issues?q=is%3Aissue)

如果你发现机器人有 bug，请先[搜索问题追踪器](https://github.com/freqtrade/freqtrade/issues?q=is%3Aissue)。如果尚未被报告，请[创建新问题](https://github.com/freqtrade/freqtrade/issues/new/choose)，并确保遵循模板指引，以便团队能够尽快协助你。

对于每一个创建好的 [issue](https://github.com/freqtrade/freqtrade/issues/new/choose)，请跟进并在问题解决后标记满意度或提醒关闭。

-- 维护 GitHub 的[社区政策](https://docs.github.com/en/site-policy/github-terms/github-community-code-of-conduct)--

### [功能请求](https://github.com/freqtrade/freqtrade/labels/enhancement)

你有改进机器人的好主意想要分享？请先搜索该功能是否[已被讨论过](https://github.com/freqtrade/freqtrade/labels/enhancement)。如果尚未被提出，请[创建新请求](https://github.com/freqtrade/freqtrade/issues/new/choose)，并确保遵循模板指引，以免被淹没在 bug 报告中。

### [Pull Request](https://github.com/freqtrade/freqtrade/pulls)

觉得机器人缺少某个功能？我们欢迎你的 Pull Request！

请在提交 Pull Request 之前阅读[贡献指南](https://github.com/freqtrade/freqtrade/blob/develop/CONTRIBUTING.md)以了解相关要求。

不一定要写代码才能贡献 — 也许可以从改进文档开始？
标记为 [good first issue](https://github.com/freqtrade/freqtrade/labels/good%20first%20issue) 的问题适合作为第一次贡献，有助于你熟悉代码库。

**注意** 在开始任何重大新功能开发之前，*请先创建一个 issue 描述你计划要做的事情*，或在 [Discord](https://discord.gg/p7nuUNVfP7) 上与我们讨论（请使用 #dev 频道）。这样可以确保相关人员能够对该功能提供有价值的反馈，也让其他人知道有人正在开发它。

**重要提示：** 始终针对 `develop` 分支创建你的 PR，而非 `stable`。

## 系统要求

### 准确的时钟

时钟必须准确，并频繁与 NTP 服务器同步，以避免与交易所通信时出现问题。

### 最低硬件要求

运行此机器人，我们建议使用云实例，最低配置如下：

- 最低（建议）系统要求：2GB 内存、1GB 磁盘空间、2vCPU

### 软件要求

- [Python >= 3.11](http://docs.python-guide.org/en/latest/starting/installation/)
- [pip](https://pip.pypa.io/en/stable/installing/)
- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [TA-Lib](https://ta-lib.github.io/ta-lib-python/)
- [virtualenv](https://virtualenv.pypa.io/en/stable/installation.html) （推荐）
- [Docker](https://www.docker.com/products/docker) （推荐）