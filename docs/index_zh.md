![freqtrade](assets/freqtrade_poweredby.svg)

[![Freqtrade CI](https://github.com/freqtrade/freqtrade/actions/workflows/ci.yml/badge.svg?branch=develop)](https://github.com/freqtrade/freqtrade/actions/workflows/ci.yml)
[![DOI](https://joss.theoj.org/papers/10.21105/joss.04864/status.svg)](https://doi.org/10.21105/joss.04864)
[![codecov](https://codecov.io/gh/freqtrade/freqtrade/branch/develop/graph/badge.svg?token=AD5BG3ATKI)](https://codecov.io/gh/freqtrade/freqtrade)
[![Documentation](https://readthedocs.org/projects/freqtrade/badge/)](https://www.freqtrade.io)
[![Discord Server](https://img.shields.io/badge/Freqtrade_Discord-4E4E4E?logo=discord)](https://discord.gg/p7nuUNVfP7)

<!-- GitHub action buttons -->
[:octicons-star-16: Star](https://github.com/freqtrade/freqtrade){ .md-button .md-button--sm }
[:octicons-repo-forked-16: Fork](https://github.com/freqtrade/freqtrade/fork){ .md-button .md-button--sm }
[:octicons-download-16: Download](https://github.com/freqtrade/freqtrade/archive/stable.zip){ .md-button .md-button--sm }

## 简介

Freqtrade 是一个用 Python 编写的免费开源加密货币交易机器人。它设计用于支持所有主流交易所，并可通过 Telegram 或 WebUI 进行控制。它包含回测、绘图和资金管理工具，以及通过机器学习进行策略优化。

!!! Danger "免责声明"
    本软件仅供教育目的使用。不要冒险投入你害怕亏损的资金。使用本软件的风险由你自行承担。作者及所有关联方对你的交易结果不承担任何责任。

    始终先在模拟（Dry-run）模式下运行交易机器人，在理解其工作原理以及预期盈亏之前不要投入真实资金。

    我们强烈建议你具备基本的编程技能和 Python 知识。请毫不犹豫地阅读源代码并理解本机器人的机制、算法和技术。

![freqtrade screenshot](assets/freqtrade-screenshot.png)

## 功能

- 开发你的策略：用 Python 编写策略，使用 [pandas](https://pandas.pydata.org/)。可在[策略仓库](https://github.com/freqtrade/freqtrade-strategies)中找到可供参考的示例策略。
- 下载市场数据：下载你希望交易的交易所和市场的历史数据。
- 回测：在下载的历史数据上测试你的策略。
- 优化：使用超参优化（采用机器学习方法）找到策略的最佳参数。你可以优化买入、卖出、止盈（ROI）、止损和跟踪止损参数。
- 选择市场：创建你的静态交易对列表，或使用基于交易量和/或价格的自动列表（回测期间不可用）。你也可以明确将不想交易的交易对列入黑名单。
- 运行：用模拟资金测试你的策略（Dry-Run 模式），或用真实资金部署（Live-Trade 模式）。
- 控制/监控：使用 Telegram 或 WebUI（启动/停止机器人、显示盈亏、每日摘要、当前未平仓交易结果等）。
- 分析：可以对回测数据或 Freqtrade 交易历史（SQL 数据库）进行进一步分析，包括自动化标准图表，以及将数据加载到[交互式环境](data-analysis.md)中的方法。

## 支持的交易所市场

请阅读[交易所特定说明](exchanges.md)，了解每个交易所可能需要进行的特殊配置。

### 支持的现货交易所

- [X] [Binance](https://www.binance.com/)
- [X] [BingX](https://bingx.com/invite/0EM9RX)
- [X] [Bitget](https://www.bitget.com/)
- [X] [Bitmart](https://bitmart.com/)
- [X] [Bybit](https://bybit.com/)
- [X] [Gate.io](https://www.gate.io/ref/6266643)
- [X] [HTX](https://www.htx.com/)
- [X] [Hyperliquid](https://hyperliquid.xyz/)（去中心化交易所，DEX）
- [X] [Kraken](https://kraken.com/)
- [X] [OKX](https://okx.com/)
- [X] [MyOKX](https://okx.com/)（OKX EEA）
- [ ] [可能通过 <img alt="ccxt" width="30px" src="assets/ccxt-logo.svg" /> 支持许多其他交易所](https://github.com/ccxt/ccxt/)。_（我们无法保证它们能正常工作）_

### 支持的合约交易所

- [X] [Binance](https://www.binance.com/)
- [X] [Bitget](https://www.bitget.com/)
- [X] [Bybit](https://bybit.com/)
- [X] [Gate.io](https://www.gate.io/ref/6266643)
- [X] [Hyperliquid](https://hyperliquid.xyz/)（去中心化交易所，DEX）
- [X] [OKX](https://okx.com/)
- [X] [Kraken](https://www.kraken.com/features/futures)

在开始之前，请务必阅读[交易所特定说明](exchanges.md)以及[杠杆交易](leverage.md)文档。

### 社区已验证

经社区确认可正常工作的交易所：

- [X] [Bitvavo](https://bitvavo.com/)
- [X] [Kucoin](https://www.kucoin.com/)

## 社区展示

--8<-- "includes/showcase.md"

## 要求

### 硬件要求

要运行此机器人，我们建议使用至少满足以下条件的 Linux 云实例：

- 2GB RAM
- 1GB 磁盘空间
- 2vCPU

### 软件要求

- Docker（推荐）

或者

- Python 3.11+
- pip（pip3）
- git
- TA-Lib
- virtualenv（推荐）

## 支持

### 帮助 / Discord

对于文档未涵盖的任何问题，或有关机器人的更多信息，或仅仅是与志同道合的人交流，我们鼓励你加入 Freqtrade [Discord 服务器](https://discord.gg/p7nuUNVfP7)。

## 准备好了吗？

从阅读[适用于 Docker 的安装指南](docker_quickstart.md)（推荐）或[不使用 Docker 的安装指南](installation.md)开始。