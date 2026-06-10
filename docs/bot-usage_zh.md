# 启动机器人

本页介绍机器人的各项参数及运行方式。

!!! Note
    如果你使用了 `setup.sh`，别忘了在运行 freqtrade 命令之前激活你的虚拟环境（`source .venv/bin/activate`）。

!!! Warning "时钟同步"
    运行机器人的系统时钟必须准确，并需频繁同步至 NTP 服务器，以避免与交易所通信时出现问题。

## 机器人命令

--8<-- "commands/main.md"

### 机器人交易命令

--8<-- "commands/trade.md"

### 如何指定要使用的配置文件？

机器人允许你通过 `-c/--config` 命令行选项来选择使用哪个配置文件：

```bash
freqtrade trade -c path/far/far/away/config.json
```

默认情况下，机器人会从当前工作目录加载 `config.json` 配置文件。

### 如何使用多个配置文件？

机器人允许你通过在命令行中指定多个 `-c/--config` 选项来使用多个配置文件。
后续配置文件中定义的参数会覆盖之前命令行中较早指定的配置文件中同名参数的值。

例如，你可以单独创建一个包含交易所 key 和 secret 的配置文件，而默认配置文件中 key 和 secret 留空，在模拟模式下运行（模拟模式实际上不需要它们）：

```bash
freqtrade trade -c ./config.json
```

并在实盘交易模式下同时指定两个配置文件：

```bash
freqtrade trade -c ./config.json -c path/to/secrets/keys.config.json
```

这样可以帮助你在本地机器上隐藏私有交易所 key 和 secret，只需为包含实际密钥的文件设置适当的文件权限即可，此外还能防止在项目 issue 或网上发布配置示例时意外泄露敏感私密数据。

更多关于此技术及示例详见 [配置](configuration.md) 文档页面。

### 自定义数据存放位置

Freqtrade 允许使用 `freqtrade create-userdir --userdir someDirectory` 创建用户数据目录。
此目录结构如下：

```
user_data/
├── backtest_results
├── data
├── hyperopts
├── hyperopt_results
├── plot
└── strategies
```

你可以在配置中添加 `user_data_dir` 设置项，让机器人始终指向此目录。
或者，每次执行命令时传入 `--userdir`。
如果目录不存在，机器人将无法启动，但会自动创建必要的子目录。

此目录应包含你的自定义策略、自定义超参优化和超参损失函数、回测历史数据（使用回测命令或下载脚本下载）以及图表输出。

建议使用版本控制来跟踪策略的变更。

### 如何使用 **--strategy**？

此参数允许你加载自定义策略类。
要测试机器人安装，可以使用 `create-userdir` 子命令安装的 `SampleStrategy`（通常位于 `user_data/strategy/sample_strategy.py`）。

机器人会在 `user_data/strategies` 中搜索你的策略文件。
要使用其他目录，请参阅下一节关于 `--strategy-path` 的内容。

要加载策略，只需在此参数中传入类名（例如 `CustomStrategy`）。

**示例：**
在 `user_data/strategies` 中有一个名为 `my_awesome_strategy.py` 的文件，其中包含一个名为 `AwesomeStrategy` 的策略类，要加载它：

```bash
freqtrade trade --strategy AwesomeStrategy
```

如果机器人找不到你的策略文件，它会在错误消息中显示原因（文件未找到，或代码中存在错误）。

了解更多关于策略文件的信息，请参阅
[策略定制](strategy-customization.md)。

### 如何使用 **--strategy-path**？

此参数允许你添加一个额外的策略查找路径，该路径会在默认位置之前被检查（传入的路径必须是目录！）：

```bash
freqtrade trade --strategy AwesomeStrategy --strategy-path /some/directory
```

#### 如何安装一个策略？

非常简单。将你的策略文件复制粘贴到 `user_data/strategies` 目录中，或使用 `--strategy-path`。然后，机器人就可以使用了。

### 如何使用 **--db-url**？

在模拟模式下运行机器人时，默认不会将任何交易记录存储到数据库中。如果你想将机器人的操作存储到数据库中，可以使用 `--db-url`。
这也可以用于在生产模式下指定自定义数据库。示例命令：

```bash
freqtrade trade -c config.json --db-url sqlite:///tradesv3.dry_run.sqlite
```

## 下一步

机器人的最优策略会随着市场趋势的变化而变化。下一步是
[策略定制](strategy-customization.md)。