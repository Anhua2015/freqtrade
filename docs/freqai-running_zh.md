# 运行 FreqAI

训练和部署自适应机器学习模型有两种方式——实盘部署和历史回测。在这两种情况下，FreqAI 都会运行/模拟模型的定期重新训练，如下图所示：

![freqai-window](assets/freqai_moving-window.jpg)

## 实盘部署

FreqAI 可以使用以下命令在模拟/实盘模式下运行：

```bash
freqtrade trade --strategy FreqaiExampleStrategy --config config_freqai.example.json --freqaimodel LightGBMRegressor
```

启动后，FreqAI 将根据配置设置开始训练一个新模型，带有新的 `identifier`。训练完成后，模型将用于对传入的 K 线进行预测，直到有新模型可用。新模型通常尽可能频繁地生成，FreqAI 管理内部交易对队列以尽量保持所有模型同样新。FreqAI 将始终使用最近训练的模型对传入的实时数据进行预测。如果你不希望 FreqAI 尽可能频繁地重新训练新模型，可以设置 `live_retrain_hours` 来告诉 FreqAI 在训练新模型之前至少等待该小时数。此外，你可以设置 `expired_hours` 来告诉 FreqAI 避免使用超过该小时数的模型进行预测。

训练好的模型默认保存到磁盘，以便在回测期间或崩溃后重用。你可以选择[清理旧模型](#purging-old-model-data)以节省磁盘空间，在配置中设置 `"purge_old_models": true`。

要从保存的回测模型（或之前崩溃的模拟/实盘会话）启动模拟/实盘运行，你只需指定特定模型的 `identifier`：

```json
    "freqai": {
        "identifier": "example",
        "live_retrain_hours": 0.5
    }
```

在这种情况下，虽然 FreqAI 将以预训练模型启动，它仍会检查自模型训练以来已过去了多少时间。如果自加载的模型结束以来已过去一个完整的 `live_retrain_hours`，FreqAI 将开始训练一个新模型。

### 自动数据下载

FreqAI 自动下载确保通过定义的 `train_period_days` 和 `startup_candle_count`（这些参数的详细描述见[参数表](freqai-parameter-table.md)）训练模型所需的数据量。

### 保存预测数据

特定 `identifier` 模型生命周期内进行的所有预测都存储在 `historic_predictions.pkl` 中，以便在崩溃或配置更改后重新加载。

### 清理旧模型数据

FreqAI 在每次成功训练后存储新的模型文件。随着新模型生成以适应新的市场条件，这些文件会变得过时。如果你计划让 FreqAI 长时间运行并进行高频率重新训练，你应该在配置中启用 `purge_old_models`：

```json
    "freqai": {
        "purge_old_models": 4,
    }
```

这将自动清除超过最近训练的四个模型之外的所有旧模型，以节省磁盘空间。输入 "0" 将永远不清除任何模型。

## 回测

FreqAI 回测模块可以使用以下命令执行：

```bash
freqtrade backtesting --strategy FreqaiExampleStrategy --strategy-path freqtrade/templates --config config_examples/config_freqai.example.json --freqaimodel LightGBMRegressor --timerange 20210501-20210701
```

如果此命令从未在现有配置文件下执行过，FreqAI 将为每个交易对、扩展后的 `--timerange` 内的每个回测窗口训练一个新模型。

回测模式需要在部署前[下载必要的数据](#downloading-data-to-cover-the-full-backtest-period)（与模拟/实盘模式下 FreqAI 自动处理数据下载不同）。你需要注意下载数据的时间范围大于回测时间范围。这是因为 FreqAI 需要在期望的回测时间范围之前就有数据来训练模型，使其能够在设定的回测时间范围的第一根 K 线上进行预测。关于如何计算要下载数据的更多细节，请参见[此处](#deciding-the-size-of-the-sliding-training-window-and-backtesting-duration)。

!!! Note "模型重用"
    一旦训练完成，你可以使用相同的配置文件再次执行回测，FreqAI 将找到训练好的模型并加载它们，而不是花费时间训练。如果你想在策略中调整（甚至超参优化）买入和卖出条件，这非常有用。如果你*希望*使用相同的配置文件重新训练新模型，只需更改 `identifier`。
    这样，你可以通过简单指定 `identifier` 返回到使用你想要的任何模型。

!!! Note
    回测为每个回测窗口调用一次 `set_freqai_targets()`（其中窗口数量是完整回测时间范围除以 `backtest_period_days` 参数）。这样做意味着目标模拟了模拟/实盘行为，没有前瞻偏差。然而，`feature_engineering_*()` 中的特征定义是在整个训练时间范围上一次性执行的。这意味着你应该确保特征不会向前看到未来。
    有关前瞻偏差的更多细节，请参见[常见错误](strategy-customization.md#common-mistakes-when-developing-strategies)。

---

### 保存回测预测数据

为了允许调整你的策略（**不是**特征！），FreqAI 将在回测期间自动保存预测数据，以便它们可以被未来的回测和使用相同 `identifier` 模型的实盘运行重用。这提供了面向**高层次超参优化**入场/出场条件的性能增强。

在 `unique-id` 文件夹中会创建一个名为 `backtesting_predictions` 的额外目录，其中包含以 `feather` 格式存储的所有预测。

要更改你的**特征**，你**必须**在配置中设置一个新的 `identifier` 来告知 FreqAI 训练新模型。

要将特定回测期间生成的模型保存下来，以便从中启动实盘部署而不是训练新模型，你必须在配置中将 `save_backtest_models` 设置为 `True`。

!!! Note
    为确保模型可以重用，freqAI 将以长度为 1 的数据框调用你的策略。
    如果你的策略需要超过此数量的数据来生成相同的特征，则无法将回测预测重用于实盘部署，需要为每次新的回测更新你的 `identifier`。

!!! Danger "安全通知"
    从磁盘加载保存的模型可能引发安全问题，如果使用了远程模型文件（你从互联网下载或从不受信任的来源接收的文件），因为必须设置 `weights_only=False`，这可能会产生安全问题。
    只要你只加载自己训练的模型，就没有风险。

### 回测实盘收集的预测

FreqAI 允许你通过回测参数 `--freqai-backtest-live-models` 重用实盘历史预测。当你想重用在模拟/实盘中生成的预测以进行比较或其他研究时，这很有用。
`--timerange` 参数不需要指定，因为它将根据历史预测文件中的数据自动计算。

### 下载数据以覆盖完整回测期

对于实盘/模拟部署，FreqAI 将自动下载必要的数据。然而，要使用回测功能，你需要使用 `download-data` 下载必要的数据（详细说明见[此处](data-download.md#data-downloading)）。你需要仔细注意理解需要下载多少*额外*数据，以确保在回测时间范围开始*之前*有足够数量的训练数据。额外数据量可以通过将时间范围的开始日期从期望的回测时间范围开始处向后移动 `train_period_days` 和 `startup_candle_count`（这些参数的详细描述见[参数表](freqai-parameter-table.md)）来粗略估算。

例如，使用[示例配置](freqai-configuration.md#setting-up-the-configuration-file)对 `--timerange 20210501-20210701` 进行回测，该配置将 `train_period_days` 设置为 30，`startup_candle_count: 40` 在最大 `include_timeframes` 为 1h 的情况下，下载数据的开始日期需要是 `20210501` - 30 天 - 40 * 1h / 24 小时 = 20210330（比期望的训练时间范围开始早 31.7 天）。

### 决定滑动训练窗口的大小和回测持续时间

回测时间范围由典型的 `--timerange` 参数在配置文件中定义。滑动训练窗口的持续时间由 `train_period_days` 设置，而 `backtest_period_days` 是滑动回测窗口，两者均以天数为单位（`backtest_period_days` 可以是浮点数，以指示模拟/实盘模式中低于一天的重新训练）。在所示的[示例配置](freqai-configuration.md#setting-up-the-configuration-file)中（位于 `config_examples/config_freqai.example.json`），用户要求 FreqAI 使用 30 天的训练期，并在随后的 7 天进行回测。模型训练后，FreqAI 将对随后的 7 天进行回测。"滑动窗口"然后向前移动一周（模拟 FreqAI 在实盘模式中每周重新训练一次），新模型使用之前的 30 天（包括前一个模型用于回测的 7 天）进行训练。这重复进行直到 `--timerange` 结束。这意味着如果你设置 `--timerange 20210501-20210701`，FreqAI 将在 `--timerange` 结束时训练了 8 个独立的模型（因为完整范围包含 8 周）。

!!! Note
    虽然允许使用小数的 `backtest_period_days`，但你应该知道 `--timerange` 会除以该值来确定 FreqAI 需要训练多少个模型才能完成完整范围的回测。例如，设置 `--timerange` 为 10 天，`backtest_period_days` 为 0.1，FreqAI 需要为每个交易对训练 100 个模型才能完成完整回测。因此，FreqAI 自适应训练的真实回测将花费*非常*长的时间。全面测试模型的最佳方法是进行模拟运行并让它持续训练。在这种情况下，回测将花费与模拟运行完全相同的时间。

## 定义模型过期

在模拟/实盘模式期间，FreqAI 按顺序训练每个交易对（在与主要 Freqtrade 机器人分开的线程/GPU 上）。这意味着模型之间总是存在时间差异。如果你在训练 50 个交易对，每个交易对需要 5 分钟来训练，最老的模型将超过 4 小时。如果策略的特征时间尺度（目标交易持续时间）小于 4 小时，这可能不理想。你可以通过设置配置文件中的 `expiration_hours` 来决定仅在模型存在时间少于某个小时数时才进行交易：

```json
    "freqai": {
        "expiration_hours": 0.5,
    }
```

在所示的示例配置中，用户仅允许对存在时间少于 0.5 小时的模型进行预测。

## 控制模型学习过程

模型训练参数对所选的机器学习库是唯一的。FreqAI 允许你使用配置文件中的 `model_training_parameters` 字典为任何库设置任何参数。示例配置（位于 `config_examples/config_freqai.example.json`）显示了与 `Catboost` 和 `LightGBM` 关联的一些示例参数，但你可以添加这些库中任何可用的参数或你选择实现的任何其他机器学习库的参数。

数据拆分参数在 `data_split_parameters` 中定义，可以是与 scikit-learn 的 `train_test_split()` 函数关联的任何参数。`train_test_split()` 有一个名为 `shuffle` 的参数，允许洗牌数据或保持原样。这对于避免使用时间自相关数据进行训练偏差特别有用。有关这些参数的更多详细信息，请访问 [scikit-learn 网站](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html)（外部网站）。

FreqAI 特定参数 `label_period_candles` 定义了用于 `labels` 的偏移量（未来 K 线数量）。在所示的[示例配置](freqai-configuration.md#setting-up-the-configuration-file)中，用户要求 `labels` 是未来 24 根 K 线。

## 持续学习

你可以通过在配置中设置 `"continual_learning": true` 来采用持续学习方案。启用 `continual_learning` 后，在从头训练初始模型后，后续训练将从前一次训练的最终模型状态开始。这给了新模型一个关于先前状态的"记忆"。默认情况下，此参数设置为 `False`，意味着所有新模型都从头训练，不受先前模型的影响。

???+ danger "持续学习强制固定的参数空间"
    由于 `continual_learning` 意味着模型参数空间在训练之间*不能*改变，当 `continual_learning` 启用时，`principal_component_analysis` 将自动禁用。提示：PCA 会改变参数空间和特征数量，在此处了解更多关于 PCA 的信息。

???+ danger "实验性功能"
    请注意，这目前是一种朴素的增量学习方法，在市场从你的模型偏离时，有很大的概率过拟合/陷入局部最小值。我们主要在 FreqAI 中提供这些机制是出于实验目的，并为未来在加密货币市场这类混沌系统中的更成熟的持续学习方法做好准备。

## 超参优化

你可以使用与[典型 Freqtrade 超参优化](hyperopt.md)相同的命令进行超参优化：

```bash
freqtrade hyperopt --hyperopt-loss SharpeHyperOptLoss --strategy FreqaiExampleStrategy --freqaimodel LightGBMRegressor --strategy-path freqtrade/templates --config config_examples/config_freqai.example.json --timerange 20220428-20220507
```

`hyperopt` 要求你以与[回测](#backtesting)相同的方式预先下载数据。此外，在尝试超参优化 FreqAI 策略时，你必须考虑一些限制：

- `--analyze-per-epoch` 超参优化参数与 FreqAI 不兼容。
- 不可能超参优化 `feature_engineering_*()` 和 `set_freqai_targets()` 函数中的指标。这意味着你不能使用超参优化来优化模型参数。除此例外，可以优化所有其他[空间](hyperopt.md#running-hyperopt-with-smaller-search-space)。
- 回测说明同样适用于超参优化。

结合超参优化和 FreqAI 的最佳方法是专注于超参优化入场/出场阈值/条件。你需要专注于超参优化那些未在特征中使用的参数。例如，你不应该尝试超参优化特征创建中的滚动窗口长度，或 FreqAI 配置中改变预测的任何部分。为了高效地超参优化 FreqAI 策略，FreqAI 将预测存储为数据框并重用它们。因此要求仅超参优化入场/出场阈值/条件。

FreqAI 中可超参优化参数的一个好的例子是[差异指数 (DI)](freqai-feature-engineering.md#identifying-outliers-with-the-dissimilarity-index-di) `DI_values` 的阈值，超过该值我们将数据点视为异常值：

```python
di_max = IntParameter(low=1, high=20, default=10, space='buy', optimize=True, load=True)
dataframe['outlier'] = np.where(dataframe['DI_values'] > self.di_max.value/10, 1, 0)
```

这个特定的超参优化将帮助你理解适合你特定参数空间的 `DI_values`。

## 使用 Tensorboard

!!! note "可用性"
    FreqAI 为包括 XGBoost、所有 PyTorch 模型、强化学习和 Catboost 在内的多种模型提供 Tensorboard 支持。如果你希望看到 Tensorboard 集成到其他模型类型中，请在 [Freqtrade GitHub](https://github.com/freqtrade/freqtrade/issues) 上提交 issue。

!!! danger "要求"
    Tensorboard 日志记录需要 FreqAI torch 安装/Docker 镜像。

使用 Tensorboard 最简单的方法是确保配置文件中 `freqai.activate_tensorboard` 设置为 `True`（默认设置），运行 FreqAI，然后打开单独的 shell 并运行：

```bash
cd freqtrade
tensorboard --logdir user_data/models/unique-id
```

其中 `unique-id` 是 `freqai` 配置文件中设置的 `identifier`。如果你想在浏览器中查看 127.0.0.1:6060（6060 是 Tensorboard 使用的默认端口）的输出，此命令必须在单独的 shell 中运行。

![tensorboard](assets/tensorboard.jpg)

!!! note "禁用以提升性能"
    Tensorboard 日志记录可能会减慢训练速度，应该在生产使用中停用。