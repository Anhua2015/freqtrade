# FreqUI

Freqtrade 提供内置的 Web 服务器，可以托管 [FreqUI](https://github.com/freqtrade/frequi)，即 freqtrade 的前端界面。

默认情况下，UI 会作为安装（脚本、Docker）的一部分自动安装。
freqUI 也可以使用 `freqtrade install-ui` 命令手动安装。
此命令同样可用于将 freqUI 更新到新版本。

一旦机器人在交易 / 模拟模式下启动（使用 `freqtrade trade`），UI 将在配置的 API 端口上可用（默认为 `http://127.0.0.1:8080`）。

??? Note "想要为 freqUI 做贡献？"
    开发者不应使用此方法，而应克隆相应的仓库并使用 [freqUI 仓库](https://github.com/freqtrade/frequi) 中描述的方法获取 freqUI 的源代码。构建前端需要安装 node 的工作环境。

!!! tip "freqUI 不是运行 freqtrade 的必需组件"
    freqUI 是 freqtrade 的可选组件，不是运行机器人所必需的。
    它是一个可用于监控机器人和与之交互的前端——但 freqtrade 本身在没有它的情况下也能完美运行。

## 配置

FreqUI 没有自己的配置文件——但假设 [rest-api](rest-api.md) 的工作设置已就绪。
请参考相应的文档页面来设置 freqUI。

## 用户界面

FreqUI 是一个现代化、响应式的 Web 应用程序，可用于监控和与你的机器人交互。

FreqUI 提供浅色和深色两种主题。
可以通过页面顶部显眼的按钮轻松切换主题。
本页截图的主题将适应当前文档主题，因此要查看深色（或浅色）版本，请切换文档的主题。

### 登录

以下截图显示 freqUI 的登录界面。

![FreqUI - login](assets/frequi-login-CORS.png#only-dark)
![FreqUI - login](assets/frequi-login-CORS-light.png#only-light)

!!! Hint "CORS"
    此截图中显示的 CORS 错误是由于 UI 运行在与 API 不同的端口上，而 [CORS](#cors) 尚未正确设置。

### 交易视图

交易视图允许你可视化机器人正在进行的交易并与机器人交互。
在此页面上，你还可以通过启动和停止机器人与之交互——如果已配置——还可以强制入场和出场。

![FreqUI - trade view](assets/freqUI-trade-pane-dark.png#only-dark)
![FreqUI - trade view](assets/freqUI-trade-pane-light.png#only-light)

### 仪表盘

仪表盘视图提供机器人表现和状态的概览。
如果连接了多个机器人，仪表盘将显示所有已连接机器人的概览，允许你轻松切换它们或仅显示可用机器人的子集。

#### 钱包余额

Freqtrade 2026.4 新增功能：显示机器人的余额随时间变化的情况。

与"累计利润"图表相比，此图表将显示机器人随时间变化的实际余额，包括未实现盈亏以及存取款。

历史数据已基于可用的交易所数据重新填充——但假定为尽力而为，可能并非 100% 准确。
更具体地说，它不会涵盖存取款，并假定起始余额为当前余额减去盈亏。

为清晰起见，图表上显示了一条"Capture start"标记线，表示迁移到新钱包余额跟踪系统的时间点。
只有在此时间点之后，钱包余额才是准确的。

### 图表配置器

FreqUI 图表可以通过策略中的 `plot_config` 配置对象（可通过 "from strategy" 按钮加载）或通过 UI 进行配置。
可以创建多个图表配置并随意切换——从而为你的图表提供灵活、不同的视图。

图表配置可以通过交易视图右上角的"Plot Configurator"（齿轮图标）按钮访问。

![FreqUI - plot configuration](assets/freqUI-plot-configurator-dark.png#only-dark)
![FreqUI - plot configuration](assets/freqUI-plot-configurator-light.png#only-light)

### 设置

可以通过访问设置页面更改与 UI 相关的多个设置。

你可以更改的内容（除其他外）：

* UI 的时区
* 将未平仓交易可视化为网站图标（浏览器标签页）的一部分
* K 线颜色（上涨/下跌 → 红色/绿色）
* 启用 / 禁用应用内通知类型

![FreqUI - Settings view](assets/frequi-settings-dark.png#only-dark)
![FreqUI - Settings view](assets/frequi-settings-light.png#only-light)

## Web 服务器模式

当 freqtrade 以 [Web 服务器模式](utils.md#webserver-mode) 启动时（freqtrade 使用 `freqtrade webserver` 启动），Web 服务器将以特殊模式启动，允许使用额外的功能，例如：

* 下载数据
* 测试交易对列表
* [回测策略](#backtesting)
* ... 更多功能即将推出

### 回测

当 freqtrade 以 [Web 服务器模式](utils.md#webserver-mode) 启动时（freqtrade 使用 `freqtrade webserver` 启动），回测视图将变为可用。
此视图允许你回测策略并可视化结果。

你还可以加载和可视化之前的回测结果，并相互比较结果。

![FreqUI - Backtesting](assets/freqUI-backtesting-dark.png#only-dark)
![FreqUI - Backtesting](assets/freqUI-backtesting-light.png#only-light)

--8<-- "includes/cors.md"