# Telegram 使用

## 设置你的 Telegram 机器人

### 1. 创建你的 Telegram 机器人

与 [Telegram BotFather](https://telegram.me/BotFather) 开始对话，发送 `/newbot`。

选择机器人的公开名称（例如 `Freqtrade bot`），然后选择机器人的用户名 ID（例如 `My_own_freqtrade_bot`）。

BotFather 将返回一个 Token（API 密钥），如 `22222222:APITOKEN`。将其用于配置参数 `token`。

不要忘记通过点击 `/START` 按钮开始与你的机器人对话。

### 2. Telegram user_id

与 [userinfobot](https://telegram.me/userinfobot) 对话获取你的 "Id"，用于配置参数 `chat_id`。

#### 使用群组 ID

将机器人添加到群组，启动 freqtrade，发送 `/tg_info` 命令。这将返回群组 ID。

```json
 {
    "enabled": true,
    "token": "********",
    "chat_id": "-1001332619709",
    "topic_id": "122"
}
```

## 控制 Telegram 通知噪音

每个设置有以下可能值：
* `on` - 发送消息并通知用户。
* `silent` - 发送消息，无声音/振动通知。
* `off` - 完全跳过发送此消息类型。

``` json
"telegram": {
    "enabled": true,
    "notification_settings": {
        "status": "silent",
        "warning": "on",
        "startup": "off",
        "entry": "silent",
        "entry_fill": "on",
        "entry_cancel": "silent",
        "exit": {
            "roi": "silent",
            "emergency_exit": "on",
            "force_exit": "on",
            "exit_signal": "silent",
            "stop_loss": "on",
            "custom_exit": "silent",
            "partial_exit": "on"
        },
        "exit_cancel": "on",
        "exit_fill": "off",
        "protection_trigger": "off",
        "strategy_msg": "off",
        "show_candle": "off"
    },
    "reload": true,
    "balance_dust_level": 0.01
},
```

* `entry` 通知在下单时发送，`entry_fill` 在订单成交时发送。
* `exit` 通知在下出场单时发送，`exit_fill` 在出场单成交时发送。
* `show_candle` - 在入场/出场消息中显示 K 线值。可能值：`"ohlc"` 或 `"off"`。
* `balance_dust_level` - 定义 `/balance` 命令视为"粉尘"的阈值。
* `reload` - 允许禁用选定消息上的重新加载按钮。

## Telegram 命令

|  命令 | 描述 |
|----------|-------------|
| `/start` | 启动机器人 |
| `/stop` | 停止机器人 |
| `/status` | 列出所有未平仓交易 |
| `/status table` | 以表格形式列出所有未平仓交易 |
| `/trades` | 列出最近交易 |
| `/profit` | 显示盈亏摘要 |
| `/forcesell <tradeid>` | 强制平掉指定交易 |
| `/forcebuy <pair>` | 强制买入交易对 |
| `/delete <tradeid>` | 删除交易 |
| `/performance` | 显示交易对表现 |
| `/balance` | 显示账户余额 |
| `/daily <n>` | 显示最近 n 天的每日盈亏 |
| `/weekly <n>` | 显示最近 n 天的每周盈亏 |
| `/monthly <n>` | 显示最近 n 天的每月盈亏 |
| `/count` | 显示交易槽位使用情况 |
| `/reload_config` | 重新加载配置 |
| `/show_config` | 显示当前配置 |
| `/stopbuy` | 停止开新仓位 |
| `/whitelist` | 显示当前白名单 |
| `/blacklist [pair]` | 显示或管理黑名单 |
| `/logs <n>` | 显示最近 n 条日志消息 |
| `/help` | 显示帮助信息 |