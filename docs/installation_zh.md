# 安装

本页介绍如何为运行机器人准备环境。

Freqtrade 文档描述了多种安装方式：

* [Docker 镜像](docker_quickstart.md)（独立页面）
* [脚本安装](#script-installation)
* [手动安装](#manual-installation)
* [Conda 安装](#installation-with-conda)

请考虑使用预构建的 [docker 镜像](docker_quickstart.md)来快速开始。

!!! Note "Windows 用户"
    我们**强烈**建议 Windows 用户使用 [Docker](docker_quickstart.md)，因为这样会更容易、更顺畅（也更安全）。如果不可行，尝试使用 Windows Linux 子系统（WSL）。如果确实想在 Windows 上原生安装，最好使用 `./setup.ps1` 安装脚本。

## 要求

* [Python >= 3.11](http://docs.python-guide.org/en/latest/starting/installation/)
* [pip](https://pip.pypa.io/en/stable/installing/)
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [virtualenv](https://virtualenv.pypa.io/en/stable/installation.html)（推荐）

### Debian/Ubuntu

```bash
sudo apt-get update
sudo apt install -y python3-pip python3-venv python3-dev python3-pandas git curl
```

### MacOS

```bash
brew install gettext libomp
```

## Freqtrade 仓库

```bash
# 下载 freqtrade 仓库的 `develop` 分支
git clone https://github.com/freqtrade/freqtrade.git
cd freqtrade

# 新手用户
git checkout stable

# 高级用户
git checkout develop
```

## 脚本安装

如果使用 Debian、Ubuntu 或 MacOS，freqtrade 提供安装脚本。

```bash
# --install，从头安装 freqtrade
./setup.sh -i
```

其他选项：

```bash
# --update，执行 git pull 更新
./setup.sh -u
# --reset，硬重置你的 develop/stable 分支
./setup.sh -r
```

### 激活虚拟环境

每次打开新终端时，必须运行以下命令激活虚拟环境：

```bash
source ./.venv/bin/activate
```

## 手动安装

```bash
# 在 /freqtrade/.venv 目录中创建 virtualenv
python3 -m venv .venv
# 激活 virtualenv
source .venv/bin/activate

# 安装依赖
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
python3 -m pip install -e .
```

## Conda 安装

```bash
conda create --name freqtrade python=3.12
conda activate freqtrade

python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
python3 -m pip install -e .
```

## 准备就绪

```bash
# 步骤 1 - 初始化用户文件夹
freqtrade create-userdir --userdir user_data

# 步骤 2 - 创建新配置文件
freqtrade new-config --config user_data/config.json
```

启动机器人：

```bash
freqtrade trade --config user_data/config.json --strategy SampleStrategy
```

!!! Warning
    你应该通读文档的其余部分，回测你打算使用的策略，并在启用真实资金交易之前使用模拟运行。

## 故障排除

### 常见问题："command not found"

如果你使用脚本或手动安装，需要在虚拟环境中运行机器人。确保 venv 已激活：

```bash
source ./.venv/bin/activate
```

### Windows 安装错误

```
error: Microsoft Visual C++ 14.0 is required.
```

你需要安装 Visual C++ build tools。可以从 [Visual Studio 网站](https://visualstudio.microsoft.com/visual-cpp-build-tools/)下载，或考虑使用 WSL2 或 [docker compose](docker_quickstart.md)。