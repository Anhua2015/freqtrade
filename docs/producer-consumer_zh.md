# 生产者 / 消费者模式

freqtrade 提供了一种机制，允许一个实例（也称为 `consumer`）使用消息 WebSocket 监听来自上游 freqtrade 实例（也称为 `producer`）的消息。主要是 `analyzed_df` 和 `whitelist` 消息。这允许在多个机器人之间重用已计算的指标（和信号），而无需多次计算。

请参阅 REST API 文档中的[消息 WebSocket](rest-api.md#message-websocket) 来设置你消息 WebSocket 的 `api_server` 配置（这将是你的生产者）。

!!! Note
    我们强烈建议将 `ws_token` 设置为某种随机且只有你自己知道的值，以防止对你的机器人进行未授权访问。

## 配置

通过在消费者的配置文件中添加 `external_message_consumer` 部分来启用对某个实例的订阅。

```json
{
    //...
   "external_message_consumer": {
        "enabled": true,
        "producers": [
            {
                "name": "default", // 可以是任何你喜欢的名称，默认是 "default"
                "host": "127.0.0.1", // 生产者 api_server 配置中的主机
                "port": 8080, // 生产者 api_server 配置中的端口
                "secure": false, // 使用安全的 WebSocket 连接，默认 false
                "ws_token": "sercet_Ws_t0ken" // 生产者 api_server 配置中的 ws_token
            }
        ],
        // 以下配置是可选的，通常不需要
        // "wait_timeout": 300,
        // "ping_timeout": 10,
        // "sleep_time": 10,
        // "remove_entry_exit_signals": false,
        // "message_size_limit": 8
    }
    //...
}
```

|  参数 | 描述 |
|------------|-------------|
| `enabled` | **必选。** 启用消费者模式。如果设置为 false，此部分中的所有其他设置将被忽略。<br>*默认值为 `false`。*<br> **数据类型：** 布尔值。 |
| `producers` | **必选。** 生产者列表 <br> **数据类型：** 数组。 |
| `producers.name` | **必选。** 此生产者的名称。如果使用多个生产者，必须在 `get_producer_pairs()` 和 `get_producer_df()` 的调用中使用此名称。<br> **数据类型：** 字符串 |
| `producers.host` | **必选。** 生产者的主机名或 IP 地址。<br> **数据类型：** 字符串 |
| `producers.port` | **必选。** 匹配上述主机的端口。<br>*默认值为 `8080`。*<br> **数据类型：** 整数 |
| `producers.secure` | **可选。** 在 WebSocket 连接中使用 SSL。默认 False。<br> **数据类型：** 字符串 |
| `producers.ws_token` | **必选。** 生产者在生产者上配置的 `ws_token`。<br> **数据类型：** 字符串 |
| | **可选设置** |
| `wait_timeout` | 如果未收到消息，再次 ping 前的超时时间。<br>*默认值为 `300`。*<br> **数据类型：** 整数 - 秒。 |
| `ping_timeout` | Ping 超时 <br>*默认值为 `10`。*<br> **数据类型：** 整数 - 秒。 |
| `sleep_time` | 重新尝试连接前的休眠时间。<br>*默认值为 `10`。*<br> **数据类型：** 整数 - 秒。 |
| `remove_entry_exit_signals` | 在收到数据框时从中移除信号列（将它们设为 0）。<br>*默认值为 `false`。*<br> **数据类型：** 布尔值。 |
| `initial_candle_limit` | 期望从生产者获取的初始 K 线数量。<br>*默认值为 `1500`。*<br> **数据类型：** 整数 - K 线数量。 |
| `message_size_limit` | 每条消息的大小限制<br>*默认值为 `8`。*<br> **数据类型：** 整数 - 兆字节。 |

作为在 `populate_indicators()` 中计算指标的替代（或补充），跟随者实例监听与生产者实例（或高级配置中的多个生产者实例）的消息连接，并为活跃白名单中的每个交易对请求生产者最近分析过的数据框。

消费者实例随后将拥有分析过的数据框的完整副本，而无需自行计算。

## 示例

### 示例 - 生产者策略

一个带有多个指标的简单策略。策略本身不需要特殊考虑。

```py
class ProducerStrategy(IStrategy):
    #...
    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        以标准 freqtrade 方式计算指标，然后可广播给其他实例
        """
        dataframe['rsi'] = ta.RSI(dataframe)
        bollinger = qtpylib.bollinger_bands(qtpylib.typical_price(dataframe), window=20, stds=2)
        dataframe['bb_lowerband'] = bollinger['lower']
        dataframe['bb_middleband'] = bollinger['mid']
        dataframe['bb_upperband'] = bollinger['upper']
        dataframe['tema'] = ta.TEMA(dataframe, timeperiod=9)

        return dataframe

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        为给定的数据框填充入场信号
        """
        dataframe.loc[
            (
                (qtpylib.crossed_above(dataframe['rsi'], self.buy_rsi.value)) &
                (dataframe['tema'] <= dataframe['bb_middleband']) &
                (dataframe['tema'] > dataframe['tema'].shift(1)) &
                (dataframe['volume'] > 0)
            ),
            'enter_long'] = 1

        return dataframe
```

!!! Tip "FreqAI"
    你可以使用此设置在强大的机器上设置 [FreqAI](freqai.md)，而在树莓派等简单机器上运行消费者，这些消费者可以以不同方式解读生产者生成的信号。

### 示例 - 消费者策略

一个逻辑等同的策略，自身不计算任何指标，但将拥有相同的分析数据框，可根据生产者的指标做出交易决策。在此示例中，消费者具有相同的入场条件，但这并非必需。消费者可以使用不同的逻辑来入场/出场，并仅使用指定的指标。

```py
class ConsumerStrategy(IStrategy):
    #...
    process_only_new_candles = False # 消费者必需

    _columns_to_expect = ['rsi_default', 'tema_default', 'bb_middleband_default']

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        使用 WebSocket API 从另一个 freqtrade 实例获取预填充的指标。
        使用 `self.dp.get_producer_df(pair)` 获取数据框
        """
        pair = metadata['pair']
        timeframe = self.timeframe

        producer_pairs = self.dp.get_producer_pairs()
        # 你可以通过以下方式指定从哪个生产者获取交易对：
        # self.dp.get_producer_pairs("my_other_producer")

        # 此函数返回分析过的数据框以及分析时间
        producer_dataframe, _ = self.dp.get_producer_df(pair)
        # 如果生产者使其可用，你可以获取其他数据：
        # self.dp.get_producer_df(
        #   pair,
        #   timeframe="1h",
        #   candle_type=CandleType.SPOT,
        #   producer_name="my_other_producer"
        # )

        if not producer_dataframe.empty:
            # 如果你打算直接传递生产者的入场/出场信号，
            # 请指定 ffill=False，否则会产生意外结果
            merged_dataframe = merge_informative_pair(dataframe, producer_dataframe,
                                                      timeframe, timeframe,
                                                      append_timeframe=False,
                                                      suffix="default")
            return merged_dataframe
        else:
            dataframe[self._columns_to_expect] = 0

        return dataframe

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        """
        为给定的数据框填充入场信号
        """
        # 如同我们自己计算一样使用数据框列
        dataframe.loc[
            (
                (qtpylib.crossed_above(dataframe['rsi_default'], self.buy_rsi.value)) &
                (dataframe['tema_default'] <= dataframe['bb_middleband_default']) &
                (dataframe['tema_default'] > dataframe['tema_default'].shift(1)) &
                (dataframe['volume'] > 0)
            ),
            'enter_long'] = 1

        return dataframe
```

!!! Tip "使用上游信号"
    通过设置 `remove_entry_exit_signals=false`，你也可以直接使用生产者的信号。它们应作为 `enter_long_default` 可用（假设使用了 `suffix="default"`）——可直接作为信号使用，或作为附加指标使用。