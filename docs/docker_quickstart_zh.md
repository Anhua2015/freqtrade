# 使用 Docker 运行 Freqtrade

本页介绍如何使用 Docker 运行机器人。它并非开箱即用。你仍然需要通读文档并理解如何正确配置它。

## 安装 Docker

首先下载并安装适用于你平台的 Docker / Docker Desktop：

* [Mac](https://docs.docker.com/docker-for-mac/install/)
* [Windows](https://docs.docker.com/docker-for-windows/install/)
* [Linux](https://docs.docker.com/install/)

!!! Info "Docker compose 安装"
    Freqtrade 文档假设使用 Docker desktop（或 docker compose 插件）。
    虽然独立的 docker-compose 安装方式仍然可用，但需要将所有 `docker compose` 命令从 `docker compose` 改为 `docker-compose` 才能工作（例如 `docker compose up -d` 将变为 `docker-compose up -d`）。

??? Warning "Windows 上的 Docker"
    如果你刚刚在 Windows 系统上安装了 docker，请确保重启系统，否则可能会遇到与 docker 容器网络连接相关的无法解释的问题。

## 使用 Docker 运行 Freqtrade

Freqtrade 在 [Dockerhub](https://hub.docker.com/r/freqtradeorg/freqtrade/) 上提供官方 Docker 镜像，以及可以直接使用的 [docker compose 文件](https://github.com/freqtrade/freqtrade/blob/stable/docker-compose.yml)。

!!! Note
    - 以下部分假设 `docker` 已安装且对登录用户可用。
    - 以下所有命令使用相对目录，需要从包含 `docker-compose.yml` 文件的目录中执行。

### Docker 快速开始

创建一个新目录并将 [docker-compose 文件](https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml)放入此目录。

``` bash
mkdir ft_userdata
cd ft_userdata/
# 从仓库下载 docker-compose 文件
curl https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml -o docker-compose.yml

# 拉取 freqtrade 镜像
docker compose pull

# 创建用户目录结构
docker compose run --rm freqtrade create-userdir --userdir user_data

# 创建配置——需要回答交互式问题
docker compose run --rm freqtrade new-config --config user_data/config.json
```

以上代码片段创建一个名为 `ft_userdata` 的新目录，下载最新的 compose 文件并拉取 freqtrade 镜像。
代码片段的最后两步创建 `user_data` 目录，以及根据你的选择（交互式地）创建默认配置。

!!! Question "如何编辑机器人配置？"
    你可以随时编辑配置，使用上述配置时配置文件为 `user_data/config.json`（在 `ft_userdata` 目录内）。

    你也可以通过编辑 `docker-compose.yml` 文件的 command 部分来更改策略和命令。

#### 添加自定义策略

1. 配置现在位于 `user_data/config.json`
2. 将自定义策略复制到目录 `user_data/strategies/`
3. 将策略类名添加到 `docker-compose.yml` 文件中

默认运行的是 `SampleStrategy`。

!!! Danger "`SampleStrategy` 只是一个演示！"
    `SampleStrategy` 仅供参考，为你的策略提供思路。
    在冒险投入真实资金之前，请始终回测你的策略并先使用模拟运行一段时间！
    你可以在[策略文档](strategy-customization.md)中找到更多关于策略开发的信息。

完成后，你就可以在交易模式下启动机器人（模拟运行或实盘交易，取决于你对上述相应问题的回答）。

``` bash
docker compose up -d
```

!!! Warning "默认配置"
    虽然生成的配置大部分情况下可以工作，但在启动机器人之前，你仍然需要验证所有选项是否符合你的需求（如定价、交易对列表等）。

#### 访问 UI

如果你在 `new-config` 步骤中选择了启用 FreqUI，你将在端口 `localhost:8080` 上拥有 freqUI。

你现在可以通过在浏览器中输入 localhost:8080 来访问 UI。

??? Note "远程服务器上的 UI 访问"
    如果你在 VPS 上运行，你应该考虑使用 ssh 隧道，或设置 VPN（openVPN、wireguard）来连接到你的机器人。
    这将确保 freqUI 不直接暴露在互联网上，出于安全原因不推荐这样做（freqUI 本身不支持 https）。
    这些工具的设置不属于本教程的范围，但互联网上可以找到很多好的教程。
    另请阅读 [Docker 的 API 配置](rest-api.md#configuration-with-docker)部分以了解更多关于此配置的信息。

#### 监控机器人

你可以使用 `docker compose ps` 检查正在运行的实例。
这应该将服务 `freqtrade` 列为 `running`。如果不是这种情况，最好查看日志（见下一条）。

#### Docker compose 日志

日志将写入：`user_data/logs/freqtrade.log`。
你也可以使用命令 `docker compose logs -f` 查看最新日志。

#### 数据库

数据库位于：`user_data/tradesv3.sqlite`

#### 使用 Docker 更新 freqtrade

使用 `docker` 时更新 freqtrade 非常简单，只需运行以下两个命令：

``` bash
# 下载最新镜像
docker compose pull
# 重启镜像
docker compose up -d
```

这将首先拉取最新镜像，然后使用刚刚拉取的版本重启容器。

!!! Warning "检查更新日志"
    你应该始终检查更新日志了解破坏性变更/需要手动干预的内容，并确保机器人在更新后正常启动。

### 编辑 docker-compose 文件

高级用户可以进一步编辑 docker-compose 文件以包含所有可能的选项或参数。

所有 freqtrade 参数都可以通过运行 `docker compose run --rm freqtrade <command> <optional arguments>` 来使用。

!!! Warning "交易命令的 `docker compose`"
    交易命令（`freqtrade trade <...>`）不应通过 `docker compose run` 运行——而应使用 `docker compose up -d`。
    这确保了容器被正确启动（包括端口转发），并确保容器在系统重启后会重启。
    如果你打算使用 freqUI，请确保也相应地[调整配置](rest-api.md#configuration-with-docker)，否则 UI 将不可用。

!!! Note "`docker compose run --rm`"
    包含 `--rm` 将在完成后移除容器，强烈推荐在除交易模式外的所有模式中使用（使用 `freqtrade trade` 命令运行时）。

??? Note "不使用 docker compose 使用 docker"
    "`docker compose run --rm`" 需要提供 compose 文件。
    某些不需要认证的 freqtrade 命令（如 `list-pairs`）可以改用 "`docker run --rm`" 运行。
    例如 `docker run --rm freqtradeorg/freqtrade:stable list-pairs --exchange binance --quote BTC --print-json`。
    这对于获取交易所信息以添加到 `config.json` 而不影响正在运行的容器很有用。

#### 示例：使用 Docker 下载数据

从 Binance 下载 ETH/BTC 交易对 5 天、1h 时间周期的回测数据。数据将存储在主机上的 `user_data/data/` 目录中。

``` bash
docker compose run --rm freqtrade download-data --pairs ETH/BTC --exchange binance --days 5 -t 1h
```

前往[数据下载文档](data-download.md)了解更多关于下载数据的详细信息。

#### 示例：使用 Docker 回测

在 docker 容器中为 SampleStrategy 和指定的历史数据时间范围、5m 时间周期运行回测：

``` bash
docker compose run --rm freqtrade backtesting --config user_data/config.json --strategy SampleStrategy --timerange 20190801-20191001 -i 5m
```

前往[回测文档](backtesting.md)了解更多。

### Docker 的额外依赖项

如果你的策略需要默认镜像中未包含的依赖项——将需要在你的主机上构建镜像。
为此，请创建一个包含额外依赖项安装步骤的 Dockerfile（参见 [docker/Dockerfile.custom](https://github.com/freqtrade/freqtrade/blob/develop/docker/Dockerfile.custom) 的示例）。

然后你还需要修改 `docker-compose.yml` 文件并取消注释 build 步骤，并重命名镜像以避免命名冲突。

``` yaml
    image: freqtrade_custom
    build:
      context: .
      dockerfile: "./Dockerfile.<yourextension>"
```

然后你可以运行 `docker compose build --pull` 来构建 docker 镜像，并使用上述命令运行它。

### 使用 Docker 绘图

命令 `freqtrade plot-profit` 和 `freqtrade plot-dataframe`（[文档](plotting.md)）可通过在你的 `docker-compose.yml` 文件中将镜像更改为 `*_plot` 来使用。
然后你可以按如下方式使用这些命令：

``` bash
docker compose run --rm freqtrade plot-dataframe --strategy AwesomeStrategy -p BTC/ETH --timerange=20180801-20180805
```

输出将存储在 `user_data/plot` 目录中，可以用任何现代浏览器打开。

### 使用 docker compose 进行数据分析

Freqtrade 提供了一个可启动 jupyter lab 服务器的 docker-compose 文件。
你可以使用以下命令运行此服务器：

``` bash
docker compose -f docker/docker-compose-jupyter.yml up
```

这将创建一个运行 jupyter lab 的 docker 容器，可通过 `https://127.0.0.1:8888/lab` 访问。
请使用启动后控制台中打印的链接来简化登录。

由于此镜像的一部分是在你的机器上构建的，建议不时重建镜像以保持 freqtrade（和依赖项）为最新。

``` bash
docker compose -f docker/docker-compose-jupyter.yml build --no-cache
```

## 故障排除

### Windows 上的 Docker

* 错误：`"Timestamp for this request is outside of the recvWindow."`
  市场 API 请求需要同步的时钟，但 docker 容器中的时间会随着时间的推移略微向过去偏移。
  要临时修复此问题，你需要运行 `wsl --shutdown` 并重新启动 docker（Windows 10 上会弹出窗口要求你这样做）。
  永久解决方案要么将 docker 容器托管在 linux 主机上，要么使用调度器不时重启 wsl。

  ``` bash
  taskkill /IM "Docker Desktop.exe" /F
  wsl --shutdown
  start "" "C:\Program Files\Docker\Docker\Docker Desktop.exe"
  ```

* 无法连接到 API（Windows）
  如果你在 Windows 上且刚刚安装了 Docker（desktop），请确保重启系统。Docker 在未重启的情况下可能会有网络连接问题。
  你显然也应该确保你的[设置](#accessing-the-ui)相应地配置。

!!! Warning
    由于上述原因，我们不推荐在生产环境中使用 Windows 上的 docker，仅用于实验、数据下载和回测。
    最好使用 linux-VPS 来可靠地运行 freqtrade。