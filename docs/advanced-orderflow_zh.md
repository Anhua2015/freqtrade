# Orderflow 订单流数据

本指南将引导你利用公开交易数据进行 Freqtrade 中的高级订单流分析。

!!! Warning "实验性功能"
    订单流功能目前处于 beta 阶段，未来版本中可能会有所变更。请在 [Freqtrade GitHub 仓库](https://github.com/freqtrade/freqtrade/issues) 上报告任何问题或反馈。
    目前尚未与 freqAI 进行测试——在现阶段结合这两个功能被视为超出范围。

!!! Warning "性能"
    订单流需要原始交易数据。这些数据相当大，当 freqtrade 需要下载最近 X 根 K 线的交易数据时，可能导致较慢的初始启动。此外，启用此功能将增加内存使用。请确保有足够的资源可用。

## 入门

### 启用公开交易数据

在你的 `config.json` 文件中，将 `exchange` 部分下的 `use_public_trades` 选项设置为 true。

```json
"exchange": {
   ...
   "use_public_trades": true,
}
```

### 配置订单流处理

在 config.json 的 orderflow 部分中定义你所需的订单流处理设置。你可以调整以下因素：

- `cache_size`：将多少先前的订单流 K 线保存到缓存中，而不是每根新 K 线重新计算
- `max_candles`：筛选你要获取交易数据的 K 线数量。
- `scale`：控制足迹图的价格档位大小。
- `stacked_imbalance_range`：定义考虑的连续不平衡价格水平的最小数量。
- `imbalance_volume`：过滤掉低于此阈值的成交量不平衡。
- `imbalance_ratio`：过滤掉比率（买卖量差）低于此值的不平衡。

```json
"orderflow": {
    "cache_size": 1000, 
    "max_candles": 1500, 
    "scale": 0.5, 
    "stacked_imbalance_range": 3, //  至少需要这么多连续的不平衡
    "imbalance_volume": 1, //  过滤低于此值的
    "imbalance_ratio": 3 //  过滤比率低于此值的
  },
```

## 下载回测用的交易数据

要下载历史交易数据进行回测，使用 `--dl-trades` 标志配合 `freqtrade download-data` 命令。

```bash
freqtrade download-data -p BTC/USDT:USDT --timerange 20230101- --trading-mode futures --timeframes 5m --dl-trades
```

!!! Warning "数据可用性"
    并非所有交易所都提供公开交易数据。对于支持的交易所，如果你使用 `--dl-trades` 标志开始下载数据，freqtrade 会在公开交易数据不可用时发出警告。

## 访问订单流数据

一旦激活，你的 DataFrame 中将提供几个新列：

``` python

dataframe["trades"] # 包含每笔单独交易的信息。
dataframe["orderflow"] # 表示足迹图字典（见下文）
dataframe["imbalances"] # 包含订单流中不平衡的信息。
dataframe["bid"] # 总买入量 
dataframe["ask"] # 总卖出量
dataframe["delta"] # 买卖量之间的差值。
dataframe["min_delta"] # K 线内的最小 delta
dataframe["max_delta"] # K 线内的最大 delta
dataframe["total_trades"] # 交易总数
dataframe["stacked_imbalances_bid"] # 堆叠买入不平衡范围起点的价格水平列表
dataframe["stacked_imbalances_ask"] # 堆叠卖出不平衡范围起点的价格水平列表
```

你可以在策略代码中访问这些列以进行进一步分析。以下是一个示例：

``` python
def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    # 计算累积 delta
    dataframe["cum_delta"] = cumulative_delta(dataframe["delta"])
    # 访问总交易数
    total_trades = dataframe["total_trades"]
    ...

def cumulative_delta(delta: Series):
    cumdelta = delta.cumsum()
    return cumdelta

```

### 足迹图（`dataframe["orderflow"]`）

此列提供了在不同价格水平上买卖订单的详细细分，为订单流动态提供了宝贵的洞察。配置中的 `scale` 参数决定了此表示的价格档位大小。

`orderflow` 列包含一个具有以下结构的字典：

``` output
{
    "price": {
        "bid_amount": 0.0,
        "ask_amount": 0.0,
        "bid": 0,
        "ask": 0,
        "delta": 0.0,
        "total_volume": 0.0,
        "total_trades": 0
    }
}
```

#### 订单流列说明

- 键: 价格档位 - 以 `scale` 间隔进行分档
- `bid_amount`: 每个价格水平的总买入量。
- `ask_amount`: 每个价格水平的总卖出量。
- `bid`: 每个价格水平的买入订单数量。
- `ask`: 每个价格水平的卖出订单数量。
- `delta`: 每个价格水平上买卖量之间的差值。
- `total_volume`: 每个价格水平的总成交量（卖量 + 买量）。
- `total_trades`: 每个价格水平的总交易数（卖 + 买）。

通过利用这些功能，你可以基于订单流分析获得对市场情绪和潜在交易机会的宝贵洞察。

### 原始交易数据（`dataframe["trades"]`）

包含该 K 线期间发生的每笔单独交易的列表。此数据可用于更细粒度地分析订单流动态。

每个单独的条目包含一个具有以下键的字典：

- `timestamp`: 交易的时间戳。
- `date`: 交易的日期。
- `price`: 交易的价格。
- `amount`: 交易的成交量。
- `side`: 买入或卖出。
- `id`: 交易的唯一标识符。
- `cost`: 交易的总成本（price * amount）。

### 不平衡（`dataframe["imbalances"]`）

此列提供一个包含订单流中不平衡信息的字典。当给定价格水平上买卖量存在显著差异时，就会发生不平衡。

每行如下所示——以 price 为索引，对应的买卖不平衡值作为列：

``` output
{
    "price": {
        "bid_imbalance": False,
        "ask_imbalance": False
    }
}
```