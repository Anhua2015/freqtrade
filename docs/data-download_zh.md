# 数据下载

## 获取回测和超参优化所需的数据

要下载回测和超参优化所需的 K 线 / OHLCV 数据，请使用 `freqtrade download-data` 命令。

如果未指定额外参数，freqtrade 将下载最近 30 天的 `"1m"` 和 `"5m"` 时间周期数据。
交易所和交易对来源于 `config.json`（如果通过 `-c/--config` 指定）。
如果没有提供配置文件，`--exchange` 变为必选。

你可以使用相对时间范围（`--days 20`）或绝对起始点（`--timerange 20200101-`）。对于增量下载，应使用相对方式。

!!! Tip "提示：更新现有数据"
    如果你的数据目录中已有回测数据并希望将其刷新到今天，freqtrade 将自动计算现有交易对的缺失时间范围，下载将从最近可用点进行直到"现在"，不需要 `--days` 或 `--timerange` 参数。Freqtrade 将保留可用数据，仅下载缺失的数据。
    如果在插入你没有数据的新交易对后更新现有数据，请使用 `--new-pairs-days xx` 参数。指定的天数将为新交易对下载，而旧交易对将仅更新缺失的数据。

### 用法

--8<-- "commands/download-data.md"

!!! Tip "下载一种计价货币的所有数据"
    通常，你希望下载特定计价货币的所有交易对数据。在这种情况下，你可以使用以下简写：
    `freqtrade download-data --exchange binance --pairs ".*/USDT" <...>`。提供的 "pairs" 字符串将被扩展为包含交易所上所有活跃的交易对。
    要同时下载非活跃（已下架）交易对的数据，请在命令中添加 `--include-inactive-pairs`。

!!! Note "启动期"
    `download-data` 是一个与策略无关的命令。其思想是一次下载大量数据，然后迭代地增加存储的数据量。

    因此，`download-data` 不关心策略中定义的"启动期"。如果回测应在特定时间点开始（同时考虑启动期），用户需要自行下载额外的天数。

### 开始下载

一个非常简单的命令（假设有可用的 `config.json` 文件）如下所示。

```bash
freqtrade download-data --exchange binance
```

这将为配置中定义的所有货币对下载历史 K 线（OHLCV）数据。

或者，直接指定交易对

```bash
freqtrade download-data --exchange binance --pairs ETH/USDT XRP/USDT BTC/USDT
```

或作为正则表达式（本例中，下载所有活跃的 USDT 交易对）

```bash
freqtrade download-data --exchange binance --pairs ".*/USDT"
```

### 其他说明

* 要使用不同于交易所特定默认值的目录，使用 `--datadir user_data/data/some_directory`。
* 要更改用于下载历史数据的交易所，使用 `--exchange <exchange>`——或指定不同的配置文件。
* 要使用来自其他目录的 `pairs.json`，使用 `--pairs-file some_other_dir/pairs.json`。
* 要仅为 10 天下载历史 K 线（OHLCV）数据，使用 `--days 10`（默认为 30 天）。
* 要从固定的起始点下载历史 K 线（OHLCV）数据，使用 `--timerange 20200101-`——这将下载从 2020 年 1 月 1 日起的所有数据。
* 如果数据已可用，给定的起始点将被忽略，仅下载到今天为止缺失的数据。
* 使用 `--timeframes` 指定要下载哪个时间周期的历史 K 线数据。默认为 `--timeframes 1m 5m`，将下载 1 分钟和 5 分钟数据。
* 要使用配置文件中定义的交易所、时间周期和交易对列表，使用 `-c/--config` 选项。使用此选项，脚本将使用配置中定义的白名单作为要下载数据的货币对列表，不需要 pairs.json 文件。你可以将 `-c/--config` 与大多数其他选项结合使用。
* 在下载合约数据时（`--trading-mode futures` 或指定合约模式的配置），freqtrade 将自动下载必要的 K 线类型（例如 `mark` 和 `funding_rate` K 线），除非通过 `--candle-types` 另有指定。

??? Note "权限被拒绝错误"
    如果你的配置目录 `user_data` 是由 docker 创建的，你可能会收到以下错误：

    ```
    cp: cannot create regular file 'user_data/data/binance/pairs.json': Permission denied
    ```

    你可以按如下方式修复用户数据目录的权限：

    ```
    sudo chown -R $UID:$GID user_data
    ```

