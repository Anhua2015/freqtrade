# 绘图

本页介绍如何绘制价格、指标和利润。

!!! Warning "已弃用"
    本页描述的命令（`plot-dataframe`、`plot-profit`）应被视为已弃用，处于维护模式。
    这主要是因为即使是中等大小的图表也可能导致性能问题，同时也因为"存储文件并在浏览器中打开"在 UI 角度上不太直观。

    虽然目前没有立即移除它们的计划，但它们不再积极维护——如果需要进行重大更改才能保持它们正常工作，可能会短期内移除。

    请使用 [FreqUI](freq-ui.md) 来满足绘图需求，它不存在同样的性能问题。

## 安装 / 设置

绘图模块使用 Plotly 库。你可以通过运行以下命令来安装 / 升级：

``` bash
pip install -U -r requirements-plot.txt
```

## 绘制价格和指标

`freqtrade plot-dataframe` 子命令显示一个包含三个子图的交互式图表：

* 主图包含 K 线图和跟随价格的指标（SMA/EMA）
* 成交量柱
* 由 `--indicators2` 指定的附加指标

![plot-dataframe](assets/plot-dataframe.png)

可能的参数：

--8<-- "commands/plot-dataframe.md"

示例：

``` bash
freqtrade plot-dataframe -p BTC/ETH --strategy AwesomeStrategy
```

`-p/--pairs` 参数可用于指定你想要绘制的交易对。

!!! Note
    `freqtrade plot-dataframe` 子命令为每个交易对生成一个图表文件。

指定自定义指标。
对主图使用 `--indicators1`，对下方子图使用 `--indicators2`（如果值处于与价格不同的范围内）。

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH --indicators1 sma ema --indicators2 macd
```

### 更多使用示例

要绘制多个交易对，用空格分隔它们：

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH XRP/ETH
```

要绘制一个时间范围（放大）

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH --timerange=20180801-20180805
```

要绘制数据库中存储的交易，使用 `--db-url` 结合 `--trade-source DB`：

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy --db-url sqlite:///tradesv3.dry_run.sqlite -p BTC/ETH --trade-source DB
```

要绘制回测结果中的交易，使用 `--export-filename <filename>`

``` bash
freqtrade plot-dataframe --strategy AwesomeStrategy --export-filename user_data/backtest_results/backtest-result.json -p BTC/ETH
```

### 数据框图表基础

![plot-dataframe2](assets/plot-dataframe2.png)

`plot-dataframe` 子命令需要回测数据、一个策略，以及包含与策略对应交易的回测结果文件或数据库。

生成的图表将包含以下元素：

* 绿色三角形：策略的买入信号。（注意：并非每个买入信号都会产生交易，与青色圆圈对比。）
* 红色三角形：策略的卖出信号。（同样，并非每个卖出信号都会终止交易，与红色和绿色方块对比。）
* 青色圆圈：交易入场点。
* 红色方块：亏损或 0% 利润交易的出场点。
* 绿色方块：盈利交易的出场点。
* 值与 K 线尺度对应的指标（例如 SMA/EMA），如 `--indicators1` 指定。
* 成交量（主图底部的柱状图）。
* 成交量柱下方不同尺度的指标（例如 MACD、RSI），如 `--indicators2` 指定。

!!! Note "布林带"
    如果列 `bb_lowerband` 和 `bb_upperband` 存在，布林带会自动添加到图表中，并以从下轨到上轨的浅蓝色区域呈现。

#### 高级图表配置

可以在策略的 `plot_config` 参数中指定高级图表配置。

使用 `plot_config` 时的额外功能包括：

* 为每个指标指定颜色
* 指定额外的子图
* 指定指标对以填充之间的区域

以下示例图表配置为指标指定了固定颜色。否则，连续的图表可能每次产生不同的配色方案，使比较变得困难。
它还允许多个子图同时显示 MACD 和 RSI。

图表类型可以使用 `type` 键配置。可能的类型有：

* `scatter` 对应散点图。
* `bar` 对应柱状图。

`plotly.graph_objects.*` 构造函数的额外参数可以在 `plotly` 字典中指定——这些仅在使用 plotly 作为绘图库时支持，使用 freq-ui 时将被忽略。

示例配置及解释流程的行内注释：

