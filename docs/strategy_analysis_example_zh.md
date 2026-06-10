# 策略分析示例

调试策略可能非常耗时。Freqtrade 提供辅助函数来可视化原始数据。
以下假设你使用 SampleStrategy，来自 Binance 的 5m 时间周期数据，并已将它们下载到默认位置的数据目录中。
请参阅[文档](https://www.freqtrade.io/en/stable/data-download/)了解更多详细信息。

## 设置

### 将工作目录切换到仓库根目录

```python
import os
from pathlib import Path

# 更改目录
# 修改此单元格以确保输出显示正确的路径。
# 将所有路径定义为相对于单元格输出中显示的项目根目录
project_root = "somedir/freqtrade"
i = 0
try:
    os.chdir(project_root)
    if not Path("LICENSE").is_file():
        i = 0
        while i < 4 and (not Path("LICENSE").is_file()):
            os.chdir(Path(Path.cwd(), "../"))
            i += 1
        project_root = Path.cwd()
except FileNotFoundError:
    print("Please define the project root relative to the current directory")
print(Path.cwd())
```

### 配置 Freqtrade 环境

```python
from freqtrade.configuration import Configuration

# 根据你的需要自定义这些。

# 初始化空配置对象
config = Configuration.from_files([])
# 可选（推荐），使用现有配置文件
# config = Configuration.from_files(["user_data/config.json"])

# 定义一些常量
config["timeframe"] = "5m"
# 策略类名
config["strategy"] = "SampleStrategy"
# 数据位置
data_location = config["datadir"]
# 要分析的交易对——此处仅使用一个交易对
pair = "BTC/USDT"
```

```python
# 使用上面设置的值加载数据
from freqtrade.data.history import load_pair_history
from freqtrade.enums import CandleType

candles = load_pair_history(
    datadir=data_location,
    timeframe=config["timeframe"],
    pair=pair,
    data_format="json",  # 确保将其更新为你的数据格式
    candle_type=CandleType.SPOT,
)

# 确认成功
print(f"Loaded {len(candles)} rows of data for {pair} from {data_location}")
candles.head()
```

## 加载和运行策略
* 每次策略文件更改时重新运行

```python
# 使用上面设置的值加载策略
from freqtrade.data.dataprovider import DataProvider
from freqtrade.resolvers import StrategyResolver

strategy = StrategyResolver.load_strategy(config)
strategy.dp = DataProvider(config, None, None)
strategy.ft_bot_start()

# 使用策略生成买入/卖出信号
df = strategy.analyze_ticker(candles, {"pair": pair})
df.tail()
```

### 显示交易详情

* 注意，使用 `data.head()` 也可以，但大多数指标在数据框顶部有一些"启动"数据。
* 一些可能的问题：
    * 数据框末尾有 NaN 值的列
    * 在 `crossed*()` 函数中使用的列具有完全不同的单位
* 与完整回测的比较：
    * 从 `analyze_ticker()` 输出中一个交易对得到 200 个买入信号并不一定意味着回测期间会产生 200 笔交易。
    * 假设你仅使用一个条件如 `df['rsi'] < 30` 作为买入条件，这将按顺序为每个交易对生成多个"买入"信号（直到 rsi 回到 > 29）。机器人只会在这些信号中的第一个上买入（且仅在交易槽位（"max_open_trades"）仍然可用时），或在某个中间信号上买入，一旦有"槽位"变得可用。

```python
# 报告结果
print(f"Generated {df['enter_long'].sum()} entry signals")
data = df.set_index("date", drop=False)
data.tail()
```

## 将现有对象加载到 Jupyter notebook 中

以下单元格假设你已经使用 CLI 生成了数据。
它们将允许你深入挖掘你的结果，并执行那些否则由于信息过载而使得输出难以理解的分析。

### 加载回测结果到 pandas 数据框

分析交易数据框（也用于下面的绘图）

```python
from freqtrade.data.btanalysis import load_backtest_data, load_backtest_stats

# 如果 backtest_dir 指向一个目录，它将自动加载最新的回测文件。
backtest_dir = config["user_data_dir"] / "backtest_results"
# backtest_dir 也可以指向特定文件
# backtest_dir = (
#   config["user_data_dir"] / "backtest_results/backtest-result-2020-07-01_20-04-22.json"
# )
```

```python
# 你可以使用以下命令获取完整的回测统计信息。
# 这包含用于生成回测结果的所有信息。
stats = load_backtest_stats(backtest_dir)

strategy = "SampleStrategy"
# 所有统计信息可按策略获取，因此如果在回测期间使用了 `--strategy-list`，
# 这里也会反映出来。
# 示例用法：
print(stats["strategy"][strategy]["results_per_pair"])
# 获取此回测使用的交易对列表
print(stats["strategy"][strategy]["pairlist"])
# 获取市场变化（回测期间所有交易对从开始到结束的平均变化）
print(stats["strategy"][strategy]["market_change"])
# 最大回撤（绝对值）
print(stats["strategy"][strategy]["max_drawdown_abs"])
# 最大回撤开始和结束
print(stats["strategy"][strategy]["drawdown_start"])
print(stats["strategy"][strategy]["drawdown_end"])

# 获取策略比较（仅在比较了多个策略时相关）
print(stats["strategy_comparison"])
```

```python
# 将回测交易加载为数据框
trades = load_backtest_data(backtest_dir)

# 显示每个交易对的数值计数
trades.groupby("pair")["exit_reason"].value_counts()
```

## 绘制每日利润 / 净值曲线

```python
# 绘制净值曲线（从第 1 天 0 开始，加上每个回测日的每日利润）

import pandas as pd
import plotly.express as px

from freqtrade.configuration import Configuration
from freqtrade.data.btanalysis import load_backtest_stats

# strategy = 'SampleStrategy'
# config = Configuration.from_files(["user_data/config.json"])
# backtest_dir = config["user_data_dir"] / "backtest_results"

stats = load_backtest_stats(backtest_dir)
strategy_stats = stats["strategy"][strategy]

df = pd.DataFrame(columns=["dates", "equity"], data=strategy_stats["daily_profit"])
df["equity_daily"] = df["equity"].cumsum()

fig = px.line(df, x="dates", y="equity_daily")
fig.show()
```

### 将实盘交易结果加载到 pandas 数据框

如果你已经进行了一些交易并希望分析你的表现

```python
from freqtrade.data.btanalysis import load_trades_from_db

# 从数据库获取交易
trades = load_trades_from_db("sqlite:///tradesv3.sqlite")

# 显示结果
trades.groupby("pair")["exit_reason"].value_counts()
```

## 分析加载的交易以了解交易并行性
这对于在使用回测结合非常高的 `max_open_trades` 设置时找到最佳的 `max_open_trades` 参数非常有用。

`analyze_trade_parallelism()` 返回一个时间序列数据框，其中包含 "open_trades" 列，指定每根 K 线的未平仓交易数量。

```python
from freqtrade.data.btanalysis import analyze_trade_parallelism

# 分析上述内容
parallel_trades = analyze_trade_parallelism(trades, "5m")

parallel_trades.plot()
```

## 绘制结果

Freqtrade 基于 plotly 提供交互式绘图功能。

```python
from freqtrade.plot.plotting import generate_candlestick_graph

# 限制图表时间范围以保持 plotly 快速和灵敏

# 过滤交易到一个交易对
trades_red = trades.loc[trades["pair"] == pair]

data_red = data["2019-06-01":"2019-06-10"]
# 生成 K 线图
graph = generate_candlestick_graph(
    pair=pair,
    data=data_red,
    trades=trades_red,
    indicators1=["sma20", "ema50", "ema55"],
    indicators2=["rsi", "macd", "macdsignal", "macdhist"],
)
```

```python
# 内联显示图表
# graph.show()

# 在单独窗口中渲染图表
graph.show(renderer="browser")
```

## 将每笔交易的平均利润绘制为分布图

```python
import plotly.figure_factory as ff

hist_data = [trades.profit_ratio]
group_labels = ["profit_ratio"]  # 数据集名称

fig = ff.create_distplot(hist_data, group_labels, bin_size=0.01)
fig.show()
```

如果你有关于如何最佳分析数据的想法，欢迎提交 issue 或 Pull Request 来改进本文档。