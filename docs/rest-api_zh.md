# REST API

## FreqUI

FreqUI 现在有自己的专用[文档部分](freq-ui.md)——请参考该部分获取有关 FreqUI 的所有信息。

## 配置

通过在配置中添加 api_server 部分并将 `api_server.enabled` 设置为 `true` 来启用 REST API。

示例配置：

``` json
    "api_server": {
        "enabled": true,
        "listen_ip_address": "127.0.0.1",
        "listen_port": 8080,
        "verbosity": "error",
        "enable_openapi": false,
        "jwt_secret_key": "somethingRandomSomethingRandom123",
        "CORS_origins": [],
        "username": "Freqtrader",
        "password": "SuperSecret1!",
        "ws_token": "sercet_Ws_t0ken"
    },
```

!!! Danger "安全警告"
    默认情况下，配置仅监听 localhost（因此无法从其他系统访问）。我们强烈建议不要将此 API 暴露到互联网，并选择强且唯一的密码，因为其他人可能能够控制你的机器人。

### 安全性

要生成安全密码，最好使用密码管理器，或使用以下代码：

``` python
import secrets
secrets.token_hex()
```

### Docker 配置

如果你使用 docker 运行机器人，你需要让机器人监听传入的连接。安全性则由 docker 处理。

``` json
    "api_server": {
        "enabled": true,
        "listen_ip_address": "0.0.0.0",
        "listen_port": 8080,
        "username": "Freqtrader",
        "password": "SuperSecret1!",
    },
```

确保你的 docker-compose 文件中有以下 2 行：

```yml
    ports:
      - "127.0.0.1:8080:8080"
```

## REST API

### 使用 API

我们建议使用受支持的 `freqtrade-client` 包（也可作为 `scripts/rest_client.py` 使用）来调用 API。

此命令可以使用 `pip install freqtrade-client` 独立于任何正在运行的 freqtrade 机器人安装。

``` bash
freqtrade-client <command> [optional parameters]
```

### 可用端点

如果你希望通过其他方式手动调用 REST API，例如直接通过 `curl`，下表显示了相关的 URL 端点和参数。

|  端点 | 方法 | 描述 / 参数 |
|-----------|--------|--------------------------|
| `/ping` | GET | 测试 API 就绪状态的简单命令——无需认证。 |
| `/start` | POST | 启动交易者。 |
| `/pause` | POST | 暂停交易者。按其规则优雅处理未平仓交易。不开新仓位。 |
| `/stop` | POST | 停止交易者。 |
| `/stopbuy` | POST | 停止交易者开新交易。按其规则优雅平仓未平仓交易。 |
| `/reload_config` | POST | 重新加载配置文件。 |
| `/trades` | GET | 列出最近的交易。每次调用限制 500 条。 |
| `/trade/<tradeid>` | GET | 获取特定交易。 |
| `/status` | GET | 列出所有未平仓交易。 |
| `/profit` | GET | 显示已平仓交易的盈亏摘要和表现统计。 |
| `/forceexit` | POST | 立即平掉给定交易（忽略 `minimum_roi`），使用给定的订单类型。 |
| `/forceenter` | POST | 立即进入给定交易对。需将 `force_entry_enable` 设为 True。 |
| `/performance` | GET | 显示按交易对分组的每笔已完成交易的表现。 |
| `/balance` | GET | 显示每种货币的账户余额。 |
| `/daily` | GET | 显示最近 n 天的每日盈亏。 |
| `/weekly` | GET | 显示最近 n 天的每周盈亏。 |
| `/monthly` | GET | 显示最近 n 天的每月盈亏。 |
| `/whitelist` | GET | 显示当前白名单。 |
| `/blacklist` | GET/POST/DELETE | 管理黑名单。 |
| `/version` | GET | 显示版本。 |
| `/health` | GET | 显示机器人健康状况（最近一次机器人循环）。 |

### 消息 WebSocket

API 服务器包含一个 WebSocket 端点，用于订阅来自 freqtrade 机器人的 RPC 消息。
这可用于消费机器人的实时数据，如入场/出场成交消息、白名单变更、交易对的填充指标等。

这也是设置 Freqtrade [生产者/消费者模式](producer-consumer.md)的方式。

要访问 WebSocket 端点，需要 `ws_token` 作为端点 URL 中的查询参数。

``` json
"api_server": {
    "ws_token": "hZ-y58LXyX_HZ8O1cJzVyN6ePWrLpNQv4Q"
},
```

然后可以连接到 `http://localhost:8080/api/v1/message/ws?token=hZ-y58LXyX_HZ8O1cJzVyN6ePWrLpNQv4Q`。

连接后，机器人将向订阅者广播 RPC 消息。要订阅消息列表，必须通过 WebSocket 发送 JSON 请求：

``` json
{
  "type": "subscribe",
  "data": ["whitelist", "analyzed_df"]
}
```

### OpenAPI 接口

要启用内置的 openAPI 接口（Swagger UI），在 api_server 配置中指定 `"enable_openapi": true`。
这将在 `/docs` 端点启用 Swagger UI。

### 使用 JWT 令牌的高级 API 用法

Freqtrade 的 REST API 也提供 JWT（JSON Web Tokens）。
你可以使用以下命令登录，随后使用生成的 access_token。

``` bash
> curl -X POST --user Freqtrader http://localhost:8080/api/v1/token/login
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...","refresh_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."}
```

由于 access token 有较短的超时时间（15 分钟）——应定期使用 `token/refresh` 请求获取新的 access token。

--8<-- "includes/cors.md"