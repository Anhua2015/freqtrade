# 参数表

下表列出了 FreqAI 可用的所有配置参数。部分参数在 `config_examples/config_freqai.example.json` 中有示例。

必选参数标记为 **Required**，必须通过建议的方式之一进行设置。

### 通用配置参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`config.freqai` 树中的通用配置参数** |
| `freqai` | **必选。** <br> 包含控制 FreqAI 的所有参数的父字典。<br> **数据类型：** 字典。 |
| `train_period_days` | **必选。** <br> 用于训练数据的天数（滑动窗口宽度）。<br> **数据类型：** 正整数。 |
| `backtest_period_days` | **必选。** <br> 在滑动上述定义的 `train_period_days` 窗口并重新训练模型之前，从训练好的模型进行推理的天数（回测期间，更多信息见[此处](freqai-running.md#backtesting)）。可以为小数天数，但请注意提供的 `timerange` 将除以该数以得出完成回测所需的训练次数。<br> **数据类型：** 浮点数。 |
| `identifier` | **必选。** <br> 当前模型的唯一 ID。如果模型保存到磁盘，`identifier` 允许重新加载特定的预训练模型/数据。<br> **数据类型：** 字符串。 |
| `live_retrain_hours` | 模拟/实盘运行期间重新训练的频率。<br> **数据类型：** 大于 0 的浮点数。<br> 默认值：`0`（模型尽可能频繁地重新训练）。 |
| `expiration_hours` | 如果模型存在时间超过 `expiration_hours` 小时，则避免进行预测。<br> **数据类型：** 正整数。<br> 默认值：`0`（模型永不过期）。 |
| `purge_old_models` | 要保留在磁盘上的模型数量（与回测无关）。默认为 2，意味着模拟/实盘运行将在磁盘上保留最新的 2 个模型。设置为 0 则保留所有模型。此参数也接受布尔值以保持向后兼容。<br> **数据类型：** 整数。<br> 默认值：`2`。 |
| `save_backtest_models` | 在运行回测时将模型保存到磁盘。回测通过保存预测数据并在后续运行中直接重用以最高效地运行（当你想调整入场/出场参数时）。将回测模型保存到磁盘也允许使用相同的模型文件启动具有相同模型 `identifier` 的模拟/实盘实例。<br> **数据类型：** 布尔值。<br> 默认值：`False`（不保存模型）。 |
| `fit_live_predictions_candles` | 用于从预测数据计算目标（标签）统计信息的历史 K 线数量，而不是从训练数据集中计算（更多信息见[此处](freqai-configuration.md#creating-a-dynamic-target-threshold)）。<br> **数据类型：** 正整数。 |
| `continual_learning` | 使用最近训练模型的最终状态作为新模型的起点，允许增量学习（更多信息见[此处](freqai-running.md#continual-learning)）。请注意，这目前是一种朴素的增量学习方法，在市场从你的模型偏离时，有很大的概率过拟合/陷入局部最小值。我们将其保留在此主要是出于实验目的，并为未来在加密货币市场这类混沌系统中的更成熟的持续学习方法做好准备。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |
| `write_metrics_to_disk` | 将训练耗时、推理耗时和 CPU 使用率收集到 JSON 文件中。<br> **数据类型：** 布尔值。<br> 默认值：`False` |
| `data_kitchen_thread_count` | <br> 指定你想用于数据处理（异常值方法、归一化等）的线程数。这不影响用于训练的线程数。如果用户不设置（默认），FreqAI 将使用最大线程数减 2（留下 1 个物理核心用于 Freqtrade 机器人和 FreqUI）。<br> **数据类型：** 正整数。 |
| `activate_tensorboard` | <br> 指示是否为启用了 tensorboard 的模块（目前为强化学习、XGBoost、Catboost 和 PyTorch）激活 tensorboard。Tensorboard 需要安装 Torch，这意味着你需要使用 torch/RL docker 镜像，或者对安装过程中是否要安装 Torch 的问题回答 "yes"。<br> **数据类型：** 布尔值。<br> 默认值：`True`。 |
| `wait_for_training_iteration_on_reload` | <br> 在使用 /reload 或 ctrl-c 时，等待当前训练迭代完成再完成优雅关闭。如果设置为 `False`，FreqAI 将中断当前训练迭代，允许更快地优雅关闭，但你会丢失当前的训练迭代。<br> **数据类型：** 布尔值。<br> 默认值：`True`。 |

### 特征参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.feature_parameters` 子字典中的特征参数** |
| `feature_parameters` | 包含用于构建特征集的参数的字典。详细信息和示例见[此处](freqai-feature-engineering.md)。<br> **数据类型：** 字典。 |
| `include_timeframes` | `feature_engineering_expand_*()` 中的所有指标将为其创建的时间周期列表。该列表作为特征添加到基础指标数据集中。<br> **数据类型：** 时间周期（字符串）列表。 |
| `include_corr_pairlist` | FreqAI 将作为附加特征添加到所有 `pair_whitelist` 交易对的关联币种列表。特征工程期间 `feature_engineering_expand_*()` 中设置的所有指标（详情见[此处](freqai-feature-engineering.md)）将为每个关联币种创建。关联币种的特征会添加到基础指标数据集中。<br> **数据类型：** 资产（字符串）列表。 |
| `label_period_candles` | 创建标签的未来 K 线数量。可用于 `set_freqai_targets()`（详细用法见 `templates/FreqaiExampleStrategy.py`）。此参数并非必需，你可以创建自定义标签并选择是否使用此参数。请参见 `templates/FreqaiExampleStrategy.py` 查看示例用法。<br> **数据类型：** 正整数。 |
| `include_shifted_candles` | 将之前 K 线的特征添加到后续 K 线，目的是增加历史信息。如果使用，FreqAI 将复制并偏移 `include_shifted_candles` 根之前 K 线的所有特征，使后续 K 线可以使用这些信息。<br> **数据类型：** 正整数。 |
| `weight_factor` | 根据数据点的近期性对训练数据点进行加权（详情见[此处](freqai-feature-engineering.md#weighting-features-for-temporal-importance)）。<br> **数据类型：** 正浮点数（通常小于 1）。 |
| `indicator_max_period_candles` | **不再使用（#7325）**。替换为在[策略](freqai-configuration.md#building-a-freqai-strategy)中设置的 `startup_candle_count`。`startup_candle_count` 与时间周期无关，定义了 `feature_engineering_*()` 中指标创建所使用的最大*周期*。FreqAI 使用此参数与 `include_time_frames` 中的最大时间周期来计算需要下载多少数据点，使第一个数据点不包含 NaN。<br> **数据类型：** 正整数。 |
| `indicator_periods_candles` | 计算指标的时间周期。指标会添加到基础指标数据集中。<br> **数据类型：** 正整数列表。 |
| `principal_component_analysis` | 使用主成分分析自动降低数据集的维度。详见其工作原理[此处](freqai-feature-engineering.md#data-dimensionality-reduction-with-principal-component-analysis)。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |
| `plot_feature_importances` | 为每个模型创建特征重要性图，展示前/后 `plot_feature_importances` 个特征。图表存储在 `user_data/models/<identifier>/sub-train-<COIN>_<timestamp>.html`。<br> **数据类型：** 整数。<br> 默认值：`0`。 |
| `DI_threshold` | 当设置为大于 0 的值时，激活使用差异指数进行异常值检测。详见其工作原理[此处](freqai-feature-engineering.md#identifying-outliers-with-the-dissimilarity-index-di)。<br> **数据类型：** 正浮点数（通常小于 1）。 |
| `use_SVM_to_remove_outliers` | 训练一个支持向量机来检测并移除训练数据集以及传入数据点中的异常值。详见其工作原理[此处](freqai-feature-engineering.md#identifying-outliers-using-a-support-vector-machine-svm)。<br> **数据类型：** 布尔值。 |
| `svm_params` | Sklearn 的 `SGDOneClassSVM()` 中可用的所有参数。部分选定参数的详情见[此处](freqai-feature-engineering.md#identifying-outliers-using-a-support-vector-machine-svm)。<br> **数据类型：** 字典。 |
| `use_DBSCAN_to_remove_outliers` | 使用 DBSCAN 算法对数据进行聚类，以识别并移除训练和预测数据中的异常值。详见其工作原理[此处](freqai-feature-engineering.md#identifying-outliers-with-dbscan)。<br> **数据类型：** 布尔值。 |
| `noise_standard_deviation` | 如果设置，FreqAI 会向训练特征添加噪声以防止过拟合。FreqAI 从标准差为 `noise_standard_deviation` 的高斯分布生成随机偏差并将其添加到所有数据点。`noise_standard_deviation` 应相对于归一化空间保持，即在 -1 和 1 之间。换句话说，由于 FreqAI 中的数据始终归一化到 -1 和 1 之间，`noise_standard_deviation: 0.05` 将导致 32% 的数据被随机增加/减少超过 2.5%（即落入第一个标准差内的数据百分比）。<br> **数据类型：** 整数。<br> 默认值：`0`。 |
| `outlier_protection_percentage` | 启用以防止异常值检测方法丢弃过多数据。如果超过 `outlier_protection_percentage` % 的点被 SVM 或 DBSCAN 检测为异常值，FreqAI 将记录一条警告消息并忽略异常值检测，即保留原始数据集不变。如果触发了异常值保护，将不基于此训练数据集进行任何预测。<br> **数据类型：** 浮点数。<br> 默认值：`30`。 |
| `reverse_train_test_order` | 拆分特征数据集（见下文），使用最新数据拆分用于训练，并在历史数据拆分上进行测试。这允许模型训练到最近的数据点，同时避免过拟合。然而，在使用此参数之前，你应注意理解其非常规性质。<br> **数据类型：** 布尔值。<br> 默认值：`False`（不反转）。 |
| `shuffle_after_split` | 将数据拆分为训练集和测试集，然后分别洗牌两个集合。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |
| `buffer_train_data_candles` | 在指标填充*之后*，从训练数据的开头和末尾各裁掉 `buffer_train_data_candles` 根 K 线。主要用例是预测最大值和最小值时，argrelextrema 函数无法知道时间范围边缘的最大值/最小值。为提升模型精度，最好在完整时间范围上计算 argrelextrema，然后使用此函数按内核裁掉边缘（缓冲区）。在另一种情况下，如果目标设置为偏移价格变动，此缓冲区则不需要，因为时间范围末尾的偏移 K 线将是 NaN，FreqAI 将自动将其从训练数据集中切除。<br> **数据类型：** 整数。<br> 默认值：`0`。 |

### 数据拆分参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.data_split_parameters` 子字典中的数据拆分参数** |
| `data_split_parameters` | 包含来自 scikit-learn `test_train_split()` 的任何附加参数，见[此处](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html)（外部网站）。<br> **数据类型：** 字典。 |
| `test_size` | 应用于测试而非训练的数据比例。<br> **数据类型：** 小于 1 的正浮点数。 |
| `shuffle` | 在训练期间洗牌训练数据点。通常，为了保留时间序列数据的时间顺序，此参数设置为 `False`。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |

### 模型训练参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.model_training_parameters` 子字典中的模型训练参数** |
| `model_training_parameters` | 一个灵活的字典，包含所选模型库可用的所有参数。例如，如果你使用 `LightGBMRegressor`，此字典可以包含 `LightGBMRegressor` 可用的任何参数，见[此处](https://lightgbm.readthedocs.io/en/latest/pythonapi/lightgbm.LGBMRegressor.html)（外部网站）。如果你选择不同的模型，此字典可以包含该模型的任何参数。当前可用模型列表见[此处](freqai-configuration.md#using-different-prediction-models)。<br> **数据类型：** 字典。 |
| `n_estimators` | 模型训练中要拟合的增强树数量。<br> **数据类型：** 整数。 |
| `learning_rate` | 模型训练期间的增强学习率。<br> **数据类型：** 浮点数。 |
| `n_jobs`, `thread_count`, `task_type` | 设置并行处理的线程数以及 `task_type`（`gpu` 或 `cpu`）。不同的模型库使用不同的参数名称。<br> **数据类型：** 浮点数。 |

### 强化学习参数

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.rl_config` 子字典中的强化学习参数** |
| `rl_config` | 包含强化学习模型控制参数的字典。<br> **数据类型：** 字典。 |
| `train_cycles` | 训练时间步长将基于 `train_cycles * 训练数据点数量` 设置。<br> **数据类型：** 整数。 |
| `max_trade_duration_candles`| 引导智能体训练将交易保持在期望长度以下。示例用法见 `prediction_models/ReinforcementLearner.py` 中可自定义的 `calculate_reward()` 函数。<br> **数据类型：** 整数。 |
| `model_type` | 来自 stable_baselines3 或 SBcontrib 的模型字符串。可用字符串包括：`'TRPO'、'ARS'、'RecurrentPPO'、'MaskablePPO'、'PPO'、'A2C'、'DQN'`。用户应通过访问 stable_baselines3 文档确保 `model_training_parameters` 与相应的模型匹配。[PPO 文档](https://stable-baselines3.readthedocs.io/en/master/modules/ppo.html)（外部网站）<br> **数据类型：** 字符串。 |
| `policy_type` | 来自 stable_baselines3 的可用策略类型之一。<br> **数据类型：** 字符串。 |
| `max_training_drawdown_pct` | 智能体在训练期间允许承受的最大回撤。<br> **数据类型：** 浮点数。<br> 默认值：0.8 |
| `cpu_count` | 专门用于强化学习训练过程的线程/CPU 数量（取决于是否选择了 `ReinforcementLearner_multiproc`）。建议保持此值不变，默认情况下，此值设置为物理核心总数减 1。<br> **数据类型：** 整数。 |
| `model_reward_parameters` | 在 `ReinforcementLearner.py` 中可自定义的 `calculate_reward()` 函数内使用的参数。<br> **数据类型：** 整数。 |
| `add_state_info` | 告诉 FreqAI 在训练和推理的特征集中包含状态信息。当前状态变量包括交易持续时间、当前利润、交易仓位。仅在模拟/实盘运行中可用，回测时自动切换为 false。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |
| `net_arch` | 网络架构，在 [`stable_baselines3` 文档](https://stable-baselines3.readthedocs.io/en/master/guide/custom_policy.html#examples)中有详细描述。简而言之：`[<共享层>, dict(vf=[<非共享价值网络层>], pi=[<非共享策略网络层>])]`。默认设置为 `[128, 128]`，定义 2 个各有 128 个单元的共享隐藏层。 |
| `randomize_starting_position` | 随机化每个回合的起点，以避免过拟合。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |
| `drop_ohlc_from_features` | 不要在训练期间传递给智能体的特征集中包含归一化的 OHLC 数据（OHLC 在所有情况下仍然会用于驱动环境）。<br> **数据类型：** 布尔值。<br> **默认值：** `False` |
| `progress_bar` | 显示一个进度条，包含当前进度、已用时间和预估剩余时间。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |

### PyTorch 参数

#### 通用

|  参数 | 描述 |
|------------|-------------|
|  |  **`freqai.model_training_parameters` 子字典中的模型训练参数** |
| `learning_rate` | 传递给优化器的学习率。<br> **数据类型：** 浮点数。<br> 默认值：`3e-4`。 |
| `model_kwargs` | 传递给模型类的参数。<br> **数据类型：** 字典。<br> 默认值：`{}`。 |
| `trainer_kwargs` | 传递给训练器类的参数。<br> **数据类型：** 字典。<br> 默认值：`{}`。 |

#### trainer_kwargs

| 参数    | 描述 |
|--------------|-------------|
|              |  **`freqai.model_training_parameters.model_kwargs` 子字典中的模型训练参数** |
| `n_epochs`   | `n_epochs` 参数是 PyTorch 训练循环中的一个关键设置，决定整个训练数据集被用于更新模型参数的次数。一个 epoch 表示完整遍历整个训练数据集一次。会覆盖 `n_steps`。必须设置 `n_epochs` 或 `n_steps` 之一。<br><br> **数据类型：** 整数，可选。<br> 默认值：`10`。 |
| `n_steps`    | 设置 `n_epochs` 的替代方式——要运行的训练迭代次数。此处的迭代指调用 `optimizer.step()` 的次数。如果设置了 `n_epochs` 则忽略。该函数的简化版：<br><br> n_epochs = n_steps / (n_obs / batch_size)<br><br> 这样做的动机是 `n_steps` 更容易优化，并在不同的 n_obs（数据点数量）下保持稳定。<br> <br> **数据类型：** 整数，可选。<br> 默认值：`None`。 |
| `batch_size` | 训练期间使用的批次大小。<br><br> **数据类型：** 整数。<br> 默认值：`64`。 |
| `early_stopping_patience` | 在验证损失不再改善后，停止训练前的等待轮数。这通过当模型停止改进时提前停止训练来防止过拟合。设置为 `0` 则禁用提前停止。需要有测试/验证集拆分（`test_size > 0`）。<br><br> **数据类型：** 整数。<br> 默认值：`0`（禁用）。 |

### 附加参数

|  参数 | 描述 |
|------------|-------------|
|  |  **额外参数** |
| `freqai.keras` | 如果所选模型使用了 Keras（通常用于基于 TensorFlow 的预测模型），需要激活此标志，以便模型保存/加载遵循 Keras 标准。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |
| `freqai.conv_width` | 神经网络输入张量的宽度。这替代了对偏移 K 线的需求（`include_shifted_candles`），通过将历史数据点作为张量的第二维度输入。技术上，此参数也可用于回归器，但只会增加计算开销而不改变模型训练/预测。<br> **数据类型：** 整数。<br> 默认值：`2`。 |
| `freqai.reduce_df_footprint` | 将所有数值列重新转换类型为 float32/int32，目的是减少内存/磁盘使用并缩短训练/推理时间。此参数设置在 Freqtrade 配置文件的主层级中（不在 FreqAI 内部）。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |
| `freqai.override_exchange_check` | 覆盖交易所检查，强制 FreqAI 使用可能没有足够历史数据的交易所。如果你知道你的 FreqAI 模型和策略不需要历史数据，请将其设置为 True。<br> **数据类型：** 布尔值。<br> 默认值：`False`。 |