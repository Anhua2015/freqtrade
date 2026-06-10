# 如何更新

要更新你的 freqtrade 安装，请根据你的安装方式使用以下方法之一。

!!! Note "跟踪变更"
    破坏性变更 / 行为变更将记录在每次发布时附带的更新日志中。
    对于 develop 分支，请关注 PR 以免被变更措手不及。

## 为什么要更新？

保持机器人更新不仅能确保你拥有最新功能和改进，还是让机器人平稳运行的必要条件。
Freqtrade 高度依赖底层交易所 API，而跨交易所来看，这些 API 变化相当频繁。
为确保持续的兼容性，请务必定期更新你的机器人。

## Docker

!!! Note "使用 `master` 镜像的传统安装"
    我们正在将发布镜像从 master 切换到 stable——请调整你的 Docker 文件，将 `freqtradeorg/freqtrade:master` 替换为 `freqtradeorg/freqtrade:stable`

``` bash
docker compose pull
docker compose up -d
```

## 通过 setup 脚本安装

``` bash
./setup.sh --update
```

!!! Note
    确保在禁用虚拟环境的情况下运行此命令！

## 裸机原生安装

请确保你也在更新依赖项——否则可能在不知不觉中出现问题。

``` bash
git pull
pip install -U -r requirements.txt
pip install -e .

# 确保 freqUI 是最新版本
freqtrade install-ui 
```

## 更新问题

更新问题通常来自缺少依赖项（你没有按照上述说明操作）——或来自无法安装的依赖项。
我们尽量确保重量级依赖项在主流平台上都有可用的 wheel，但有时这无法实现。

请参考相应的安装部分（常见问题部分链接如下）。

[常见安装问题](installation.md#troubleshooting)
[常见安装问题 - Windows](installation.md#windows-installation-error)