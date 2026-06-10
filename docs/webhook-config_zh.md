# Webhook 使用

## 配置

通过在配置文件中添加 webhook 部分并将 `webhook.enabled` 设置为 `true` 来启用 webhooks。

示例配置（已使用 IFTTT 测试）。

```json
  "webhook": {
        "enabled": true,
        "url": "https://maker.ifttt.com/trigger/<YOUREVENT>/with/key/<YOURKEY>/",
        "entry": {
            "value1": "Buying {pair}",
            "value2": "limit {limit:8f}",
            "value3": "{stake_amount:8f} {stake_currency}"
        },
        "entry_cancel": {
            "value1": "Cancelling Open Buy Order for {pair}",
            "value2": "limit {limit:8f}",
            "value3": "{stake_amount:8f} {stake_currency}"
        },
         "entry_fill": {
            "value1": "Buy Order for {pair} filled",
            "value2": "at {open_rate:8f}",
            "value3": ""
        },
        "exit": {
            "value1": "Exiting {pair}",
            "value2": "limit {limit:8f}",
            "value3": "profit: {profit_amount:8f} {stake_currency} ({profit_ratio})"
        },
        "exit_cancel": {
            "value1": "Cancelling Open Exit Order for {pair}",
            "value2": "limit {limit:8f}",
            "value3": "profit: {profit_amount:8f} {stake_currency} ({profit_ratio})"
        },
        "exit_fill": {
            "value1": "Exit Order for {pair} filled",
            "value2": "at {close_rate:8f}.",
            "value3": ""
        },
        "status": {
            "value1": "Status: {status}",
            "value2": "",
            "value3": ""
        }
    },
```

`webhook.url` 中的 url 应指向你的 webhook 的正确地址。如果你使用 [IFTTT](https://ifttt.com)（如上例所示），请在 url 中插入你的事件和密钥。

你可以将 POST 正文格式设置为 Form-Encoded（默认）、JSON-Encoded 或原始数据。分别使用 `"format": "form"`、`"format": "json"` 或 `"format": "raw"`。

### 嵌套 Webhook 配置

某些 webhook 目标需要嵌套结构。
这可以通过将内容设置为字典或列表而不是直接文本来实现。
这仅支持 JSON 格式。

### 额外配置

`webhook.retries` 参数可以设置 webhook 请求在失败时的最大重试次数（即 HTTP 响应状态码不为 200）。默认设为 `0`（禁用）。额外的 `webhook.retry_delay` 参数可以指定重试之间的秒数。默认为 `0.1`（即 100ms）。注意如果 webhook 存在连接问题，增加重试次数或重试延迟可能会减慢交易速度。
你还可以指定 `webhook.timeout`——定义机器人在假定对方主机无响应之前等待的时间（默认为 10 秒）。

自定义消息可以通过 `self.dp.send_msg()` 函数从策略发送到 Webhook 端点。要启用此功能，将 `allow_custom_messages` 选项设置为 `true`。

## Webhook 消息类型

### 入场 / 入场成交

`webhook.entry` 和 `webhook.entry_fill` 字段在机器人下多/空单增加仓位时或该订单成交时填充。参数使用 string.format 填充。

可用参数：`trade_id`、`exchange`、`pair`、`direction`、`leverage`、`open_rate`、`amount`、`open_date`、`stake_amount`、`stake_currency`、`base_currency`、`quote_currency`、`fiat_currency`、`order_type`、`current_rate`、`enter_tag`

### 入场取消

`webhook.entry_cancel` 字段在机器人取消多/空订单时填充。

可用参数：`trade_id`、`exchange`、`pair`、`direction`、`leverage`、`limit`、`amount`、`open_date`、`stake_amount`、`stake_currency`、`base_currency`、`quote_currency`、`fiat_currency`、`order_type`、`current_rate`、`enter_tag`

### 出场 / 出场成交

`webhook.exit` 和 `webhook.exit_fill` 字段在机器人下出场单或该出场单成交时填充。

可用参数：`trade_id`、`exchange`、`pair`、`direction`、`leverage`、`gain`、`amount`、`open_rate`、`close_rate`、`current_rate`、`profit_amount`、`profit_ratio`、`stake_currency`、`base_currency`、`quote_currency`、`fiat_currency`、`enter_tag`、`exit_reason`、`order_type`、`open_date`、`close_date`、`sub_trade`、`is_final_exit`

### 出场取消

`webhook.exit_cancel` 字段在机器人取消出场订单时填充。

### 状态

`webhook.status` 字段用于常规状态消息（Started / Stopped / ...）。唯一可用值是 `{status}`。

## Discord

Discord 有一种特殊的 webhook 形式。你可以按如下方式配置：

```json
"discord": {
    "enabled": true,
    "webhook_url": "https://discord.com/api/webhooks/<Your webhook URL ...>",
    "exit_fill": [
        {"Trade ID": "{trade_id}"},
        {"Exchange": "{exchange}"},
        ...
    ],
    "entry_fill": [
        {"Trade ID": "{trade_id}"},
        ...
    ]
}
```

以上代表默认配置（`exit_fill` 和 `entry_fill` 是可选的，将默认为上述配置）——显然可以进行修改。
要禁用默认值之一，可以给它们分配空数组（`exit_fill: []`）。

自定义消息可以从策略通过 dataprovider.send_msg() 函数发送到 Discord 端点。要启用此功能，将 `allow_custom_messages` 选项设置为 `true`。