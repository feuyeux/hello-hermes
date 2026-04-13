# Hello Hermes Agent ☤

This is a workspace for exploring and analyzing [Hermes Agent](https://github.com/nousresearch/hermes-agent) `v0.8.0 (v2026.4.8)` `86960cdb`.

> ## Pronunciation
>
> Note: The "Hermes" in this project refers to the Greek deity.
>
> ✔️ **Hermes**: `/ˈhɜːrmiːz/` — The Greek god of language and writing, and messenger of the gods.
>
> ✖️ **Hermès**: `/ɛʁ.mɛs/` — French luxury brand.

## Source Code Analysis

```sh
git clone --depth 1 --branch v2026.4.8 https://github.com/nousresearch/hermes-agent
```

- [Hermes Architecture Analysis (Part 1): Flow · Full Source Code Execution Lifecycle](./Hermes%20架构解析%20(一)：流程篇%20·%20源代码执行全生命周期.md)
- [Hermes Architecture Analysis (Part 2): Data · State Model and Context Governance](./Hermes%20架构解析%20(二)：数据篇%20·%20状态模型与上下文治理.md)
- [Hermes Architecture Analysis (Part 3): Extension · Complete Guide to Plugins and Skills Development](./Hermes%20架构解析%20(三)：扩展篇%20·%20插件与技能开发全指南.md)

## Quick Start

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

## PyCharm Breakpoint Debugging

Debugging `hermes-agent/` with PyCharm breakpoints:

### 1. Configure Project

```bash
cd hermes-agent
uv venv venv --python 3.11
source venv/bin/activate   # Windows: venv\Scripts\activate
uv pip install -e ".[all,dev]"
```

### 2. Configure PyCharm

1. **Python Interpreter**: File → Project Structure → Add Python Interpreter, select `hermes-agent/venv/bin/python` (or `venv\Scripts\python.exe` on Windows)
2. **SDK Root**: Project → Project Structure, mark `hermes-agent/` as Sources
3. **Working Directory**: Project Structure → Modules → Paths → Add Content Root, add `hermes-agent/src` (if it exists)

### 3. Configure Run Configuration

| Field | Value |
|-------|-------|
| **Script path** | `hermes-agent/src/hermes_cli/main.py` |
| **Working directory** | `hermes-agent/` |
| **Environment variables** | `PYTHONPATH=hermes-agent/src` (or the appropriate `src` directory path) |

> **Entry Point**: `hermes_cli/main.py`'s `main()` is the unified CLI entry. You can also debug `run_agent.py:main()` (standalone agent kernel mode) or `acp_adapter/entry.py:main()` (ACP adapter mode). See [pyproject.toml](https://github.com/nousresearch/hermes-agent/blob/main/pyproject.toml) lines 99–102 for the `[project.scripts]` mapping.

### 4. Add Breakpoints

| File | Location | Purpose |
|------|----------|---------|
| `run_agent.py` | `AIAgent.run()` L433 | Main loop entry |
| `run_agent.py` | `run_conversation()` | Turn execution logic |
| `run_agent.py` | `AIAgent.__call__()` | External interface |
| `cli.py` | `HermesCLI.cmd_chat()` | TUI command handling |
| `model_tools.py` | `_discover_tools()` | Tool discovery |
| `hermes_state.py` | `SessionDB` methods | State persistence |

### 5. Start Debugging

Click Debug (green beetle icon) in PyCharm, or right-click → "Debug 'main'".

### Troubleshooting

- **Breakpoints not hitting**: Ensure `venv` is the sole interpreter for this project, not the system global Python.
- **Module not found** (`ModuleNotFoundError`): Ensure `PYTHONPATH` includes `hermes-agent/src` (or project root depending on source layout).
- **Windows path issues**: Use backslashes or `pathlib.Path`, avoid hardcoded Unix paths.
- **Breakpoints stop in venv**: Mark the `venv` folder as Excluded in PyCharm.

---

## Resources

- **Official Repository**: <https://github.com/nousresearch/hermes-agent>
- **Official Website**: <https://hermes-agent.nousresearch.com>
- **Quickstart Documentation**: <https://hermes-agent.nousresearch.com/docs/getting-started/quickstart>

<img src="hello-hermes.png" alt="hello-hermes" style="height:800px; display: block; margin-left: 0;" />
