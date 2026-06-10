## 交易对列表和交易对列表处理器

交易对列表处理器定义了机器人应交易的交易对列表。它们在配置设置的 `pairlists` 部分中配置。

可用的处理器：
* `StaticPairList` - 使用静态白名单
* `VolumePairList` - 按交易量排序/筛选
* `PercentChangePairList` - 按价格百分比变化排序
* `ProducerPairList` - 从生产者重用交易对列表
* `RemotePairList` - 从远程服务器获取交易对列表
* `MarketCapPairList` - 按市值排序
* `CrossMarketPairList` - 跨市场交易对

过滤器：`AgeFilter`、`DelistFilter`、`PrecisionFilter`、`PriceFilter`、`ShuffleFilter`、`SpreadFilter`、`VolatilityFilter`、`RangeStabilityFilter` 等。

交易对黑名单支持正则表达式，例如 `BNB/.*` 排除所有 BNB 交易对。

### 常用配置示例：

```json
"exchange": {
    "pair_whitelist": ["BTC/USDT", "ETH/USDT"],
    "pair_blacklist": ["BNB/USDT"]
},
"pairlists": [
    {"method": "VolumePairList", "number_assets": 20, "sort_key": "quoteVolume"},
    {"method": "AgeFilter", "min_days_listed": 10},
    {"method": "PrecisionFilter"},
    {"method": "PriceFilter", "low_price_ratio": 0.01},
    {"method": "SpreadFilter", "max_spread_ratio": 0.005},
    {"method": "ShuffleFilter", "seed": 42}
],
```

完整文档请参考原版 [pairlists 文档](plugins.md#pairlists-and-pairlist-handlers)。