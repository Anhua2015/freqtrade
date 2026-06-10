# 交易所特定说明

本页汇总了各交易所特定的常见问题和信息。

## 支持的交易所功能快速概览

--8<-- "includes/exchange-features.md"

## 交易所配置

Freqtrade 基于 [CCXT 库](https://github.com/ccxt/ccxt)，该库支持超过 100 个加密货币交易所。

示例配置（Binance）：

```json
"exchange": {
    "name": "binance",
    "key": "your_exchange_key",
    "secret": "your_exchange_secret",
    "ccxt_config": {},
    "ccxt_async_config": {},
```

### 设置速率限制

```json
"exchange": {
    "name": "kraken",
    "ccxt_config": {"enableRateLimit": true},
    "ccxt_async_config": {
        "enableRateLimit": true,
        "rateLimit": 3100
    },
```

## Binance

!!! Warning "服务器位置和地域 IP 限制"
    请注意 Binance 根据服务器国家限制 API 访问。当前受限国家包括加拿大、马来西亚、荷兰和美国。

!!! Tip "交易所止损"
    Binance 支持 `stoploss_on_exchange` 并使用 `stop-loss-limit` 订单。在合约上，Binance 同时支持 `stop-limit` 和 `stop-market` 订单。

### Binance 站点

* [binance.com](https://www.binance.com/) - 国际用户。使用 exchange id: `binance`。
* [binance.us](https://www.binance.us/) - 美国用户。使用 exchange id: `binanceus`。

### Binance 合约

在 Binance 合约市场交易时，必须使用订单簿。

``` jsonc
  "entry_pricing": {
      "use_order_book": true,
      "order_book_top": 1,
  },
  "exit_pricing": {
      "use_order_book": true,
      "order_book_top": 1
  },
```

用户还需要将合约设置 "Position Mode" 设置为 "One-way Mode"，"Asset Mode" 设置为 "Single-Asset Mode"。

## Kraken

!!! Tip "交易所止损"
    Kraken 支持 `stoploss_on_exchange`，可使用止损市价和止损限价订单。

### Kraken 历史数据

Kraken API 仅提供 720 根历史 K 线。要下载 Kraken 交易所的数据，必须使用 `--dl-trades`。

## 其他交易所

- **BingX**：支持 `stoploss_on_exchange`，可使用止损限价和止损市价订单。
- **Bybit**：支持统一交易账户。`stoploss_on_exchange` 参数将被忽略。
- **Gate.io**：不支持市价单。需要将订单类型设置为 `limit`。
- **OKX**：支持 `stoploss_on_exchange`，在统一账户模式下需要 `trading_mode: "futures"` 和 `margin_mode: "isolated"`。