### 在当前时间范围之前下载额外数据

假设你下载了从 2022 年开始的所有数据（`--timerange 20220101-`）——但现在你还想用更早的数据进行回测。
你可以通过使用 `--prepend` 标志，结合 `--timerange`——指定一个结束日期来实现。

``` bash
freqtrade download-data --exchange binance --pairs ETH/USDT XRP/USDT BTC/USDT --prepend --timerange 20210101-20220101
```

!!! Note
    在此模式下，如果数据可用，Freqtrade 将忽略结束日期，将结束日期更新为现有数据的起始点。

### 数据格式

Freqtrade 目前支持以下数据格式：

* `feather` - 基于 Apache Arrow 的数据格式
* `json` - 纯"文本" json 文件
* `jsongz` - gzip 压缩版本的 json 文件
* `parquet` - 列式数据存储（仅 OHLCV）

默认情况下，OHLCV 数据和交易数据均以 `feather` 格式存储。

这可以分别通过 `--data-format-ohlcv` 和 `--data-format-trades` 命令行参数进行更改。
要持久化此更改，你还应将以下代码段添加到配置中，这样你就不必每次都插入上述参数：

``` jsonc
    // ...
    "dataformat_ohlcv": "feather",
    "dataformat_trades": "feather",
    // ...
```

如果在下载期间更改了默认数据格式，则配置文件中的 `dataformat_ohlcv` 和 `dataformat_trades` 键也需要调整为所选的数据格式。

