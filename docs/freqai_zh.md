![freqai-logo](assets/freqai_doc_logo.svg)

# FreqAI

## 简介

FreqAI 是一个软件，旨在自动化与训练预测性机器学习模型相关的各种任务，以根据一组输入信号生成市场预测。总的来说，FreqAI 旨在成为一个沙箱，用于在实时数据上轻松部署稳健的机器学习库（[详情](#freqai-position-in-open-source-machine-learning-landscape)）。

!!! Note
    FreqAI 是并将永远是一个非盈利的开源项目。FreqAI *不*发行加密货币代币，FreqAI *不*出售信号，FreqAI 除了当前的 [freqtrade 文档](https://www.freqtrade.io/en/stable/freqai/)之外没有其他域名。

功能包括：

* **自适应重新训练** - 在[实盘部署](freqai-running.md#live-deployments)期间重新训练模型，以有监督的方式自适应市场
* **快速特征工程** - 基于用户创建的简单策略创建大规模丰富的[特征集](freqai-feature-engineering.md#feature-engineering)（10k+ 特征）
* **高性能** - 线程化允许在独立线程（或 GPU，如果可用）上进行自适应模型重新训练，与模型推理（预测）和机器人交易操作分开。最新模型和数据保存在 RAM 中以实现快速推理
* **真实回测** - 使用自动重新训练的[回测模块](freqai-running.md#backtesting)在历史数据上模拟自适应训练
* **可扩展性** - 通用且稳健的架构允许整合 Python 中可用的任何[机器学习库/方法](freqai-configuration.md#using-different-prediction-models)。目前有八个示例可用，包括分类器、回归器和卷积神经网络
* **智能异常值移除** - 使用多种[异常值检测技术](freqai-feature-engineering.md#outlier-detection)从训练和预测数据集中移除异常值
* **崩溃恢复能力** - 将训练好的模型存储到磁盘，以便从崩溃中快速轻松地重新加载，并[清理过时文件](freqai-running.md#purging-old-model-data)以持续进行模拟/实盘运行
* **自动数据归一化** - 以智能且统计上安全的方式[归一化数据](freqai-feature-engineering.md#building-the-data-pipeline)
* **自动数据下载** - 计算数据下载的时间范围并更新历史数据（在实盘部署中）
* **传入数据清理** - 在训练和模型推理之前安全处理 NaN
* **降维** - 通过[主成分分析](freqai-feature-engineering.md#data-dimensionality-reduction-with-principal-component-analysis)减小训练数据的大小
* **部署机器人集群** - 设置一个机器人来训练模型，同时由一组[消费者](producer-consumer.md)使用信号。

## 快速开始

快速测试 FreqAI 最简单的方法是在模拟模式下使用以下命令运行它：

```bash
freqtrade trade --config config_examples/config_freqai.example.json --strategy FreqaiExampleStrategy --freqaimodel LightGBMRegressor --strategy-path freqtrade/templates
```

你将看到自动数据下载的启动过程，随后是同时进行的训练和交易。

!!! danger "非生产用途"
    Freqtrade 源代码中提供的示例策略旨在展示/测试各种 FreqAI 功能。它也被设计为在小型计算机上运行，以便用作开发者和用户之间的基准。它*不*被设计用于生产环境。

示例策略、预测模型和配置文件可作为起点，分别位于 `freqtrade/templates/FreqaiExampleStrategy.py`、`freqtrade/freqai/prediction_models/LightGBMRegressor.py` 和 `config_examples/config_freqai.example.json`。

## 总体方法

你向 FreqAI 提供一组自定义*基础指标*（与[典型 Freqtrade 策略](strategy-customization.md)中的方式相同）以及目标值（*标签*）。对于白名单中的每个交易对，FreqAI 训练一个模型来根据自定义指标的输入预测目标值。然后以预定的频率持续重新训练模型，以适应市场条件。FreqAI 既提供回测策略（在历史数据上模拟定期重新训练的现实），也支持部署模拟/实盘运行。在模拟/实盘条件下，FreqAI 可以设置为在后台线程中持续重新训练，以保持模型尽可能最新。

以下是算法概述，解释了数据处理流水线和模型使用：

![freqai-algo](assets/freqai_algo.jpg)

### 重要的机器学习词汇

**特征（Features）** - 基于历史数据训练模型的参数。单根 K 线的所有特征存储为一个向量。在 FreqAI 中，你可以从策略中构建的任何东西构建特征数据集。

**标签（Labels）** - 模型训练所朝向的目标值。每个特征向量与你在策略中定义的单个标签关联。这些标签有意地看向未来，是你训练模型能够预测的目标。

**训练（Training）** - "教授"模型将特征集与关联标签匹配的过程。不同类型的模型以不同的方式"学习"，这意味着对于特定应用场景，某一种可能比另一种更好。有关 FreqAI 中已实现的不同模型的更多信息，请参见[此处](freqai-configuration.md#using-different-prediction-models)。

**训练数据（Train data）** - 特征数据集的一个子集，在训练期间喂给模型以"教授"模型如何预测目标。这些数据直接影响模型中的权重连接。

**测试数据（Test data）** - 特征数据集的一个子集，用于评估训练后模型的性能。这些数据不影响模型内的节点权重。

**推理（Inferencing）** - 将训练好的模型喂入新的未见过的数据，模型将对这些数据进行预测的过程。

## 安装前提条件

正常的 Freqtrade 安装过程会询问你是否要安装 FreqAI 依赖项。如果你希望使用 FreqAI，应对此问题回答"yes"。如果你未回答 yes，可以在安装后使用以下命令手动安装这些依赖项：

``` bash
pip install -r requirements-freqai.txt
```

!!! Note
    Catboost 不会安装在低功耗 ARM 设备（树莓派）上，因为它不为此平台提供 wheel。

### 与 Docker 一起使用

如果你使用 Docker，可以使用专用的 `:freqai` 标签，其中包含 FreqAI 依赖项。因此，你可以将 docker compose 文件中的 image 行替换为 `image: freqtradeorg/freqtrade:stable_freqai`。此镜像包含常规的 FreqAI 依赖项。与原生安装类似，Catboost 在 ARM 设备上不可用。如果你想使用 PyTorch 或强化学习，应使用 torch 或 RL 标签：`image: freqtradeorg/freqtrade:stable_freqaitorch`、`image: freqtradeorg/freqtrade:stable_freqairl`。

!!! note "docker-compose-freqai.yml"
    我们在 `docker/docker-compose-freqai.yml` 中为此提供了显式的 docker-compose 文件——可通过 `docker compose -f docker/docker-compose-freqai.yml run ...` 使用——或可复制替换原始的 docker 文件。此 docker-compose 文件还包含一个（已禁用的）部分，用于在 docker 容器中启用 GPU 资源。这显然假设系统有可用的 GPU 资源。

### FreqAI 在开源机器学习生态中的定位

预测基于混沌时间序列的系统（如股票/加密货币市场）需要一套广泛的工具，用于测试各种假设。幸运的是，稳健的机器学习库（如 `scikit-learn`）的近期成熟为研究开辟了广泛的可能性。来自不同领域的科学家现在可以轻松地在大量成熟的机器学习算法上原型化他们的研究。同样，这些用户友好的库使"公民科学家"能够使用他们的基本 Python 技能进行数据探索。然而，在历史和实时混沌数据源上利用这些机器学习库在逻辑上可能困难且昂贵。此外，稳健的数据收集、存储和处理也带来了不同的挑战。`FreqAI` 旨在提供一个通用且可扩展的开源框架，面向自适应建模的实盘部署以进行市场预测。`FreqAI` 框架实际上是一个面向开源机器学习库丰富世界的沙箱。在 `FreqAI` 沙箱内，用户发现他们可以组合各种第三方库，在免费的实时 24/7 混沌数据源——加密货币交易所数据——上测试创造性假设。

### 引用 FreqAI

FreqAI 已[发表在 Journal of Open Source Software](https://joss.theoj.org/papers/10.21105/joss.04864)上。如果你发现 FreqAI 对你的研究有用，请使用以下引用：

```bibtex
@article{Caulk2022, 
    doi = {10.21105/joss.04864},
    url = {https://doi.org/10.21105/joss.04864},
    year = {2022}, publisher = {The Open Journal},
    volume = {7}, number = {80}, pages = {4864},
    author = {Robert A. Caulk and Elin Törnquist and Matthias Voppichler and Andrew R. Lawless and Ryan McMullan and Wagner Costa Santos and Timothy C. Pogue and Johan van der Vlugt and Stefan P. Gehring and Pascal Schmidt},
    title = {FreqAI: generalizing adaptive modeling for chaotic time-series market forecasts},
    journal = {Journal of Open Source Software} } 
```

## 常见陷阱

FreqAI 不能与动态 `VolumePairlists`（或任何动态添加和移除交易对的交易对列表过滤器）结合使用。这是出于性能原因——FreqAI 依赖快速预测/重新训练。为了有效做到这一点，它需要在模拟/实盘实例开始时下载所有训练数据。FreqAI 会自动存储和追加新 K 线以用于未来的重新训练。这意味着如果新的交易对在模拟运行后期通过成交量交易对列表到达，它将没有准备好的数据。然而，FreqAI 可以与 `ShufflePairlist` 或保持总交易对列表不变（但根据成交量重新排序交易对）的 `VolumePairlist` 一起使用。

## 额外学习资料

这里我们整理了一些外部资料，提供对 FreqAI 各个组件的更深入了解：

- [实时对战：使用 XGBoost 和 CatBoost 对金融市场数据进行自适应建模](https://emergentmethods.medium.com/real-time-head-to-head-adaptive-modeling-of-financial-market-data-using-xgboost-and-catboost-995a115a7495)
- [FreqAI - 从价格到预测](https://emergentmethods.medium.com/freqai-from-price-to-prediction-6fadac18b665)

## 支持

你可以在多个地方找到 FreqAI 的支持，包括 [Freqtrade discord](https://discord.gg/Jd8JYeWHc4)、专门的 [FreqAI discord](https://discord.gg/7AMWACmbjT) 以及 [github issues](https://github.com/freqtrade/freqtrade/issues)。

## 致谢

FreqAI 由一群各自为项目贡献特定技能的个人开发。

概念和软件开发：
Robert Caulk @robcaulk

理论思考和数据分析：
Elin Törnquist @th0rntwig

代码审查和软件架构思考：
@xmatthias

软件开发：
Wagner Costa @wagnercosta
Emre Suzen @aemr3
Timothy Pogue @wizrds

测试和错误报告：
Stefan Gehring @bloodhunter4rc, @longyu, Andrew Lawless @paranoidandy, Pascal Schmidt @smidelis, Ryan McMullan @smarmau, Juha Nykänen @suikula, Johan van der Vlugt @jooopiert, Richárd Józsa @richardjosza