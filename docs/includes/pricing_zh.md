## 订单价格

订单价格可通过 `entry_pricing`（入场）和 `exit_pricing`（出场）参数结构控制。

### 入场价格方向

`entry_pricing.price_side` 定义机器人在买入时查看订单簿的哪一侧。可选值：`ask`、`bid`、`same`、`other`。

推荐使用 `"same"` 或 `"other"`，因为方向（多/空）会导致不同结果。

### 启用订单簿的入场价格

设置 `entry_pricing.use_order_book=True` 且 `entry_pricing.order_book_top` 指定使用订单簿中第几档的订单。

### 不启用订单簿的入场价格

`entry_pricing.price_last_balance` 控制价格在 `side` 价格和 `last` 价格之间插值的比例：`0.0` 使用 side 价格，`1.0` 使用 last 价格。

### 市场深度检查

`entry_pricing.check_depth_of_market.enabled=True` 时，基于订单簿深度过滤入场信号。
`bids_to_ask_delta` 参数定义买卖深度之比的最小阈值。

### 出场价格

配置逻辑与入场价格相同，方向矩阵相反。

### 市价单定价

使用市价单时，价格应配置为使用订单簿的"正确"一侧：

``` jsonc
  "order_types": {"entry": "market", "exit": "market"},
  "entry_pricing": {"price_side": "other"},
  "exit_pricing": {"price_side": "other"},
```