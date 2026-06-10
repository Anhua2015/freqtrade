# 特征工程

## 定义特征

底层特征工程在用户策略中的一组名为 `feature_engineering_*` 的函数中执行。这些函数设置`基础特征`，如 `RSI`、`MFI`、`EMA`、`SMA`、日内时间、成交量等。

|  函数 | 描述 |
|---------------|-------------|
| `feature_engineering_expand_all()` | 自动按配置中的 `indicator_periods_candles`、`include_timeframes`、`include_shifted_candles` 和 `include_corr_pairs` 扩展特征。 |
| `feature_engineering_expand_basic()` | 自动按配置中的 `include_timeframes`、`include_shifted_candles` 和 `include_corr_pairs` 扩展特征（不扩展 `indicator_periods_candles`）。 |
| `feature_engineering_standard()` | 在基础时间周期的数据框上调用一次。这是最后被调用的函数，此时数据框已包含所有其他特征工程函数创建的特征。适合做自定义特征（如 tsfresh）或不应自动扩展的特征（如星期几）。 |
| `set_freqai_targets()` | 必需函数，设置模型的目标。所有目标必须以 `&` 为前缀。 |

同时，高层特征工程在 FreqAI 配置中的 `"feature_parameters":{}` 内处理。可以决定在基础特征之上进行大规模特征扩展，如"包含关联交易对"、"包含信息性时间周期"或"包含近期 K 线"。

示例：

```python
    def feature_engineering_expand_all(self, dataframe: DataFrame, period, metadata, **kwargs) -> DataFrame:
        dataframe["%-rsi-period"] = ta.RSI(dataframe, timeperiod=period)
        dataframe["%-mfi-period"] = ta.MFI(dataframe, timeperiod=period)
        dataframe["%-adx-period"] = ta.ADX(dataframe, timeperiod=period)
        dataframe["%-sma-period"] = ta.SMA(dataframe, timeperiod=period)
        dataframe["%-ema-period"] = ta.EMA(dataframe, timeperiod=period)
        return dataframe

    def set_freqai_targets(self, dataframe: DataFrame, metadata, **kwargs) -> DataFrame:
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

配置示例：

```json
    "freqai": {
        "feature_parameters" : {
            "include_timeframes": ["5m","15m","4h"],
            "include_corr_pairlist": ["ETH/USD", "LINK/USD", "BNB/USD"],
            "label_period_candles": 24,
            "include_shifted_candles": 2,
            "indicator_periods_candles": [10, 20]
        },
    }
```

`include_timeframes` 指定对 `feature_engineering_expand_*()` 的每次调用使用哪些时间周期。
`include_corr_pairlist` 将特征也应用于关联交易对。
`include_shifted_candles` 指定要在特征集中包含的之前 K 线数量。

总特征数量 = `include_timeframes` 长度 × 特征数量 × `include_corr_pairlist` 长度 × `include_shifted_candles` × `indicator_periods_candles` 长度。

### 使用 `metadata` 获得更精细的控制

所有 `feature_engineering_*` 和 `set_freqai_targets()` 函数都被传入一个 `metadata` 字典，包含 FreqAI 正在为其自动化特征构建的 `pair`、`tf`（时间周期）和 `period` 信息。

```python
def feature_engineering_expand_all(self, dataframe: DataFrame, period, metadata, **kwargs) -> DataFrame:
    if metadata["tf"] == "1h":
        dataframe["%-roc-period"] = ta.ROC(dataframe, timeperiod=period)
```

这将阻止 `ta.ROC()` 被添加到 `"1h"` 以外的任何时间周期。

## 异常值检测

FreqAI 提供多种异常值检测方法：

- **差异指数 (DI)**：衡量新数据点与训练数据集的相似程度
- **支持向量机 (SVM)**：使用 `SGDOneClassSVM` 检测异常值
- **DBSCAN**：使用 DBSCAN 聚类算法识别异常值

### 数据降维：主成分分析 (PCA)

通过设置 `"principal_component_analysis": true`，FreqAI 将自动减小特征集的维度。

### 为时间重要性加权特征

`weight_factor` 参数根据数据点的近期性对训练数据点进行加权。

## 构建数据流水线

FreqAI 自动处理数据归一化，使用训练数据的均值和标准差对特征进行归一化。所有数据始终归一化到 -1 和 1 之间。