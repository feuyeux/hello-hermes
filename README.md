# Hello Hermes Agent ☤

这是一个用于探索和分析 [Hermes Agent](https://github.com/nousresearch/hermes-agent) `v0.8.0 (v2026.4.8)` `86960cdb` 的工作区。

> ## 正确发音
>
> 注意：本项目中的 “Hermes” 指的是希腊神话中的神。
>
> ✔️ **Hermes**: `/ˈhɜːrmiːz/` — 希腊神话中的语言、文字之神，众神的使者（赫尔墨斯）。
>
> ✖️ **Hermès**: `/ɛʁ.mɛs/` — 法国奢侈品牌（爱马仕）。

## 源代码分析

```sh
git clone --depth 1 --branch v2026.4.8 https://github.com/nousresearch/hermes-agent
```

- [Hermes 架构解析 (一)：流程篇 · 源代码执行全生命周期](./Hermes%20架构解析%20(一)：流程篇%20·%20源代码执行全生命周期.md)
- [Hermes 架构解析 (二)：数据篇 · 状态模型与上下文治理](./Hermes%20架构解析%20(二)：数据篇%20·%20状态模型与上下文治理.md)
- [Hermes 架构解析 (三)：扩展篇 · 插件与技能开发全指南](./Hermes%20架构解析%20(三)：扩展篇%20·%20插件与技能开发全指南.md)

## 快速开始

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
# Windows
powershell -Command "Set-ExecutionPolicy Bypass -Scope Process -Force; irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex"
```

```bash
# Run the setup wizard
hermes setup

# View/edit configuration
code ~/.hermes/

# Start interactive chat
hermes
```

## PyCharm 断点调试

在 `hermes-agent/` 目录下使用 PyCharm 进行断点调试：

### 1. 配置项目

```bash
cd hermes-agent
uv venv venv --python 3.11
source venv/bin/activate   # Windows: venv\Scripts\activate
uv pip install -e ".[all,dev]"
```

### 2. 配置 PyCharm

1. **Python 解释器**：File → Project Structure → Add Python Interpreter，选择 `hermes-agent/venv/bin/python`（或 Windows 上的 `venv\Scripts\python.exe`）
2. **SDK 根目录**：Project → Project Structure，将 `hermes-agent/` 标记为 Sources
3. **工作目录**：Project Structure → Modules → Paths → Add Content Root，添加 `hermes-agent/src`（如果存在）

### 3. 配置 Run Configuration

| 字段 | 值 |
|------|-----|
| **Script path** | `hermes-agent/src/hermes_cli/main.py` |
| **Working directory** | `hermes-agent/` |
| **Environment variables** | `PYTHONPATH=hermes-agent/src`（或对应的 `src` 目录路径） |

> **入口点说明**：`hermes_cli/main.py` 的 `main()` 是统一的 CLI 入口。也可以直接调试 `run_agent.py:main()`（独立 agent kernel 模式）或 `acp_adapter/entry.py:main()`（ACP 适配器模式）。各入口对应关系见 [pyproject.toml](https://github.com/nousresearch/hermes-agent/blob/main/pyproject.toml) 第 99–102 行的 `[project.scripts]`。

### 4. 添加断点

常用断点位置：

| 文件 | 位置 | 用途 |
|------|------|------|
| `run_agent.py` | `AIAgent.run()` L433 | 主循环入口 |
| `run_agent.py` | `run_conversation()` | Turn 执行逻辑 |
| `run_agent.py` | `AIAgent.__call__()` | 对外接口 |
| `cli.py` | `HermesCLI.cmd_chat()` | TUI 命令处理 |
| `model_tools.py` | `_discover_tools()` | 工具发现 |
| `hermes_state.py` | `SessionDB` 各方法 | 状态持久化 |

### 5. 调试模式启动

在 PyCharm 中直接点击 Debug（绿色甲虫图标），或使用右键菜单 "Debug 'main'"。

### 常见问题

- **断点无效**：确认 `venv` 是 PyCharm 项目的唯一解释器，而非系统全局 Python。
- **找不到模块** (`ModuleNotFoundError`)：确保 `PYTHONPATH` 包含 `hermes-agent/src`（或项目根目录，视源码布局而定）。
- **Windows 路径问题**：Windows 上使用反斜杠或 `pathlib.Path`，避免硬编码 Unix 路径。
- **断点停在虚拟环境代码**：在 PyCharm 中勾选 "Flatten packages" 并将 `venv` 目录标记为 Excluded。

---

## 相关资源

- **官方仓库**: <https://github.com/nousresearch/hermes-agent>
- **官方网站**: <https://hermes-agent.nousresearch.com>
- **快速入门文档**: <https://hermes-agent.nousresearch.com/docs/getting-started/quickstart>

<img src="hello-hermes.png" alt="hello-hermes" style="height:800px; display: block; margin-left: 0;" />
