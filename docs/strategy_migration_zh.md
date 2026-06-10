# V2 到 V3 的策略迁移

为了支持新的市场和交易类型（即做空/杠杆交易），接口中必须进行一些更改。

## 快速摘要 / 迁移检查清单

### 策略方法重命名：
* `populate_buy_trend()` → `populate_entry_trend()`
* `populate_sell_trend()` → `populate_exit_trend()`
* `custom_sell()` → `custom_exit()`
* `check_buy_timeout()` → `check_entry_timeout()`
* `check_sell_timeout()` → `check_exit_timeout()`

### 数据框列变更：
* `buy` → `enter_long`
* `sell` → `exit_long`
* `buy_tag` → `enter_tag`
* 新列 `enter_short` 和对应的 `exit_short`

### 配置术语变更：
* `use_sell_signal` → `use_exit_signal`
* `sell_profit_only` → `exit_profit_only`
* `sell_profit_offset` → `exit_profit_offset`
* `ignore_roi_if_buy_signal` → `ignore_roi_if_entry_signal`
* `forcebuy_enable` → `force_entry_enable`
* `bid_strategy` → `entry_pricing`
* `ask_strategy` → `exit_pricing`

### Telegram 通知设置变更：
* `buy` → `entry`, `buy_fill` → `entry_fill`
* `sell` → `exit`, `sell_fill` → `exit_fill`, `sell_cancel` → `exit_cancel`

### Webhook 术语变更：
* `webhookbuy` → `entry`, `webhooksell` → `exit`
* `webhookbuyfill` → `entry_fill`, `webhooksellfill` → `exit_fill`

### Trade 对象变更：
* 新属性：`is_short`, `entry_side`, `exit_side`, `trade_direction`
* 重命名：`sell_reason` → `exit_reason`

### 详细示例：`populate_buy_trend` → `populate_entry_trend`

**之前：**
```python
def populate_buy_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe.loc[
        (...), ['buy', 'buy_tag']] = (1, 'rsi_cross')
    return dataframe
```

**之后：**
```python
def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe.loc[
        (...), ['enter_long', 'enter_tag']] = (1, 'rsi_cross')
    return dataframe
```

### 详细示例：`custom_sell` → `custom_exit`

**之前：**
```python
def custom_sell(self, pair: str, trade: 'Trade', current_time: 'datetime',
                current_rate: float, current_profit: float, **kwargs):
```

**之后：**
```python
def custom_exit(self, pair: str, trade: 'Trade', current_time: 'datetime',
                current_rate: float, current_profit: float, **kwargs):
```

### `custom_stake_amount` 新增 `side` 参数

```python
def custom_stake_amount(self, pair: str, current_time: datetime, current_rate: float,
                        proposed_stake: float, min_stake: float | None, max_stake: float,
                        entry_tag: str | None, side: str, **kwargs) -> float:
```

### `confirm_trade_entry` 新增 `side` 参数

```python
def confirm_trade_entry(self, pair: str, order_type: str, amount: float, rate: float,
                        time_in_force: str, current_time: datetime, entry_tag: str | None,
                        side: str, **kwargs) -> bool:
```

### `confirm_trade_exit` 变更

参数 `sell_reason` 改为 `exit_reason`。

### `nr_of_successful_buys` → `nr_of_successful_entries`

### 新增 `leverage` 回调

新的 [`leverage` 回调](strategy-callbacks.md#leverage-callback)允许按交易对动态设置杠杆。

### `INTERFACE_VERSION` 应设为 3