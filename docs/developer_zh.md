# 开发帮助

本页面面向 Freqtrade 的开发者、希望为 Freqtrade 代码库或文档做出贡献的人，或想要理解其运行的应用程序源代码的人。

欢迎所有贡献、错误报告、错误修复、文档改进、增强功能和想法。我们在 [GitHub](https://github.com) 上[跟踪 issues](https://github.com/freqtrade/freqtrade/issues)，并在 [discord](https://discord.gg/p7nuUNVfP7) 上设有开发频道供提问。

## 文档

文档位于 [https://freqtrade.io](https://www.freqtrade.io/)，每个新功能 PR 都需要附带文档。

本地测试文档：

``` bash
pip install -r docs/requirements-docs.txt
mkdocs serve
```

这将在本地启动一个服务器（通常在端口 8000）以便查看效果。

## 开发者环境设置

要配置开发环境，可以使用提供的 [DevContainer](#devcontainer-setup)，或使用 `setup.sh` 脚本并在询问 "Do you want to install dependencies for dev [y/N]?" 时回答 "y"。
或者，手动安装并运行 `pip3 install -r requirements-dev.txt`，然后运行 `pip3 install -e .[all]`。

安装 git hook 脚本：

``` bash
pre-commit install
```

这些 pre-commit 脚本会在每次提交前自动检查你的更改。如果发现格式化问题，提交将失败并提示修复。

### Devcontainer 设置

最快最简单的入门方式是使用带 Remote container 扩展的 [VSCode](https://code.visualstudio.com/)。

#### Devcontainer 依赖

* [VSCode](https://code.visualstudio.com/)
* [docker](https://docs.docker.com/install/)
* [Remote container 扩展](https://code.visualstudio.com/docs/remote)

### 测试

新代码应有基本的单元测试覆盖。使用根目录下的 `pytest` 运行所有可用测试用例。

#### 检查测试中的日志内容

Freqtrade 使用 2 个主要方法来检查测试中的日志内容：`log_has()` 和 `log_has_re()`（用于正则表达式检查）。

``` python
from tests.conftest import log_has, log_has_re

def test_method_to_test(caplog):
    method_to_test()
    assert log_has("This event happened", caplog)
    assert log_has_re(r"This dynamic event happened and produced \d+", caplog)
```

### 调试配置

推荐使用 VSCode（带 Python 扩展）调试 freqtrade，配置文件位于 `.vscode/launch.json`：

``` json
{
    "name": "freqtrade trade",
    "type": "debugpy",
    "request": "launch",
    "module": "freqtrade",
    "console": "integratedTerminal",
    "args": [
        "trade",
        "--strategy", "MyAwesomeStrategy",
    ]
},
```

## 错误处理

Freqtrade 异常均继承自 `FreqtradeException`。不应直接使用此通用错误类，而应使用多个专门的子异常：

```
+ FreqtradeException
|
+---+ OperationalException
|   +---+ ConfigurationError
|
+---+ DependencyException
|   +---+ PricingError
|   +---+ ExchangeError
|       +---+ TemporaryError
|       +---+ DDosProtection
|       +---+ InvalidOrderException
|           +---+ RetryableOrderError
|           +---+ InsufficientFundsError
|
+---+ StrategyError
```

## 插件

### 交易对列表

要开发新的 Pairlist Handler，最好先查看 [VolumePairList](https://github.com/freqtrade/freqtrade/blob/develop/freqtrade/plugins/pairlist/VolumePairList.py) Handler 并复制该文件。

修改 Handler 的类名（最好与模块文件名一致）。

基类提供了交易所实例 (`self._exchange`)、交易对列表管理器 (`self._pairlistmanager`)、主配置 (`self._config`)、交易对列表专用配置 (`self._pairlistconfig`) 以及交易对列表中的绝对位置。

!!! Tip
    不要忘记在 `constants.py` 的 `AVAILABLE_PAIRLISTS` 变量中注册你的交易对列表——否则它将不可选。

#### 交易对列表配置

交易对列表 Handler 链的配置在机器人配置文件的 `"pairlists"` 元素中完成。
按照惯例，使用 `"number_assets"` 指定交易对列表中保留的最大交易对数量。