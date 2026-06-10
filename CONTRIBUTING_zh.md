# 贡献指南

## 为 freqtrade 做贡献

觉得我们的机器人缺少某个功能？我们欢迎你的 Pull Request！

标记为 [good first issue](https://github.com/freqtrade/freqtrade/labels/good%20first%20issue) 的问题适合作为第一次贡献，有助于你熟悉代码库。

几点贡献提示：

- 针对 `develop` 分支创建 PR，而非 `stable`。
- 提交信息、PR 描述、代码注释和变量名请使用英文。
- 新功能必须包含单元测试，必须通过 CI（运行 pre-commit 和 pytest 以获得早期反馈），并且应在提交 PR 的同时提供文档。
- PR 可以声明为草稿（draft）— 表示正在进行中的 Pull Request（尚未完成）。我们仍会尽量及时对草稿 PR 提供反馈。
- 如果你使用 AI 辅助你的 PR，请在 PR 描述中注明，并且自己仔细审查生成的代码。  
  代码的最终责任在 PR 作者，而非 AI，这也意味着提交必须关联到你（人类）的账户，而非某个通用 AI 账户。

如果你不确定某个功能是否合适，请在发起 Pull Request 之前在 [Discord 服务器](https://discord.gg/p7nuUNVfP7) 或 [issue](https://github.com/freqtrade/freqtrade/issues) 中进行讨论。

## 入门

最好从阅读[文档](https://www.freqtrade.io/)开始，了解机器人的能力范围，或者直接查阅[开发者文档](https://www.freqtrade.io/en/latest/developer/)（开发中），这应该能帮助你入门。

## 提交 PR 前的准备

### 1. 运行单元测试

所有单元测试必须通过。如果某个单元测试失败，请修改你的代码使其通过。这意味着你引入了回归问题。

#### 测试整个项目

```bash
pytest
```

#### 只测试一个文件

```bash
pytest tests/test_<file_name>.py
```

#### 只测试一个文件中的一个方法

```bash
pytest tests/test_<file_name>.py::test_<method_name>
```

### 2. 检查你的代码是否符合风格指南

我们收到的大量代码未通过初步 CI 检查。  
为此，我们鼓励贡献者安装 git pre-commit 钩子，这样在你尝试提交未通过检查的代码时会立即得到提示。

你可以手动运行 pre-commit：`pre-commit run -a`，或者通过 `pre-commit install` 安装 git 钩子，使其在每次提交时自动运行。

运行 `pre-commit run -a` 将执行所有检查，包括 `ruff`、`mypy` 和 `codespell` 等。

#### 额外应用的风格规范

- 所有公开方法须有 docstring
- docstring 使用双引号
- 多行 docstring 应缩进到第一个引号的层级
- docstring 应遵循 reST 格式（`:param xxx: ...`、`:return: ...`、`:raises KeyError: ...`）

#### 手动运行单项检查

以下部分描述了如何手动运行 pre-commit 钩子中的各项检查。

##### 运行 ruff

用 ruff 检查你的代码，确保遵循风格指南。

```bash
ruff check .
ruff format .
```

##### 运行 mypy

用 mypy 检查你的代码，确保遵循类型标注规则。

```bash
mypy freqtrade
```

## （核心）提交者指南

### 流程：Pull Request

Pull Request 的优先级排序，从高到低：

1. 修复失败的测试。失败指在任何支持的平台或 Python 版本上失败。
2. 覆盖边界情况的额外测试。
3. 文档的小幅修改。
4. Bug 修复。
5. 文档的大幅修改。
6. 新功能。

确保每个 Pull Request 满足贡献指南中的所有要求。

### 流程：Issue

如果某个 issue 是需要紧急修复的 bug，将其标记为下一个补丁版本。然后要么修复它，要么标记为 "please-help"。

对于其他 issue：鼓励友好讨论，节制辩论，提供你的想法。

### 流程：你自己的代码变更

所有代码变更，无论何人提交，都需要经他人审查并合并。此规则适用于所有核心提交者。

例外情况：

- 对他人提交的 Pull Request 进行小幅修正和修复。
- 在发布正式版本时，发布经理可以做出必要、适当的更改。
- 对于强化现有内容的文档小幅更改。最常见的是但不限于拼写和语法修正。

### 职责

- 确保每一个被接受的变更都具备跨平台兼容性：Windows、Mac 和 Linux。
- 确保核心代码中不引入恶意代码。
- 为你打算进行的任何重大更改和增强创建 issue。透明地讨论问题并获取社区反馈。
- 尽可能保持功能 PR 小巧，最好每个 PR 只包含一个新功能。
- 欢迎新人，鼓励来自各种背景的多元化新贡献者。参见 Python 社区行为准则（https://www.python.org/psf/codeofconduct/）。

### 成为提交者

贡献者可被授予提交权限。优先考虑以下条件的人选：

1. 过去对 Freqtrade 和其他相关开源项目的贡献。对 Freqtrade 的贡献包括代码（已接受和待处理的）以及在 issue 追踪器和 Pull Request 审查中的友好参与。数量和质量都会被考虑。
1. 代码风格被其他核心提交者认为简单、精简、整洁。
1. 拥有跨平台开发和测试的资源。
1. 有定期投入项目的时间。

出于安全原因（用户将自己的交易所 API 密钥托付给 Freqtrade），成为提交者并不会自动授予对 `develop` 或 `stable` 分支的写入权限。

担任提交者一段时间后，提交者可能被任命为核心提交者，并获得完整的仓库访问权限。