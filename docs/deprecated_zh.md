# 已弃用的功能

本页描述了被机器人开发团队声明为 DEPRECATED（已弃用）且不再支持的命令行参数、配置参数和机器人功能。请避免在你的配置中使用它们。

## 已移除的功能

### `--refresh-pairs-cached` 命令行选项

在回测、超参优化和 edge 的上下文中，`--refresh-pairs-cached` 允许刷新用于回测的 K 线数据。
由于这导致了许多困惑，并减慢了回测速度（同时不是回测的一部分），这已被单独拆分为一个独立的 freqtrade 子命令 `freqtrade download-data`。

此命令行选项于 2019.7-dev（develop 分支）弃用，并于 2019.9 移除。

### **--dynamic-whitelist** 命令行选项

此命令行选项于 2018 年弃用，并于 freqtrade 2019.6-dev（develop 分支）和 freqtrade 2019.7 中移除。
请改用[交易对列表](plugins.md#pairlists-and-pairlist-handlers)。

### `--live` 命令行选项

回测上下文中的 `--live` 允许下载最新的 tick 数据进行回测。
仅下载最近 500 根 K 线，因此在获取良好回测数据方面效果不佳。
于 2019-7-dev（develop 分支）和 freqtrade 2019.8 中移除。

### `ticker_interval`（现为 `timeframe`）

`ticker_interval` 术语的支持于 2020.6 弃用，改用 `timeframe`——兼容代码于 2022.3 移除。

### 允许按顺序运行多个交易对列表

配置中原有的 `"pairlist"` 部分已被移除，替换为 `"pairlists"`——以列表形式指定交易对列表序列。

旧的配置参数部分（`"pairlist"`）于 2019.11 弃用，于 2020.4 移除。

### 从成交量交易对列表中弃用 bidVolume 和 askVolume

由于只有 quoteVolume 可以在资产之间比较，其他选项（bidVolume、askVolume）于 2020.4 弃用，于 2020.9 移除。

### 使用订单簿步骤确定出场价格

使用 `order_book_min` 和 `order_book_max` 曾允许步进订单簿并尝试找到下一个 ROI 槽位——尝试提前下卖出单。
但由于这会增加风险且无益处，已于 2021.7 出于可维护性目的移除。

### 旧版超参优化模式

使用单独的超参优化文件于 2021.4 弃用，于 2021.9 移除。
请切换到新的[参数化策略](hyperopt.md)以使用新的超参优化接口。

## V2 和 V3 之间的策略变更

逐仓合约 / 做空交易于 2022.4 引入。这需要对配置设置、策略接口等进行重大更改。

我们付出了巨大努力来保持与现有策略的兼容性，因此如果你只想继续在现货市场上使用 freqtrade，无需任何更改。
虽然我们可能在未来的某个时候放弃对当前接口的支持，但我们会单独宣布并有适当的过渡期。

请按照[策略迁移](strategy_migration.md)指南将你的策略迁移到新格式，以开始使用新功能。

### webhooks - 2022.4 的变更

#### `buy_tag` 已重命名为 `enter_tag`

这应该只适用于你的策略以及可能的 webhooks。
我们将保留兼容层 1-2 个版本（因此 `buy_tag` 和 `enter_tag` 仍将工作），但 webhooks 对此的支持将在之后消失。

#### 命名变更

Webhook 术语从 "sell" 改为 "exit"，从 "buy" 改为 "entry"，同时移除了 "webhook"。

* `webhookbuy`、`webhookentry` → `entry`
* `webhookbuyfill`、`webhookentryfill` → `entry_fill`
* `webhookbuycancel`、`webhookentrycancel` → `entry_cancel`
* `webhooksell`、`webhookexit` → `exit`
* `webhooksellfill`、`webhookexitfill` → `exit_fill`
* `webhooksellcancel`、`webhookexitcancel` → `exit_cancel`

## 移除 `populate_any_indicators`

版本 2023.3 移除了 `populate_any_indicators`，改用分开的特征工程和目标方法。请阅读[迁移文档](strategy_migration.md#freqai-strategy)了解完整详情。

## 从配置中移除 `protections`

通过配置设置 `"protections": [],` 来设置保护的功能在发出弃用警告超过 3 年后，于 2024.10 移除。

## hdf5 数据存储

使用 hdf5 作为数据存储于 2024.12 弃用，于 2025.1 移除。我们建议切换到 feather 数据格式。

请在更新前使用 [`convert-data` 子命令](data-download.md#sub-command-convert-data)将你的现有数据转换为受支持的格式之一。

## 通过配置设置高级日志

通过 `--logfile systemd` 和 `--logfile journald` 分别配置 syslog 和 journald 已于 2025.3 弃用。
请改用基于配置的[日志设置](advanced-setup.md#advanced-logging)。

## 移除 edge 模块

edge 模块于 2023.9 弃用，于 2025.6 移除。
edge 的所有功能已移除，配置了 edge 将导致错误。

## 动态资金费率处理的调整

从版本 2025.12 开始，动态资金费率的处理已调整为也支持低至 1h 资金间隔的动态资金费率。
因此，所有支持的合约交易所的标记和资金费率时间周期已更改为 1h。

由于标记和资金费率 K 线的时间周期已更改（通常从 8h 改为 1h）——已下载的数据需要调整或部分重新下载。
你可以重新下载所有内容（`freqtrade download-data [...] --erase` - 警告：可能需要较长时间）——或选择性地下载更新的数据。

### 策略

大多数策略应该不需要调整即可继续正常工作——然而，使用 `@informative("8h", candle_type="funding_rate")` 或类似语法的策略将需要将时间周期切换到 1h。
同样适用于 `dp.get_pair_dataframe(metadata["pair"], "8h", candle_type="funding_rate")`——需要切换到 1h。

freqtrade 将自动调整时间周期并返回 `funding_rates`，尽管时间周期传入错误。它会发出警告——但可能仍会破坏你的策略。

### 选择性数据重新下载

以下脚本应作为示例——你可能需要根据你的需要调整时间周期和交易所！

``` bash
# 清理不再需要的数据
rm user_data/data/<exchange>/futures/*-mark*
rm user_data/data/<exchange>/futures/*-funding_rate*

# 下载新数据（仅需一次即可修复标记和资金费率数据）
freqtrade download-data -t 1h --trading-mode futures --candle-types funding_rate mark [...] --timerange <你拥有其他数据的完整时间范围>

```

上述操作的结果是你的 funding_rates 和 mark 数据将具有 1h 时间周期。
你可以通过 `freqtrade list-data --exchange <yourexchange> --show` 验证。

!!! Note "额外参数"
    上述命令可能需要额外参数，如配置文件或显式的 user_data，如果它们与默认值不同。

**Hyperliquid** 现在是一个特例——不再需要 1h mark 数据——而是使用常规 K 线（此数据从未存在过，与 1h 合约 K 线相同）。由于我们不支持 hyperliquid 的 download-data（它们不提供历史数据）——hyperliquid 用户无需采取任何操作。

## FreqAI 中的 Catboost 模型

CatBoost 模型于版本 2025.12 移除，不再受到积极支持。
如果你有使用 CatBoost 模型的现有机器人，你仍然可以通过从 git 历史中复制/粘贴它们（链接如下）并手动安装 Catboost 库来在自定义模型中使用它们。
然而，我们建议切换到其他受支持的模型库，如 LightGBM 或 XGBoost，以获得更好的支持和未来的兼容性。

* [CatboostRegressor](https://github.com/freqtrade/freqtrade/blob/c6f3b0081927e161a16b116cc47fb663f7831d30/freqtrade/freqai/prediction_models/CatboostRegressor.py)
* [CatboostClassifier](https://github.com/freqtrade/freqtrade/blob/c6f3b0081927e161a16b116cc47fb663f7831d30/freqtrade/freqai/prediction_models/CatboostClassifier.py)
* [CatboostClassifierMultiTarget](https://github.com/freqtrade/freqtrade/blob/c6f3b0081927e161a16b116cc47fb663f7831d30/freqtrade/freqai/prediction_models/CatboostClassifierMultiTarget.py)