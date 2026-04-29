# 2b. Extensions (`qwen-extension.json`)

## Что это и зачем

Расширение — пакет, который агрегирует MCP-серверы, subagents, skills,
custom commands и контекст в один устанавливаемый артефакт. Удобно, чтобы
шарить интеграции между командами и проектами или ставить готовые наборы из
маркетплейсов Gemini / Claude Code.

## Где живут

`~/.qwen/extensions/<name>/` — внутри обязателен `qwen-extension.json`.

## Полный манифест

Дословно из `docs/users/extension/introduction.md`:

```json
{
  "name": "my-extension",
  "version": "1.0.0",
  "mcpServers": {
    "my-server": { "command": "node my-server.js" }
  },
  "channels": {
    "my-platform": { "entry": "dist/index.js", "displayName": "My Platform Channel" }
  },
  "contextFileName": "QWEN.md",
  "commands": "commands",
  "skills": "skills",
  "agents": "agents",
  "settings": [
    {
      "name": "API Key",
      "description": "Your API key for the service",
      "envVar": "MY_API_KEY",
      "sensitive": true
    }
  ]
}
```

Поля:

- `name`, `version` — идентификатор и версия (имя в нижнем регистре, через
  дефис; должно совпадать с именем папки).
- `mcpServers` — то же, что в `settings.json`, но без `trust`. Если такой же
  ключ есть и в `settings.json` — приоритет у `settings.json`.
- `channels` — кастомные channel-адаптеры (`entry` указывает на JS-файл,
  экспортирующий объект `plugin` интерфейса `ChannelPlugin`).
- `contextFileName` — имя файла-контекста внутри extension (по умолчанию
  `QWEN.md`).
- `commands`, `skills`, `agents` — имена подпапок (по умолчанию совпадают с
  типом). Команды — Markdown-файлы, skills — папки с `SKILL.md`,
  агенты — `.yaml` или `.md`.
- `settings[]` — спрашиваются у пользователя при установке, складываются в
  `~/.qwen/.env` (user) или `.qwen/.env` (workspace) и пробрасываются в
  MCP-сервера как env vars.

## Источники установки

```bash
qwen extensions install <owner>/<repo>             # Git shorthand
qwen extensions install https://github.com/...     # Git URL
qwen extensions install /path/to/local             # Локально (создаётся копия)
qwen extensions install @scope/my-extension@1.2.0  # npm-scoped, поддержка --registry, NPM_TOKEN
qwen extensions install <marketplace>:<plugin>     # Claude Marketplace plugin
qwen extensions install <gemini-extension-url>     # Gemini Gallery (auto-convert)
```

## Управление

```bash
qwen extensions list
qwen extensions update <name>          # или --all
qwen extensions disable <name> [--scope=workspace]
qwen extensions enable  <name> [--scope=workspace]
qwen extensions uninstall <name>
qwen extensions settings set/list/show/unset <ext> <setting> [--scope user|workspace]
```

В runtime: `/extensions` (manage, hot-reload), `/extensions install <source>`,
`/extensions explore Gemini|ClaudeCode`.

## Подстановка переменных

В `qwen-extension.json` можно использовать:

- `${extensionPath}` — полный путь до папки расширения;
- `${workspacePath}` — текущий рабочий проект;
- `${/}` или `${pathSeparator}` — OS-зависимый разделитель путей.

## Конфликт-резолюция команд

При коллизии имён команд из разных источников добавляется префикс
расширения: `[ext-name].command`. Например, `gcp` extension с командой
`deploy` доступна как `/gcp.deploy`, если уже есть пользовательская
`/deploy`.

## Полезные ссылки

- [docs/users/extension/introduction.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/extension/introduction.md)
