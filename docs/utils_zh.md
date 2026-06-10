# 工具子命令

除了实盘交易和模拟运行模式、`backtesting` 和 `hyperopt` 优化子命令以及准备历史数据的 `download-data` 子命令外，机器人还包含一些工具子命令。本节介绍它们。

## 创建用户目录

创建用于存放 freqtrade 文件的目录结构。
还会创建策略和超参优化示例供你入门。
可多次使用——使用 `--reset` 将重置示例策略和超参优化文件为默认状态。

--8<-- "commands/create-userdir.md"

```
├── backtest_results
├── data
├── hyperopt_results
├── hyperopts
│   ├── sample_hyperopt_loss.py
├── notebooks
│   └── strategy_analysis_example.ipynb
├── plot
└── strategies
    └── sample_strategy.py
```

## 创建新配置

创建一个新的配置文件，询问一些对配置重要的选择。

--8<-- "commands/new-config.md"

## 显示配置

显示配置文件（默认情况下敏感值会被遮蔽）。
尤其在使用[拆分配置文件](configuration.md#multiple-configuration-files)或[环境变量](configuration.md#environment-variables)时很有用，此命令将显示合并后的配置。

--8<-- "commands/show-config.md"

## 创建新策略

从类似 SampleStrategy 的模板创建新策略。
文件名将与你的类名一致，不会覆盖现有文件。

结果将位于 `user_data/strategies/<strategyclassname>.py`。

--8<-- "commands/new-strategy.md"

### 使用示例

```bash
freqtrade new-strategy --strategy AwesomeStrategy
```

使用高级模板（填充所有可选函数和方法）：

```bash
freqtrade new-strategy --strategy AwesomeStrategy --template advanced
```

## 列出策略

使用 `list-strategies` 子命令查看特定目录中的所有策略。

此子命令对于查找环境中加载策略的问题很有用：包含错误且加载失败的策略模块以红色打印（LOAD FAILED），而具有重复名称的策略以黄色打印（DUPLICATE NAME）。

--8<-- "commands/list-strategies.md"

## 列出超参损失函数

使用 `list-hyperoptloss` 子命令查看所有可用的超参损失函数。

--8<-- "commands/list-hyperoptloss.md"

## 列出 FreqAI 模型

使用 `list-freqaimodels` 子命令查看所有可用的 freqAI 模型。

--8<-- "commands/list-freqaimodels.md"

## 列出交易所

使用 `list-exchanges` 子命令查看机器人可用的交易所。

--8<-- "commands/list-exchanges.md"

## 列出时间周期

使用 `list-timeframes` 子命令查看交易所可用的时间周期列表。

--8<-- "commands/list-timeframes.md"

## 列出交易对/市场

`list-pairs` 和 `list-markets` 子命令允许查看交易所上可用的交易对/市场。

这些子命令具有相同的用法和相同的可用选项集：

--8<-- "commands/list-pairs.md"

默认只显示活跃的交易对/市场。你可以使用 `-a`/`-all` 选项查看所有交易对/市场的列表，包括不活跃的。

## 测试交易对列表

使用 `test-pairlist` 子命令测试[动态交易对列表](plugins.md#pairlists)的配置。

需要带有指定 `pairlists` 属性的配置。
可用于生成静态交易对列表以供回测/超参优化使用。

--8<-- "commands/test-pairlist.md"

## 转换数据库

`freqtrade convert-db` 可用于将你的数据库从一个系统转换为另一个系统（sqlite → postgres、postgres → 其他 postgres），迁移所有交易、订单和交易对锁定。

--8<-- "commands/convert-db.md"

## Web 服务器模式

!!! Warning "实验性"
    Web 服务器模式是提高回测和策略开发效率的实验性模式。

以 Web 服务器模式运行 freqtrade。
Freqtrade 将启动 Web 服务器并允许 FreqUI 启动和控制回测过程。
其优势在于回测运行之间不会重新加载数据（只要时间周期和时间范围保持不变）。
FreqUI 也将显示回测结果。

--8<-- "commands/webserver.md"

## 显示之前的回测结果

允许你显示之前的回测结果。
添加 `--show-pair-list` 输出一个排序后的交易对列表，你可以轻松复制/粘贴到配置中（省略不好的交易对）。

--8<-- "commands/backtesting-show.md"

## 详细回测分析

高级回测结果分析。

更多详情见[回测分析](advanced-backtesting.md#analyze-the-buyentry-and-sellexit-tags)部分。

--8<-- "commands/backtesting-analysis.md"

## 列出超参优化结果

你可以使用 `hyperopt-list` 子命令列出 Hyperopt 模块之前评估的超参优化轮次。

--8<-- "commands/hyperopt-list.md"

### 示例

列出所有结果，最后打印最佳结果的详细信息：
```
freqtrade hyperopt-list
```

仅列出有正利润的轮次：
```
freqtrade hyperopt-list --profitable --no-details
```

## 显示超参优化结果详情

你可以使用 `hyperopt-show` 子命令显示 Hyperopt 模块之前评估的任何超参优化轮次的详细信息。

--8<-- "commands/hyperopt-show.md"

## 显示交易记录

将数据库中选定（或全部）交易打印到屏幕。

--8<-- "commands/show-trades.md"

## 策略更新器

将列出的策略或策略文件夹中的所有策略更新为 v3 兼容。
原始策略将保留在 `user_data/strategies_orig_updater/` 目录中。

--8<-- "commands/strategy-updater.md"