``` python
@property
def plot_config(self):
    """
        构建返回字典有多种解决方案。
        唯一重要的是返回值。
        示例：
            plot_config = {'main_plot': {}, 'subplots': {}}
    """
    plot_config = {}
    plot_config['main_plot'] = {
        # 主图指标配置。
        # 假设指定了 2 个参数，emashort 和 emalong。
        f'ema_{self.emashort.value}': {'color': 'red'},
        f'ema_{self.emalong.value}': {'color': '#CCCCCC'},
        # 省略 color，则随机选择颜色。
        'sar': {},
        # 填充 senkou_a 和 senkou_b 之间的区域
        'senkou_a': {
            'color': 'green', #可选
            'fill_to': 'senkou_b',
            'fill_label': 'Ichimoku Cloud', #可选
            'fill_color': 'rgba(255,76,46,0.2)', #可选
        },
        # 也绘制 senkou_b。不仅仅是其间的区域。
        'senkou_b': {}
    }
    plot_config['subplots'] = {
         # 创建子图 MACD
        "MACD": {
            'macd': {'color': 'blue', 'fill_to': 'macdhist'},
            'macdsignal': {'color': 'orange'},
            'macdhist': {'type': 'bar', 'plotly': {'opacity': 0.9}}
        },
        # 额外的子图 RSI
        "RSI": {
            'rsi': {'color': 'red'}
        }
    }

    return plot_config
```

??? Note "作为属性（之前的方法）"
    也可以将 `plot_config` 分配为属性（这曾经是默认方式）。
    这有一个缺点，即策略参数不可用，导致某些配置无法工作。

    ``` python
        plot_config = {
            'main_plot': {
                # 主图指标配置。
                # 指定 `ema10` 为红色，`ema50` 为灰色调
                'ema10': {'color': 'red'},
                'ema50': {'color': '#CCCCCC'},
                # 省略 color，则随机选择颜色。
                'sar': {},
            # 填充 senkou_a 和 senkou_b 之间的区域
            'senkou_a': {
                'color': 'green', #可选
                'fill_to': 'senkou_b',
                'fill_label': 'Ichimoku Cloud', #可选
                'fill_color': 'rgba(255,76,46,0.2)', #可选
            },
            # 也绘制 senkou_b。不仅仅是其间的区域。
            'senkou_b': {}
            },
            'subplots': {
                # 创建子图 MACD
                "MACD": {
                    'macd': {'color': 'blue', 'fill_to': 'macdhist'},
                    'macdsignal': {'color': 'orange'},
                    'macdhist': {'type': 'bar', 'plotly': {'opacity': 0.9}}
                },
                # 额外的子图 RSI
                "RSI": {
                    'rsi': {'color': 'red'}
                }
            }
        }
    ```

!!! Note
    以上配置假设 `ema10`、`ema50`、`senkou_a`、`senkou_b`、`macd`、`macdsignal`、`macdhist` 和 `rsi` 是策略创建的 DataFrame 中的列。

!!! Warning
    `plotly` 参数仅支持 plotly 库，不适用于 freq-ui。

!!! Note "交易仓位调整"
    如果使用了 `position_adjustment_enable` / `adjust_trade_position()`，交易初始买入价格是多个订单的平均值，交易开始价格很可能出现在 K 线范围之外。

## 绘制利润

![plot-profit](assets/plot-profit.png)

`plot-profit` 子命令显示一个包含三个图的交互式图表：

* 所有交易对的平均收盘价。
* 回测产生的汇总利润。
注意这不是真实世界的利润，而更像是一种估算。
* 每个单独交易对的利润。
* 交易并行性。
* 水下曲线（回撤期间）。

第一个图有助于了解整体市场的走势。

第二个图将显示你的算法是否有效。
也许你想要一种稳定地赚取小额利润的算法，或者一种操作频率较低但波动较大的算法。
此图还将突出显示最大回撤期的开始（和结束）。

第三个图可用于发现异常值，即导致利润突增的交易对事件。

第四个图可帮助你分析交易并行性，显示 max_open_trades 被占满的频率。

`freqtrade plot-profit` 子命令的可能选项：

--8<-- "commands/plot-profit.md"

`-p/--pairs` 参数可用于限制此计算考虑的货币对。

示例：

使用自定义回测导出文件

``` bash
freqtrade plot-profit  -p LTC/BTC --export-filename user_data/backtest_results/backtest-result.json
```

使用自定义数据库

``` bash
freqtrade plot-profit  -p LTC/BTC --db-url sqlite:///tradesv3.sqlite --trade-source DB
```

``` bash
freqtrade --datadir user_data/data/binance_save/ plot-profit -p LTC/BTC
```