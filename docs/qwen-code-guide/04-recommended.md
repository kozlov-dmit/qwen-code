# 4. Готовые компоненты, которые реально помогают

Перечислены только компоненты, прямо упомянутые в официальных репозиториях
QwenLM. Дополнительно на практике почти всегда полезен общий MCP-набор
(GitHub, filesystem, Playwright, search), но конкретные имена пакетов из
community лучше выбирать самостоятельно — здесь чужие неподтверждённые
имена не приводятся.

## Bundled / референсные skills внутри `QwenLM/qwen-code`

- **`qc-helper`** — отвечает на вопросы по самому Qwen Code, помогает
  редактировать `settings.json`. Источник:
  [packages/core/src/skills/bundled/qc-helper/SKILL.md](https://github.com/QwenLM/qwen-code/blob/main/packages/core/src/skills/bundled/qc-helper/SKILL.md).
- **`qwen-code-claw`** — общий code-agent для понимания кода, генерации
  проектов, фиксов, рефакторинга
  ([`.qwen/skills/qwen-code-claw/SKILL.md`](https://github.com/QwenLM/qwen-code/blob/main/.qwen/skills/qwen-code-claw/SKILL.md)).
- **`feat-dev`** — полный feature-flow (исследование → дизайн → тест-план →
  имплементация → ревью).
- **`bugfix`** — reproduce-first ремонт багов.
- **`e2e-testing`** — headless / interactive (tmux) / MCP-server / API
  traffic тесты.
- **`structured-debugging`** — для трудных багов, когда первый фикс не
  сработал.

Эти skills документированы в корневом
[`AGENTS.md`](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md)
репозитория и являются лучшими референсами «как писать свои skills для
разработки/тестирования/отладки».

## QwenLM/qwen-code-examples

Официальная коллекция:

- **Featured skills**:
  - YouTube Transcript Extractor;
  - Image Generation (DashScope / Wanx);
  - Auto PR (review + docs);
  - Dashboard Builder (React/Next + Recharts).
- **5 demo-приложений** (ProtoFlow, DesignPrint, CoWork, NoteGenius,
  PromoStudio) — полезны как эталон structure.

Репозиторий: [QwenLM/qwen-code-examples](https://github.com/QwenLM/qwen-code-examples).

## Extensions / MCP — общие маркетплейсы

- **Gemini CLI Extensions Gallery** — `/extensions explore Gemini`,
  ставится напрямую (`qwen extensions install <gemini-url>`),
  `gemini-extension.json` авто-конвертируется.
- **Claude Code Marketplace** — `/extensions explore ClaudeCode`,
  формат `marketplace:plugin`. Дословный пример из доки:

  ```bash
  qwen extensions install f/awesome-chatgpt-prompts:prompts.chat
  ```

- Любой MCP-сервер из GitHub. Дословный пример из доки:

  ```bash
  qwen extensions install https://github.com/github/github-mcp-server
  ```

## Профиль «проектирование / анализ / разработка / тестирование»

Из подтверждённого списка можно собрать рабочее окружение:

- **Skills**: `qc-helper` (постоянно), `feat-dev`, `bugfix`,
  `e2e-testing`, `structured-debugging`.
- **MCP / Extensions**: GitHub MCP server (`github/github-mcp-server`) —
  явно фигурирует в примерах документации.
- **Subagent**: паттерн `test-engineer` из `AGENTS.md` qwen-code (агент,
  который читает код, воспроизводит баги через E2E, не редактирует
  исходники).
