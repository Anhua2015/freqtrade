# FreqAI 配置

FreqAI 通过典型的 [Freqtrade 配置文件](configuration.md)和标准的 [Freqtrade 策略](strategy-customization.md)进行配置。

## 设置配置文件

FreqAI 配置至少必须包含以下参数（参数值仅为示例）：

```json
    "freqai": {
        "enabled": true,
        "purge_old_models": 2,
        "train_period_days": 30,
        "backtest_period_days": 7,
        "identifier" : "unique-id",
        "feature_parameters" : {
            "include_timeframes": ["5m","15m","4h"],
            "include_corr_pairlist": ["ETH/USD", "LINK/USD", "BNB/USD"],
            "label_period_candles": 24,
            "include_shifted_candles": 2,
            "indicator_periods_candles": [10, 20]
        },
        "data_split_parameters" : {
            "test_size": 0.25
        }
    }
```

完整的示例配置在 `config_examples/config_freqai.example.json` 中。

!!! Note
    `identifier` 常被新手忽视，但这个值在配置中扮演重要角色。它是你选择用来描述某次运行的唯一 ID。保持相同可以维持崩溃恢复能力以及更快的回测速度。一旦你想尝试新的运行（新特征、新模型等），就应更改此值。

## 构建 FreqAI 策略

FreqAI 策略需要在标准 Freqtrade 策略中包含以下代码：

```python
    startup_candle_count: int = 20

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe = self.freqai.start(dataframe, metadata, self)
        return dataframe

    def feature_engineering_expand_all(self, dataframe: DataFrame, period, **kwargs) -> DataFrame:
        dataframe["%-rsi-period"] = ta.RSI(dataframe, timeperiod=period)
        dataframe["%-mfi-period"] = ta.MFI(dataframe, timeperiod=period)
        dataframe["%-adx-period"] = ta.ADX(dataframe, timeperiod=period)
        dataframe["%-sma-period"] = ta.SMA(dataframe, timeperiod=period)
        dataframe["%-ema-period"] = ta.EMA(dataframe, timeperiod=period)
        return dataframe

    def feature_engineering_expand_basic(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        dataframe["%-pct-change"] = dataframe["close"].pct_change()
        dataframe["%-raw_volume"] = dataframe["volume"]
        dataframe["%-raw_price"] = dataframe["close"]
        return dataframe

    def feature_engineering_standard(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        dataframe["%-day_of_week"] = (dataframe["date"].dt.dayofweek + 1) / 7
        dataframe["%-hour_of_day"] = (dataframe["date"].dt.hour + 1) / 25
        return dataframe

    def set_freqai_targets(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        dataframe["&-s_close"] = (
            dataframe["close"]
            .shift(-self.freqai_info["feature_parameters"]["label_period_candles"])
            .rolling(self.freqai_info["feature_parameters"]["label_period_candles"])
            .mean()
            / dataframe["close"]
            - 1
            )
        return dataframe
```

## 重要的数据框键模式

以下是典型策略数据框中可能包含/使用的值：

|  数据框键 | 描述 |
|------------|-------------|
| `df['&*']` | 在 `set_freqai_targets()` 中以 `&` 为前缀的任何数据框列都被视为训练目标（标签）。FreqAI 进行预测并以相同的键返回它们。 |
| `df['&*_std/mean']` | 训练期间定义标签的标准差和均值。常用于理解预测的稀有性。 |
| `df['do_predict']` | 异常数据点的指示。返回值为 -2 到 2 之间的整数，告知预测是否可信。`do_predict==1` 表示预测可信。 |
| `df['DI_values']` | 差异指数值，是 FreqAI 对预测信心水平的代理。DI 越低表示预测越接近训练数据，即预测置信度越高。 |
| `df['%*']` | 在 `feature_engineering_*()` 中以 `%` 为前缀的任何数据框列都被视为训练特征。 |
| `df['%%*']` | 与 `%` 相同，但特征会返回给策略用于绘图和监控。 |

## 创建动态目标阈值

决定何时入场或出场可以以动态方式反映当前市场条件。FreqAI 允许你从模型训练中返回额外信息。例如，`&*_std/mean` 返回值描述了*最近训练期间*目标/标签的统计分布。

```python
dataframe["target_roi"] = dataframe["&-s_close_mean"] + dataframe["&-s_close_std"] * 1.25
dataframe["sell_roi"] = dataframe["&-s_close_mean"] - dataframe["&-s_close_std"] * 1.25
```

要使用*历史预测*的总体来创建动态目标，可以在配置中设置 `fit_live_predictions_candles`：

```json
    "freqai": {
        "fit_live_predictions_candles": 300,
    }
```