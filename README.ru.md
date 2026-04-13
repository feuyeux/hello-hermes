# Hello Hermes Agent ☤

Это рабочее пространство для изучения и анализа [Hermes Agent](https://github.com/nousresearch/hermes-agent) `v0.8.0 (v2026.4.8)` `86960cdb`.

> ## Произношение
> 
> Примечание: «Hermes» в этом проекте относится к греческому божеству.
> 
> ✔️ **Hermes**: `/ˈhɜːrmiːz/` — Гермес, греческий бог языка и письменности, вестник богов.
> 
> ✖️ **Hermès**: `/ɛʁ.mɛs/` — Французский люксовый бренд.

## Анализ исходного кода

```sh
git clone --depth 1 --branch v2026.4.8 https://github.com/nousresearch/hermes-agent
```

- [Анализ архитектуры Hermes (часть 1): Поток · Полный жизненный цикл выполнения исходного кода](./Hermes%20架构解析%20(一)：流程篇%20·%20源代码执行全生命周期.md)
- [Анализ архитектуры Hermes (часть 2): Данные · Модель состояния и управление контекстом](./Hermes%20架构解析%20(二)：数据篇%20·%20状态模型与上下文治理.md)
- [Анализ архитектуры Hermes (часть 3): Расширение · Полное руководство по разработке плагинов и навыков](./Hermes%20架构解析%20(三)：扩展篇%20·%20插件与技能开发全指南.md)

## Быстрый старт

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
# Windows
powershell -Command "Set-ExecutionPolicy Bypass -Scope Process -Force; irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex"
```

```bash
# Запустить мастер настройки
hermes setup

# Просмотреть/отредактировать конфигурацию
code ~/.hermes/

# Начать интерактивный чат
hermes
```

## Отладка с точками останова в PyCharm

Отладка `hermes-agent/` с точками останова PyCharm:

### 1. Настройка проекта

```bash
cd hermes-agent
uv venv venv --python 3.11
source venv/bin/activate   # Windows: venv\Scripts\activate
uv pip install -e ".[all,dev]"
```

### 2. Настройка PyCharm

1. **Интерпретатор Python**: File → Project Structure → Add Python Interpreter, выберите `hermes-agent/venv/bin/python` (или `venv\Scripts\python.exe` в Windows)
2. **Корневая директория SDK**: Project → Project Structure, пометьте `hermes-agent/` как Sources
3. **Рабочая директория**: Project Structure → Modules → Paths → Add Content Root, добавьте `hermes-agent/src` (если существует)

### 3. Настройка конфигурации запуска

| Поле | Значение |
|------|----------|
| **Script path** | `hermes-agent/src/hermes_cli/main.py` |
| **Working directory** | `hermes-agent/` |
| **Environment variables** | `PYTHONPATH=hermes-agent/src` |

> **Точка входа**: `hermes_cli/main.py`'s `main()` — унифицированная точка входа CLI. Также можно отлаживать `run_agent.py:main()` (автономный режим ядра агента) или `acp_adapter/entry.py:main()` (режим адаптера ACP). См. [pyproject.toml](https://github.com/nousresearch/hermes-agent/blob/main/pyproject.toml) строки 99–102.

### 4. Добавление точек останова

| Файл | Расположение | Назначение |
|------|--------------|------------|
| `run_agent.py` | `AIAgent.run()` L433 | Вход в главный цикл |
| `run_agent.py` | `run_conversation()` | Логика выполнения Turn |
| `run_agent.py` | `AIAgent.__call__()` | Внешний интерфейс |
| `cli.py` | `HermesCLI.cmd_chat()` | Обработка команд TUI |
| `model_tools.py` | `_discover_tools()` | Обнаружение инструментов |
| `hermes_state.py` | Методы `SessionDB` | Сохранение состояния |

### 5. Запуск отладки

Нажмите Debug (зелёный жук) в PyCharm или используйте контекстное меню → "Debug 'main'".

### Частые проблемы

- **Точки останова не срабатывают**: Убедитесь, что `venv` — единственный интерпретатор проекта, а не системный Python.
- **Модуль не найден** (`ModuleNotFoundError`): Проверьте, что `PYTHONPATH` содержит `hermes-agent/src`.
- **Проблемы с путями в Windows**: Используйте обратные слэши или `pathlib.Path`, избегайте жёстко заданных Unix-путей.
- **Остановка в venv**: Пометьте папку `venv` как Excluded в PyCharm.

---

## Ресурсы

- **Официальный репозиторий**: https://github.com/nousresearch/hermes-agent
- **Официальный сайт**: https://hermes-agent.nousresearch.com
- **Документация по быстрому старту**: https://hermes-agent.nousresearch.com/docs/getting-started/quickstart

<img src="hello-hermes.png" alt="hello-hermes" style="height:800px; display: block; margin-left: 0;" />
