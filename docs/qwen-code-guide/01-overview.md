# 1. Краткое описание возможностей

Qwen Code — open-source AI-агент в терминале, **форк Google Gemini CLI**,
оптимизированный под модели семейства Qwen (Qwen3-Coder в вариантах 480A35 /
30BA3B и Qwen3.x-Plus). При этом он мульти-провайдерный — поддерживает
OpenAI-, Anthropic- и Gemini-совместимые API и Alibaba Cloud Coding Plan.

## Установка

```bash
# Bash-инсталлер от Alibaba (Linux/macOS)
bash -c "$(curl -fsSL https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen.sh)"

# npm
npm install -g @qwen-code/qwen-code@latest

# Homebrew
brew install qwen-code
```

Требуется Node.js ≥ 20.

## Аутентификация

API-ключи через переменные окружения:

- `DASHSCOPE_API_KEY` — Alibaba Cloud Model Studio
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `GEMINI_API_KEY`
- `BAILIAN_CODING_PLAN_API_KEY` — Alibaba Cloud Coding Plan

Бесплатный Qwen OAuth прекращён 15 апреля 2026 — нужен переход на ключи или
подписку.

## Режимы работы

- **Интерактивный TTY** — основной.
- **Headless** через `-p` / `--prompt` — для скриптов и CI.
- **IDE-интеграции** — VS Code, Zed, JetBrains.
- **SDK** — TypeScript / Python / Java.

## Built-in tools

Файловая система (`read_file`, `write_file`, `edit`, `glob`, `grep`, `ls`),
shell, `web_fetch`, `web_search`, `save_memory`, `todo_write` и др.
Плюс встроенные «Skills» и «SubAgents» — дают Claude-Code-подобный workflow.

## Подсистемы кастомизации

Extensions, Skills, SubAgents, Custom Commands, MCP, Memory (`QWEN.md`),
Permissions. Каждой посвящён отдельный раздел руководства.

## Режимы подтверждения и песочница

`tools.approvalMode` — `default`, `auto-edit`, `plan`, `yolo`. Песочница:
`docker` / `podman` / macOS seatbelt.

## Основные slash-команды

`/help`, `/clear`, `/compress`, `/stats`, `/auth`, `/model`, `/memory`,
`/tools`, `/mcp`, `/skills`, `/agents`, `/extensions`, `/bug`, `/exit`. Плюс
`/extensions install|explore`, `/mcp auth`, `/agents create|manage`.

## Совместимость

Расширения и плагины из **Gemini CLI Extensions Gallery** и **Claude Code
Marketplace** ставятся напрямую с автоконвертацией манифеста, агентов, skills
и команд (TOML→Markdown).
