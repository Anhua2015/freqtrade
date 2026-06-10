## 保护机制

保护机制通过暂时停止交易来保护你的策略免受意外事件和市场条件的影响。所有保护结束时间会向上取整到下一根 K 线，以避免突然的意外买入。

### 可用的保护机制

* `StoplossGuard` - 在特定时间窗口内发生一定数量的止损后停止交易
* `MaxDrawdown` - 达到最大回撤后停止交易
* `LowProfitPairs` - 锁定低利润的交易对
* `CooldownPeriod` - 卖出交易后不立即重新入场

### 通用设置

| 参数 | 描述 |
| --------- | ---------- |
| `method` | 保护名称 |
| `stop_duration_candles` | 锁定持续的 K 线数 |
| `lookback_period_candles` | 回溯期 K 线数 |
| `trade_limit` | 最少需要的交易数 |

### 完整示例

``` python
from freqtrade.strategy import IStrategy

class AwesomeStrategy(IStrategy):
    timeframe = '1h'
    
    @property
    def protections(self):
        return [
            {"method": "CooldownPeriod", "stop_duration_candles": 5},
            {
                "method": "MaxDrawdown",
                "lookback_period_candles": 48,
                "trade_limit": 20,
                "stop_duration_candles": 4,
                "max_allowed_drawdown": 0.2
            },
            {
                "method": "StoplossGuard",
                "lookback_period_candles": 24,
                "trade_limit": 4,
                "stop_duration_candles": 2,
                "only_per_pair": False
            },
            {
                "method": "LowProfitPairs",
                "lookback_period_candles": 6,
                "trade_limit": 2,
                "stop_duration_candles": 60,
                "required_profit": 0.02
            }
        ]
```