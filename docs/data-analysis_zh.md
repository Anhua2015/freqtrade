# 使用 Jupyter notebook 分析机器人数据

你可以使用 Jupyter notebook 轻松分析回测和交易历史的结果。使用 `freqtrade create-userdir --userdir user_data` 初始化用户目录后，示例 notebook 位于 `user_data/notebooks/` 中。

## 使用 docker 快速开始

Freqtrade 提供了一个可启动 jupyter lab 服务器的 docker-compose 文件。
你可以使用以下命令运行此服务器：`docker compose -f docker/docker-compose-jupyter.yml up`

这将创建一个运行 jupyter lab 的 docker 容器，可通过 `https://127.0.0.1:8888/lab` 访问。
请使用启动后控制台中打印的链接来简化登录。

更多信息，请访问[使用 Docker 进行数据分析](docker_quickstart.md#data-analysis-using-docker-compose)部分。

### 小技巧

* 有关使用说明，请参见 [jupyter.org](https://jupyter.org/documentation)。
* 不要忘记从你的 conda 或 venv 环境内启动 Jupyter notebook 服务器，或使用 [nb_conda_kernels](https://github.com/Anaconda-Platform/nb_conda_kernels)*
* 使用前先复制示例 notebook，这样你的更改不会在下次 freqtrade 更新时被覆盖。

### 在系统级 Jupyter 安装中使用虚拟环境

有时你可能希望使用系统级的 Jupyter notebook 安装，同时使用虚拟环境中的 jupyter kernel。
这可以避免在系统上多次安装完整的 jupyter 套件，并提供了在任务之间（freqtrade / 其他分析任务）轻松切换的方式。

为此，首先激活你的虚拟环境并运行以下命令：

``` bash
# 激活虚拟环境
source .venv/bin/activate

pip install ipykernel
ipython kernel install --user --name=freqtrade
# 重启 jupyter（lab / notebook）
# 在 notebook 中选择 kernel "freqtrade"
```

!!! Note
    本节是为了信息完整性而提供，Freqtrade 团队不会为此设置的任何问题提供完整支持，并将建议直接在虚拟环境中安装 Jupyter，因为这是启动 jupyter notebook 最简单的方法。有关此设置的帮助，请参考 [Project Jupyter](https://jupyter.org/) [文档](https://jupyter.org/documentation)或[帮助渠道](https://jupyter.org/community)。

!!! Warning
    某些任务在 notebook 中效果并不好。例如，任何使用异步执行的操作对 Jupyter 来说都是问题。此外，freqtrade 的主要入口点是 shell CLI，因此在 notebook 中使用纯 Python 会绕过为辅助函数提供所需对象和参数的命令行参数。你可能需要手动设置这些值或创建预期的对象。

## 推荐工作流程

| 任务 | 工具 |
  --- | ---
机器人操作 | CLI
重复性任务 | Shell 脚本
数据分析与可视化 | Notebook

1. 使用 CLI 来

    * 下载历史数据
    * 运行回测
    * 以实时数据运行
    * 导出结果

1. 将这些操作收集到 shell 脚本中

    * 保存带有参数的复杂命令
    * 执行多步操作
    * 自动化测试策略和为分析准备数据

1. 使用 notebook 来

    * 可视化数据
    * 处理和绘图以生成洞察

## 实用代码片段示例

### 切换到项目根目录

Jupyter notebook 从 notebook 所在目录执行。以下代码片段搜索项目根目录，以便相对路径保持一致。

```python
import os
from pathlib import Path

# 更改目录
# 修改此单元格以确保输出显示正确的路径。
# 将所有路径定义为相对于单元格输出中显示的项目根目录
project_root = "somedir/freqtrade"
i=0
try:
    os.chdir(project_root)
    assert Path('LICENSE').is_file()
except:
    while i<4 and (not Path('LICENSE').is_file()):
        os.chdir(Path(Path.cwd(), '../'))
        i+=1
    project_root = Path.cwd()
print(Path.cwd())
```

### 加载多个配置文件

此选项对于检查传入多个配置的结果时很有用。
这也将运行整个 Configuration 初始化过程，因此配置会被完全初始化以传递给其他方法。

``` python
import json
from freqtrade.configuration import Configuration

# 从多个文件加载配置
config = Configuration.from_files(["config1.json", "config2.json"])

# 显示内存中的配置
print(json.dumps(config['original_config'], indent=2))
```

对于交互式环境，请额外指定一个包含 `user_data_dir` 的配置并最后传入，这样你在运行机器人时就不必切换目录。
最好避免使用相对路径，因为这会从 jupyter notebook 的存储位置开始，除非更改目录。

``` json
{
    "user_data_dir": "~/.freqtrade/"
}
```

### 更多数据分析文档

* [策略调试](strategy_analysis_example.md) - 也以 Jupyter notebook 形式提供（`user_data/notebooks/strategy_analysis_example.ipynb`）
* [绘图](plotting.md)
* [标签分析](advanced-backtesting.md)

如果你有关于如何最佳分析数据的想法，欢迎提交 issue 或 Pull Request 来改进本文档。