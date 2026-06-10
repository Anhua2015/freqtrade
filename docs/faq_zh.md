# Freqtrade FAQ

## 支持的市场

Freqtrade 支持现货交易，以及部分选定交易所的合约交易。请参考[文档首页](index.md#supported-futures-exchanges)获取最新的支持交易所列表。

### 我的机器人可以开空头仓位吗？

Freqtrade 可以在合约市场中开空头仓位。
这需要策略为此而设计——并在配置中设置 `"trading_mode": "futures"`。
请务必先阅读[相关文档页面](leverage.md)。

在现货市场中，在某些情况下你可以使用杠杆现货代币，它们反映反向交易对（例如 BTCUP/USD、BTCDOWN/USD、ETHBULL/USD、ETHBEAR/USD 等），可以用 Freqtrade 交易。

### 我的机器人可以交易期权或期货吗？

合约交易支持部分选定的交易所。请参考[文档首页](index.md#supported-futures-exchanges)获取最新的支持交易所列表。

## 新手提示和技巧

* 在编写策略和超参优化文件时，应使用合适的代码编辑器，如 VSCode 或 PyCharm。好的代码编辑器提供语法高亮和行号，便于查找语法错误（通常由 Freqtrade 在启动时指出）。

## Freqtrade 常见问题

### Freqtrade 可以在同一交易对上同时开多个仓位吗？

不可以。Freqtrade 每次每个交易对只能开一个仓位。
但是，你可以使用 [`adjust_trade_position()` 回调](strategy-callbacks.md#adjust-trade-position)来调整已开仓位。

回测中通过 `--eps` 提供了此选项——但这仅用于突出显示"隐藏"信号，在实盘中不可用。

### Freqtrade 支持沙箱账户吗？

不支持，但你可以使用模拟模式来模拟交易，无需冒真实资金的风险。

沙箱市场是独立的模拟市场——不适合在真实环境中测试你的策略。
这些市场通常有不同的订单簿、流动性和交易行为（通常参与者很少）——这使它们不适合真实测试你的策略。

### 机器人无法启动

使用 `freqtrade trade --config config.json` 运行机器人时显示 `freqtrade: command not found`。

可能由以下原因造成：

* 虚拟环境未激活。
  * 运行 `source .venv/bin/activate` 激活虚拟环境。
* 安装未成功完成。
  * 请检查[安装文档](installation.md)。

### 机器人启动了，但处于 STOPPED 模式

确保在 config.json 中将 `initial_state` 配置选项设置为 `"running"`。

### 我已经等了 5 分钟，为什么机器人还没有进行任何交易？

* 根据入场策略、白名单币种数量、市场状况等，可能需要数小时甚至数天才能找到好的入场点位。请耐心！
* 回测会大致告诉你预期有多少交易——但这不能保证它们在时间上均匀分布——你可能一天有 20 笔交易，而本周其余时间为 0。
* 可能是配置错误。最好检查日志，它们通常告诉你机器人是否只是没有收到买入信号（仅有心跳消息），或者是否有问题（日志中有错误/异常）。

### 我已经做了 12 笔交易，为什么总利润是负的？

我理解你的失望，但很不幸，12 笔交易不足以说明任何问题。如果你运行回测，你会看到当前算法确实让你处于盈利状态，但这是经过数千笔交易之后的，即使在那时，你在某些特定币种上交易了数十次甚至数百次后仍会有亏损。我们当然在不断努力改进机器人，但它将*始终*是一种赌博，应该给你带来月度温和的收益，但你不能从少数几笔交易中得出太多结论。

### 我想对配置进行更改，可以不杀进程吗？

可以。你可以编辑配置并使用 `/reload_config` 命令重新加载配置。机器人将停止，重新加载配置和策略，然后以新配置和策略重新启动。

### 为什么我的机器人没有卖出它买入的全部？

这被称为"粉尘"，所有交易所都可能发生。
因为许多交易所从"接收货币"中扣除手续费——所以你买入 100 个 COIN——但你只得到 99.9 个 COIN。
由于 COIN 以整手数量（1 COIN 为单位）交易，你无法卖出 0.9 COIN（或 99.9 COIN）——而需要向下取整到 99 COIN。

这不是机器人的问题，手动交易也会发生。

虽然 freqtrade 可以处理这种情况（它会卖出 99 COIN），但手续费通常低于最小可交易数量（你只能交易整 COIN，不能交易 0.9 COIN）。
将粉尘（0.9 COIN）留在交易所通常是有意义的，因为下次 freqtrade 买入 COIN 时，它会消耗剩余的小额余额，这次卖出所有买入的部分，从而缓慢减少粉尘余额（尽管很可能永远不会精确达到 0）。

在可能的情况下（例如币安），使用交易所专用的手续费货币可以解决此问题。
在币安上，只需账户中有 BNB，并在个人资料中启用"使用 BNB 支付手续费"。你的 BNB 余额会缓慢减少（用于支付手续费）——但你不会再遇到粉尘问题（Freqtrade 会将手续费包含在利润计算中）。
其他交易所不提供此类可能性，你只能接受或换到其他交易所。

### 我向交易所存入了更多资金，但机器人没有识别到

Freqtrade 会在必要时更新交易所余额（在下单之前）。
RPC 调用（Telegram 的 `/balance`、API 调用 `/balance`）最多每小时触发一次更新。

如果启用了 `adjust_trade_position`（且机器人有符合条件的未平仓交易可进行仓位调整）——那么钱包将每小时刷新一次。
要强制立即更新，可以使用 `/reload_config`——这将重启机器人。

### 我想使用不完整的 K 线

Freqtrade 不会向策略提供不完整的 K 线。使用不完整的 K 线会导致重绘，从而导致策略出现"幽灵"买入，这些在回测和发生后验证都是不可能的。

你可以通过使用 [dataprovider](strategy-customization.md#orderbookpair-maximum) 的订单簿或 ticker 方法来使用"当前"市场数据——但这些在回测期间不能使用。

### 有没有设置只平掉持有的交易而不开新仓？

你可以使用 Telegram 中的 `/stopentry` 命令阻止未来的交易入场，然后使用 `/forceexit all`（卖出所有未平仓交易）。

### 我卖掉了机器人的资金，现在日志中有错误

Freqtrade 假设它开立的交易仅由机器人管理。
如果你（不小心）卖掉了机器人的资金，freqtrade 将尝试通过重新查找交易所订单来恢复。

这是尽力而为的方式，并非所有情况都适用，尤其是在使用 freqtrade 不支持的订单类型（OCO、冰山单等）时，或处理较旧的交易时（交易所不再提供完整订单信息）。
具体限制因交易所而异——详情通常记录在交易所的 API 文档中。

### 我想在同一台机器上运行多个机器人

请查看[高级设置文档页面](advanced-setup.md#running-multiple-instances-of-freqtrade)。

### 启动机器人时出现 "Impossible to load Strategy"

此错误消息表示机器人无法加载策略。
通常，你可以使用 `freqtrade list-strategies` 列出所有可用策略。
此命令的输出还将包含一个状态列，显示策略是否可以加载。

请检查以下内容：

* 你使用的策略名称是否正确？策略名称区分大小写，必须与策略类名（不是文件名！）对应。
* 策略是否在 `user_data/strategies` 目录中，且文件扩展名为 `.py`？
* 机器人在此错误之前是否显示了其他警告？可能缺少策略的某些依赖项——这将在日志中突出显示。
* 对于 docker——策略目录是否正确挂载（检查 docker-compose 文件的 volumes 部分）？

### 日志中出现 "Missing data fillup" 消息

此消息只是一个警告，表示最新的 K 线中存在缺失。
根据交易所不同，这可能表示该交易对在你使用的时间周期内没有交易——而交易所只返回有成交量的 K 线。
在低成交量交易对中，这是相当常见的情况。

如果交易对列表中的所有交易对都发生这种情况，可能表示交易所最近宕机。请检查交易所的公共渠道了解详情。

无论原因如何，Freqtrade 将用"空"K 线填充这些缺失的 K 线，其中开盘、最高、最低和收盘设为前一 K 线的收盘价——成交量为空。在图表中，这看起来像 `_`——与交易所通常表示零成交量 K 线的方式一致。

### 日志中出现 "Price jump between 2 candles detected"

此消息是一个警告，表示 K 线之间的价格跳变超过 30%。
这可能是该交易对停止交易的信号，并且发生了某种代币置换（例如 2021 年的 COCOS——价格从 0.0000154 跳变到 0.01621）。
此消息通常伴随 ["Missing data fillup"](#im-getting-missing-data-fillup-messages-in-the-log)——因为此类交易对通常会停止交易一段时间。

### 我想重置机器人的数据库

要重置机器人数据库，你可以删除数据库（默认为 `tradesv3.sqlite` 或 `tradesv3.dryrun.sqlite`），或通过 `--db-url` 使用不同的数据库 URL（例如 `sqlite:///mynewdatabase.sqlite`）。

### 日志中出现 "Outdated history for pair xxx"

机器人试图告诉你它收到了过时的最后一根 K 线（不是最后一根完整 K 线）。
因此，Freqtrade 不会对该交易对进行交易——因为基于旧信息交易通常不是所期望的。

此警告可能指向以下问题之一：

* 交易所宕机 → 检查交易所状态页面 / 博客 / Twitter 了解详情。
* 系统时间错误 → 确保你的系统时间正确。
* 极少交易的交易对 → 在交易所网页上检查该交易对，查看策略使用的时间周期。如果该交易对在某些 K 线上没有任何成交量（通常以"成交量 0"柱和"_"K 线呈现），则该交易对在此时间周期内没有任何交易。理想情况下应避免这些交易对，因为它们可能导致订单成交问题。
* API 问题 → API 返回错误数据（此处仅出于完整性，不应该发生在受支持的交易所）。

### 日志中收到 "Couldn't reuse watch for xxx" 消息

这是一条信息性消息，表示机器人尝试使用 WebSocket 的 K 线，但交易所没有提供正确的信息。
这可能发生在 WebSocket 连接中断时——或该交易对在你使用的时间周期内没有任何交易发生时。

Freqtrade 将通过回退到 REST API 优雅地处理此情况。
虽然这使得迭代略慢（由于 REST API 调用）——但不会对机器人的运行产生任何问题。

### 我收到 "Exchange XXX does not support market orders." 消息且无法运行策略

如消息所述，你的交易所不支持市价单，而你设置了某个[订单类型](configuration.md/#understand-order_types)为 "market"。你的策略可能是针对其他交易所编写的，并将 "stoploss" 订单设为 "market"——这对于大多数支持市价单的交易所是正确的且更可取（但不适用于 Gate.io）。

要修复此问题，在策略中将订单类型重新定义为使用 "limit" 而非 "market"：

``` python
    order_types = {
        ...
        "stoploss": "limit",
        ...
    }
```

如果在自定义配置而非策略中定义了订单类型，同样的修复应应用于配置文件。

### 我想实盘启动机器人，但收到 API 权限错误

像 `Invalid API-key, IP, or permissions for action` 这样的错误正如字面意思所示。
你的 API 密钥要么无效（复制/粘贴错误？检查配置中是否有前导/尾随空格）、已过期，或者你运行机器人的 IP 不在交易所的 API 控制台中启用。
通常需要 "Spot Trading"（或你使用的交易所中的等效权限）权限。
合约通常需要单独启用。

### 如何在机器人日志中搜索某些内容？

默认情况下，机器人将日志写入 stderr 流。这样实现是为了让你可以轻松地将机器人的诊断消息与回测、Edge 和超参优化结果、其他各种 Freqtrade 实用子命令的输出以及你在策略中可能插入的自定义 `print()` 输出分开。因此，如果你需要使用 grep 工具搜索日志消息，需要将 stderr 重定向到 stdout 并丢弃 stdout。

* 在 unix shell 中，通常可以这样做：
```shell
$ freqtrade --some-options 2>&1 >/dev/null | grep 'something'
```
（注意，`2>&1` 和 `>/dev/null` 应按此顺序书写）

* Bash 解释器也支持所谓的进程替换语法，你可以这样 grep 日志：
```shell
$ freqtrade --some-options 2> >(grep 'something') >/dev/null
```
或
```shell
$ freqtrade --some-options 2> >(grep -v 'something' 1>&2)
```

* 你也可以使用 `--logfile` 选项将 Freqtrade 日志消息的副本写入文件：
```shell
$ freqtrade --logfile /path/to/mylogfile.log --some-options
```
然后这样 grep：
```shell
$ cat /path/to/mylogfile.log | grep 'something'
```
或甚至在机器人运行时实时查看：
```shell
$ tail -f /path/to/mylogfile.log | grep 'something'
```
在单独的终端窗口中运行。

在 Windows 上，Freqtrade 也支持 `--logfile` 选项，你可以使用 `findstr` 命令在日志中搜索感兴趣的字符串：
```
> type \path\to\mylogfile.log | findstr "something"
```

## 超参优化模块

### 为什么 freqtrade 没有 GPU 支持？

首先，大多数指标库不支持 GPU——因此，指标计算几乎没有收益。
GPU 改进仅适用于 pandas 原生的计算——或你自己编写的计算。

GPU 仅擅长数字运算（浮点运算）。
对于超参优化，我们既需要数字运算（寻找下一个参数），也需要运行 Python 代码（运行回测）。
因此，GPU 不太适合超参优化的大部分工作。

使用 GPU 的收益因此非常微薄——不值得为添加 GPU 支持而引入的复杂性。

然而，如果你认为必须有 GPU 加速的指标，没有什么能阻止你在策略中使用它们——但你可能会对微薄的收益感到失望（相对于复杂性而言）。

### 我需要多少轮才能得到好的超参优化结果？

默认情况下，不带 `-e`/`--epochs` 命令行选项调用 Hyperopt 将仅运行 100 轮，意味着对你的触发器、守卫等进行 100 次评估。太少，无法找到好的结果（除非你非常幸运），所以你很可能需要运行 10000 轮或更多。但这将需要漫长的计算时间。

由于超参优化使用贝叶斯搜索，运行过多轮可能不会产生更好的结果。

因此建议反复运行 500-1000 轮，直到总共达到至少 10000 轮（或对结果满意）。你可以通过查看结果来判断——如果机器人不断发现更好的策略，最好继续。

```bash
freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --strategy SampleStrategy -e 1000
```

### 为什么超参优化需要很长时间？

* 通过 Hyperopt 发现一个好的策略需要时间。学习 www.freqtrade.io、Freqtrade 文档页面，加入 Freqtrade [discord 社区](https://discord.gg/p7nuUNVfP7)。耐心等待世界上最先进的免费加密机器人为你交付一个可能专门为你设计的黄金策略。

* 如果你想知道为什么 1000 轮可能需要 20 分钟到几天的时间，以下是一些答案：

这个答案是在 0.15.1 版本发布时编写的，当时有：

* 8 个触发器
* 9 个守卫：假设我们每个评估 10 个值
* 1 个止损计算：假设我们也想评估 10 个值

以下计算仍然非常粗略且不太精确，但能给出概念。仅这些触发器和守卫已经有 8\*10^9\*10 次评估。总计约 800 亿次评估。
你运行了 100,000 次评估？恭喜，你完成了搜索空间的大约 1/100,000，假设机器人从不测试相同的参数超过一次。

* 运行 1000 轮超参优化所需的时间取决于以下因素：可用 CPU、硬盘、内存、时间周期、时间范围、指标设置、指标数量、超参优化测试策略的币种数量以及最终产生的交易数量——这可能是每年 650 笔交易或 100,000 笔交易，取决于策略是追求高利润的稀有交易还是许多低利润交易。

示例：一年内 650 次 4% 利润 vs 10000 次 0.3% 利润的交易。假设你设置 --timerange 为 365 天。

示例：
`freqtrade --config config.json --strategy SampleStrategy --hyperopt SampleHyperopt -e 1000 --timerange 20190601-20200601`

## 官方渠道

Freqtrade 仅使用以下官方渠道：

* [Freqtrade discord 服务器](https://discord.gg/p7nuUNVfP7)
* [Freqtrade 文档 (https://freqtrade.io)](https://freqtrade.io)
* [Freqtrade github 组织](https://github.com/freqtrade)

与 Freqtrade 项目相关的任何人都不会向你索要交易所密钥或任何会使你的资金面临利用风险的东西。
如果有人要求你暴露交易所密钥或向某个随机钱包发送资金，请不要遵循这些指示。

不遵守这些准则的责任不由 Freqtrade 承担。

## 支持政策

我们在 [Discord 服务器](https://discord.gg/p7nuUNVfP7)和 GitHub issues 上为 Freqtrade 提供免费支持。
我们只支持最新版本（例如 2025.8）和当前的开发分支（例如 2025.9-dev）。

如果你使用的是旧版本，请按照[升级说明](updating.md)进行升级，看看你的问题是否已经被解决。

## "Freqtrade 代币"

Freqtrade 没有任何加密货币代币发售。

你在互联网上找到的提及 Freqtrade、FreqAI 或 freqUI 的代币发售必须被视为骗局，试图利用 Freqtrade 的知名度为自己牟取不正当利益。