# 策略定制

本页介绍如何定制你的策略、添加新指标和设置交易规则。

## 开发你自己的策略

### 生成策略模板

```bash
freqtrade new-strategy --strategy AwesomeStrategy
```

这将从模板创建一个名为 `AwesomeStrategy` 的新策略，位于 `user_data/strategies/AwesomeStrategy.py`。

模板级别：`--template minimal`（空策略）、`--template advanced`（高级模板）。

### 策略的组成

一个策略文件包含构建策略逻辑所需的所有信息：

- OHLCV 格式的 K 线数据
- 指标
- 入场逻辑（信号）
- 出场逻辑（信号、最小 ROI、回调）
- 止损（固定/绝对、跟踪、回调）
- 定价 [可选]
- 仓位调整 [可选]

### 数据框

Freqtrade 使用 [pandas](https://pandas.pydata.org/) 存储/提供 K 线数据。数据框中的每一行对应图表上的一根 K 线。

pandas 比较必须使用向量化方式：

``` python
    # 正确方式
    dataframe.loc[
        (dataframe['rsi'] > 30)
    , 'enter_long'] = 1
```

### 自定义指标

在 `populate_indicators()` 方法中添加指标：

```python
def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe['sar'] = ta.SAR(dataframe)
    dataframe['adx'] = ta.ADX(dataframe)
    dataframe['rsi'] = ta.RSI(dataframe)
    dataframe['sma'] = ta.SMA(dataframe, timeperiod=40)
    return dataframe
```

Freqtrade 内置安装以下技术库：ta-lib、pandas-ta、technical。

### 策略启动期

某些指标有启动期，期间没有足够的数据来计算值或计算不正确。可以通过 `startup_candle_count` 属性来设置。

### 入场信号规则

在 `populate_entry_trend()` 中定义：

```python
def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe.loc[
        (
            (dataframe['rsi'] < 30) &
            (dataframe['volume'] > 0)
        ),
        'enter_long'] = 1
    return dataframe
```

### 出场信号规则

在 `populate_exit_trend()` 中定义：

```python
def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    dataframe.loc[
        (
            (dataframe['rsi'] > 70) &
            (dataframe['volume'] > 0)
        ),
        'exit_long'] = 1
    return dataframe
```

## 信号冲突

当同一根 K 线上同时存在入场和出场信号时，它们被视为冲突，不会发起订单。

## 常见错误

- **前瞻偏差**：使用未来数据。避免使用 `shift(-1)` 和在 `populate_*` 中使用 `iloc[-1]`
- **递归问题**：使用 `recursive-analysis` 命令检查
- **重绘**：使用不完整的 K 线数据。Freqtrade 不提供不完整 K 线

## 发送通知

策略可以通过 `self.dp.send_msg()` 发送自定义消息到 Telegram 或 Webhook。需要在配置中启用 `allow_custom_messages`。