!!! Note
    你可以使用 [convert-data](#sub-command-convert-data) 和 [convert-trade-data](#sub-command-convert-trade-data) 方法在数据格式之间进行转换。

#### 数据格式比较

以下比较使用以下数据并通过 linux `time` 命令进行。

```
Found 6 pair / timeframe combinations.
+----------+-------------+--------+---------------------+---------------------+
|     Pair |   Timeframe |   Type |                From |                  To |
|----------+-------------+--------+---------------------+---------------------|
| BTC/USDT |          5m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:25:00 |
| ETH/USDT |          1m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:26:00 |
| BTC/USDT |          1m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:30:00 |
| XRP/USDT |          5m |   spot | 2018-05-04 08:10:00 | 2022-09-13 19:15:00 |
| XRP/USDT |          1m |   spot | 2018-05-04 08:11:00 | 2022-09-13 19:22:00 |
| ETH/USDT |          5m |   spot | 2017-08-17 04:00:00 | 2022-09-13 19:20:00 |
+----------+-------------+--------+---------------------+---------------------+
```

计时使用以下命令以不太科学的方式获取，该命令强制将数据读入内存。

``` bash
time freqtrade list-data --show-timerange --data-format-ohlcv <dataformat>
```

|  格式 | 大小 | 耗时 |
|------------|-------------|-------------|
| `feather` | 72Mb | 3.5s |
| `json` | 149Mb | 25.6s |
| `jsongz` | 39Mb | 27s |
| `parquet` | 83Mb | 3.8s |

大小取自上述时间范围内 BTC/USDT 1m 现货组合。

为了获得最佳的性能/大小组合，我们推荐使用默认的 feather 格式，或 parquet。

### 交易对文件

作为 `config.json` 白名单的替代方案，可以使用 `pairs.json` 文件。
例如，如果你使用 Binance：

* 创建一个目录 `user_data/data/binance`，并将 `pairs.json` 文件复制或创建在该目录中。
* 更新 `pairs.json` 文件以包含你感兴趣的货币对。

```bash
mkdir -p user_data/data/binance
touch user_data/data/binance/pairs.json
```

`pairs.json` 文件的格式是一个简单的 json 列表。
此文件允许混合不同的投入货币，因为它仅用于下载。

``` json
[
    "ETH/BTC",
    "ETH/USDT",
    "BTC/USDT",
    "XRP/ETH"
]
```

!!! Note
    `pairs.json` 文件仅在未加载配置时使用（通过隐式命名或 `--config` 标志）。
    你可以通过 `--pairs-file pairs.json` 强制使用此文件——但我们建议使用配置中的交易对列表，即 `exchange.pair_whitelist` 或配置中的 `pairs` 设置。

## 子命令 convert data

--8<-- "commands/convert-data.md"

### 转换数据示例

以下命令将 `~/.freqtrade/data/binance` 中所有可用的 K 线（OHLCV）数据从 json 转换为 jsongz，在此过程中节省磁盘空间。
它还将删除原始 json 数据文件（`--erase` 参数）。

``` bash
freqtrade convert-data --format-from json --format-to jsongz --datadir ~/.freqtrade/data/binance -t 5m 15m --erase
```

## 子命令 convert trade data

--8<-- "commands/convert-trade-data.md"

### 转换交易数据示例

以下命令将 `~/.freqtrade/data/kraken` 中所有可用的交易数据从 jsongz 转换为 json。
它还将删除原始 jsongz 数据文件（`--erase` 参数）。

``` bash
freqtrade convert-trade-data --format-from jsongz --format-to json --datadir ~/.freqtrade/data/kraken --erase
```

## 子命令 trades to ohlcv

当你需要使用 `--dl-trades`（仅 Kraken）下载数据时，将交易数据转换为 OHLCV 数据是最后一步。
此命令将允许你针对额外的时间周期重复此最后一步，而无需重新下载数据。

--8<-- "commands/trades-to-ohlcv.md"

### trade-to-ohlcv 转换示例

``` bash
freqtrade trades-to-ohlcv --exchange kraken -t 5m 1h 1d --pairs BTC/EUR ETH/EUR
```

## 子命令 list-data

你可以使用 `list-data` 子命令获取已下载数据的列表。

--8<-- "commands/list-data.md"

### list-data 示例

```bash
> freqtrade list-data --userdir ~/.freqtrade/user_data/

              Found 33 pair / timeframe combinations.
┏━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━┓
┃          Pair ┃                                 Timeframe ┃ Type ┃
┡━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━┩
│       ADA/BTC │     5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d │ spot │
│       ADA/ETH │     5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d │ spot │
│       ETH/BTC │     5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d │ spot │
│      ETH/USDT │                  5m, 15m, 30m, 1h, 2h, 4h │ spot │
└───────────────┴───────────────────────────────────────────┴──────┘
```

显示所有交易数据，包括起止时间范围

``` bash
> freqtrade list-data --show --trades
                     Found trades data for 1 pair.                     
┏━━━━━━━━━┳━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━┓
┃    Pair ┃ Type ┃                From ┃                  To ┃ Trades ┃
┡━━━━━━━━━╇━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━┩
│ XRP/ETH │ spot │ 2019-10-11 00:00:11 │ 2019-10-13 11:19:28 │  12477 │
└─────────┴──────┴─────────────────────┴─────────────────────┴────────┘
```

## 交易（Tick）数据

默认情况下，`download-data` 子命令下载 K 线（OHLCV）数据。大多数交易所也通过 API 提供历史交易数据。
如果你需要许多不同时间周期的数据，这些数据可能很有用，因为它只下载一次，然后在本地重采样到所需的时间周期。

由于此数据默认很大，文件默认使用 feather 格式。它们被存储在你的数据目录中，命名约定为 `<pair>-trades.feather`（`ETH_BTC-trades.feather`）。也支持增量模式，与历史 OHLCV 数据一样，因此每周使用 `--days 8` 下载一次数据将创建一个增量数据仓库。

要使用此模式，只需在你的调用中添加 `--dl-trades`。这将把下载方法切换为下载交易数据。
如果还提供了 `--convert`，重采样步骤将自动发生并覆盖给定交易对/时间周期组合的现有 OHLCV 数据。

!!! Warning "不要使用"
    你不应该使用此功能，除非你是 Kraken 用户（Kraken 不提供历史 OHLCV 数据）。
    大多数其他交易所提供具有足够历史的 OHLCV 数据，因此通过该方法下载多个时间周期仍然比下载交易数据快得多。

!!! Note "Kraken 用户"
    Kraken 用户在开始下载数据之前应阅读[此内容](exchanges.md#historic-kraken-data)。

    Kraken Futures 使用标准 OHLCV 下载，不需要 `--dl-trades`。

示例调用：

```bash
freqtrade download-data --exchange kraken --pairs XRP/EUR ETH/EUR --days 20 --dl-trades
```

!!! Note
    虽然此方法使用异步调用，但它会较慢，因为它需要前一次调用的结果来生成对交易所的下一次请求。

## 下一步

很好，你现在已经有了一些下载的数据，因此可以开始[回测](backtesting.md)你的策略了。