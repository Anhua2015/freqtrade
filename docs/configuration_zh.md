# 配置机器人

Freqtrade 有许多可配置的功能和可能性。
默认情况下，这些设置通过配置文件进行配置（见下文）。

## Freqtrade 配置文件

机器人在运行期间使用一组共同构成机器人配置的配置参数。它通常从文件（Freqtrade 配置文件）中读取配置。

默认情况下，机器人从位于当前工作目录的 `config.json` 文件中加载配置。

你可以通过 `-c/--config` 命令行选项指定一个不同的配置文件供机器人使用。

如果你使用[快速入门](docker_quickstart.md#docker-quick-start)方法安装机器人，安装脚本应该已经为你创建了默认配置文件（`config.json`）。

如果默认配置文件未创建，我们建议使用 `freqtrade new-config --config user_data/config.json` 生成一个基本配置文件。

Freqtrade 配置文件以 JSON 格式编写。

除了标准的 JSON 语法外，你可以在配置文件中使用单行 `// ...` 和多行 `/* ... */` 注释，以及参数列表中的尾随逗号。

如果你不熟悉 JSON 格式，不用担心——只需用你选择的编辑器打开配置文件，对你需要的参数进行一些更改，保存更改，最后重启机器人，或者如果之前已停止，使用你对配置所做的更改再次运行它。机器人在启动时验证配置文件的语法，如果你在编辑时犯了任何错误，会警告你并指出有问题的行。

### 环境变量

通过环境变量在 Freqtrade 配置中设置选项。
这优先于配置或策略中的对应值。

环境变量必须以 `FREQTRADE__` 为前缀才能加载到 Freqtrade 配置中。

`__` 作为层级分隔符，因此使用的格式应对应于 `FREQTRADE__{section}__{key}`。
因此——定义为 `export FREQTRADE__STAKE_AMOUNT=200` 的环境变量将导致 `{stake_amount: 200}`。

一个更复杂的例子可能是 `export FREQTRADE__EXCHANGE__KEY=<yourExchangeKey>` 来保密你的交易所密钥。这将值移动到配置的 `exchange.key` 部分。
使用此方案，所有配置设置也将作为环境变量可用。

请注意，环境变量将覆盖你配置中的相应设置，但命令行参数始终优先级最高。

常见示例：

``` bash
FREQTRADE__TELEGRAM__CHAT_ID=<telegramchatid>
FREQTRADE__TELEGRAM__TOKEN=<telegramToken>
FREQTRADE__EXCHANGE__KEY=<yourExchangeKey>
FREQTRADE__EXCHANGE__SECRET=<yourExchangeSecret>
```

JSON 列表按 JSON 解析——因此你可以使用以下方式设置交易对列表：

``` bash
export FREQTRADE__EXCHANGE__PAIR_WHITELIST='["BTC/USDT", "ETH/USDT"]'
```

!!! Note
    检测到的环境变量在启动时被记录——因此，如果你找不到为什么某个值与基于配置的预期不符，请确保它不是从环境变量加载的。

!!! Tip "验证合并结果"
    你可以使用 [show-config 子命令](utils.md#show-config)查看最终的合并配置。

### 多配置文件

可以指定多个配置文件供机器人使用，或机器人可以从进程标准输入流读取其配置参数。

你可以在 `add_config_files` 中指定额外的配置文件。在此参数中指定的文件将被加载并与初始配置文件合并。这些文件相对于初始配置文件进行解析。
这类似于使用多个 `--config` 参数，但使用更简单，因为你不需要为所有命令指定所有文件。

!!! Tip "使用多个配置文件保密"
    你可以使用第二个配置文件来包含你的密钥。这样你可以分享你的"主"配置文件，同时保留你的 API 密钥。
    第二个文件应仅指定你打算覆盖的内容。
    如果一个键出现在多个配置中，则"最后指定的配置"优先。

    对于一次性命令，你也可以通过指定多个 "--config" 参数使用以下语法：

    ``` bash
    freqtrade trade --config user_data/config1.json --config user_data/config-private.json <...>
    ```

### 配置参数表

必选参数标记为 **Required**，意味着必须通过某种方式设置。

|  参数 | 描述 |
|------------|-------------|
| `max_open_trades` | **必选。** 机器人允许拥有的未平仓交易数量。每个交易对只能有一笔未平仓交易，因此交易对列表的长度是另一个可能的限制。如果为 -1 则忽略（即可无限制地开仓，受交易对列表限制）。[策略可覆盖](#parameters-in-the-strategy)。<br> **数据类型：** 正整数或 -1。 |
| `stake_currency` | **必选。** 用于交易的加密货币。<br> **数据类型：** 字符串 |
| `stake_amount` | **必选。** 机器人为每笔交易使用的加密货币数量。设置为 `"unlimited"` 允许机器人使用所有可用余额。<br> **数据类型：** 正浮点数或 `"unlimited"`。 |
| `tradable_balance_ratio` | 允许机器人交易的总账户余额比例。<br>*默认为 `0.99`（99%）。*<br> **数据类型：** 介于 `0.1` 和 `1.0` 之间的正浮点数。 |
| `available_capital` | 机器人的可用起始资金。在同一交易所账户上运行多个机器人时有用。<br> **数据类型：** 正浮点数。 |
| `timeframe` | 使用的时间周期（例如 `1m`、`5m`、`15m`、`30m`、`1h` ...）。通常在配置中缺失，在策略中指定。[策略可覆盖](#parameters-in-the-strategy)。<br> **数据类型：** 字符串 |
| `fiat_display_currency` | 用于显示利润的法币货币。<br> **数据类型：** 字符串 |
| `dry_run` | **必选。** 定义机器人必须在模拟模式还是生产模式下运行。<br>*默认为 `true`。*<br> **数据类型：** 布尔值 |
| `dry_run_wallet` | 定义机器人在模拟模式下使用的模拟钱包的起始金额（以投入货币计）。<br>*默认为 `1000`。*<br> **数据类型：** 浮点数或字典 |
| `minimal_roi` | **必选。** 设置机器人用于平仓的阈值比率。[策略可覆盖](#parameters-in-the-strategy)。<br> **数据类型：** 字典 |
| `stoploss` | **必选。** 机器人使用的止损比率值。详见[止损文档](stoploss.md)。[策略可覆盖](#parameters-in-the-strategy)。<br> **数据类型：** 浮点数（比率） |
| `trading_mode` | 指定是常规交易、杠杆交易还是交易合约。[杠杆文档](leverage.md)。<br>*默认为 `"spot"`。*<br> **数据类型：** 字符串 |
| `margin_mode` | 杠杆交易时，决定交易者拥有的抵押品是共享的还是每个交易对独立的。[杠杆文档](leverage.md)。<br> **数据类型：** 字符串 |
| `liquidation_buffer` | 指定在清算价和止损之间放置多大的安全缓冲，以防仓位达到清算价。[杠杆文档](leverage.md)。<br>*默认为 `0.05`。*<br> **数据类型：** 浮点数 |
| `unfilledtimeout.entry` | **必选。** 机器人等待未成交入场订单完成的时间（分钟或秒），之后订单将被取消。[策略可覆盖](#parameters-in-the-strategy)。<br> **数据类型：** 整数 |
| `unfilledtimeout.exit` | **必选。** 机器人等待未成交出场订单完成的时间，之后订单被取消并以当前（新）价格重新下单，只要有信号。[策略可覆盖](#parameters-in-the-strategy)。<br> **数据类型：** 整数 |
| `entry_pricing.price_side` | 选择机器人查看价差哪一侧来获取入场价格。<br>*默认为 `"same"`。*<br> **数据类型：** 字符串（`ask`、`bid`、`same` 或 `other`）。 |
| `exit_pricing.price_side` | 选择机器人查看价差哪一侧来获取出场价格。<br>*默认为 `"same"`。*<br> **数据类型：** 字符串（`ask`、`bid`、`same` 或 `other`）。 |
| `use_exit_signal` | 除 `minimal_roi` 外，还使用策略产生的出场信号。[策略可覆盖](#parameters-in-the-strategy)。<br>*默认为 `true`。*<br> **数据类型：** 布尔值 |
| `position_adjustment_enable` | 允许策略使用仓位调整（额外买入或卖出）。[策略可覆盖](#parameters-in-the-strategy)。<br>*默认为 `false`。*<br> **数据类型：** 布尔值 |
| `exchange.name` | **必选。** 要使用的交易所类名。<br> **数据类型：** 字符串 |
| `exchange.key` | 交易所的 API 密钥。仅在生产模式下需要。<br>**请保密，不要公开披露。**<br> **数据类型：** 字符串 |
| `exchange.secret` | 交易所的 API 密钥。仅在生产模式下需要。<br>**请保密，不要公开披露。**<br> **数据类型：** 字符串 |
| `exchange.pair_whitelist` | 机器人用于交易和回测期间检查潜在交易的交易对列表。支持正则表达式交易对如 `.*/BTC`。<br> **数据类型：** 列表 |
| `exchange.pair_blacklist` | 机器人在交易和回测中必须绝对避免的交易对列表。<br> **数据类型：** 列表 |

### 策略中的参数

以下参数可以在配置文件或策略中设置。
配置文件中设置的值始终覆盖策略中设置的值。

* `minimal_roi`、`timeframe`、`stoploss`、`max_open_trades`
* `trailing_stop`、`trailing_stop_positive`、`trailing_stop_positive_offset`、`trailing_only_offset_is_reached`
* `use_custom_stoploss`、`process_only_new_candles`
* `order_types`、`order_time_in_force`、`unfilledtimeout`
* `disable_dataframe_checks`、`use_exit_signal`
* `exit_profit_only`、`exit_profit_offset`、`ignore_roi_if_entry_signal`
* `position_adjustment_enable`、`max_entry_position_adjustment`

### 配置每笔交易的投入金额

有几种方法可以配置机器人为每笔交易使用的投入货币量。所有方法都遵守下文所述的[可用余额配置](#tradable-balance)。

#### 静态投入金额

`stake_amount` 配置静态地设置机器人为每笔交易使用的投入货币量。

最小配置值为 0.0001，但请检查你使用的投入货币在交易所的交易最小值以避免问题。

此设置与 `max_open_trades` 配合使用。交易中占用的最大资金为 `stake_amount * max_open_trades`。

#### 动态投入金额

或者，你可以使用动态投入金额，它将使用交易所的可用余额，并按允许的交易数量（`max_open_trades`）平均分配。

要配置此功能，设置 `stake_amount="unlimited"`。我们还建议设置 `tradable_balance_ratio=0.99`（99%）——保留最低余额用于可能的手续费。

在此情况下，交易金额计算为：

```python
currency_balance / (max_open_trades - current_open_trades)
```

!!! Tip "复利"
    此配置将允许根据机器人的表现增加/减少投入（机器人亏损时投入较低，盈利时投入较高，因为有更高的余额可用），并导致利润复利。

### 理解 minimal_roi

`minimal_roi` 配置参数是一个 JSON 对象，其中键是以分钟为单位的持续时间，值是以比率表示的最低 ROI。
参见以下示例：

```json
"minimal_roi": {
    "40": 0.0,    # 40 分钟后如果利润不为负则平仓
    "30": 0.01,   # 30 分钟后如果至少有 1% 利润则平仓
    "20": 0.02,   # 20 分钟后如果至少有 2% 利润则平仓
    "0":  0.04    # 如果至少有 4% 利润则立即平仓
},
```

!!! Note "特殊情况：在特定时间后强制平仓"
    使用 `"<N>": -1` 作为 ROI 是一种特殊情况。这强制机器人在 N 分钟后平仓，无论盈利还是亏损，代表有时间限制的强制平仓。

### 理解 order_types

`order_types` 配置参数将操作（`entry`、`exit`、`stoploss`、`emergency_exit`、`force_exit`、`force_entry`）映射到订单类型（`market`、`limit` 等），并配置止损是否在交易所以及定义交易所止损更新间隔秒数。

策略语法：

```python
order_types = {
    "entry": "limit",
    "exit": "limit",
    "emergency_exit": "market",
    "force_entry": "market",
    "force_exit": "market",
    "stoploss": "market",
    "stoploss_on_exchange": False,
    "stoploss_on_exchange_interval": 60,
    "stoploss_on_exchange_limit_ratio": 0.99,
}
```

### 使用模拟模式

我们建议在模拟模式下启动机器人，以查看机器人的行为以及策略的表现。在模拟模式下，机器人不涉及你的资金。它只运行实时模拟，不在交易所上创建交易。

```json
"dry_run": true,
"db_url": "sqlite:///tradesv3.dryrun.sqlite",
```

### 切换到生产模式

在生产模式下，机器人将使用你的资金。请小心，因为错误的策略可能会让你损失所有资金。

```json
"dry_run": false,
```

**插入你的交易所 API 密钥：**

```json
{
    "exchange": {
        "name": "binance",
        "key": "af8ddd35195e9dc500b9a6f799f6f5c93d89193b",
        "secret": "08a9dc6db3d7b53e1acebd9275677f4b0a04f1a5",
    }
}
```

!!! Hint "保持密钥安全"
    为保密，我们建议使用第二个配置文件存放 API 密钥。
    **绝不**与任何人分享你的私有配置文件或交易所密钥！

## 下一步

现在你已经配置了 config.json，下一步是[启动你的机器人](bot-usage.md)。