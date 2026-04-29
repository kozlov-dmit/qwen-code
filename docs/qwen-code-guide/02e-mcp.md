# 2e. MCP (Model Context Protocol)

## Что это и зачем

MCP — стандартный способ давать агенту инструменты (tools), ресурсы и
промпты из внешних программ (локальных или удалённых). Самый «правильный»
способ создавать настоящие кастомные инструменты для Qwen Code.

## Транспорты

Выбирается ровно один из трёх:

- **stdio** — `command` (локальный процесс).
- **SSE** — `url` (Server-Sent Events).
- **streamable HTTP** — `httpUrl`.

## Полный формат `mcpServers`

В `~/.qwen/settings.json` или `.qwen/settings.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "/usr/local/bin/github-mcp-server",
      "args": ["--read-only"],
      "env": { "GITHUB_TOKEN": "$GITHUB_TOKEN" },
      "cwd": "/work",
      "timeout": 30000,
      "trust": false,
      "includeTools": ["search_code", "get_pr"],
      "excludeTools": ["delete_repo"]
    },
    "remote-search": {
      "url": "https://example.com/sse",
      "headers": { "Authorization": "Bearer $TOKEN" }
    },
    "remote-http": {
      "httpUrl": "https://example.com/mcp"
    }
  },
  "mcp": {
    "allowed": ["github"],
    "excluded": []
  }
}
```

Поля:

- `command | url | httpUrl` — один транспорт.
- `args` — аргументы CLI для stdio.
- `env` — переменные окружения, поддержка `$VAR`.
- `cwd` — рабочая директория stdio-процесса.
- `timeout` — таймаут запроса, мс (default `600000`).
- `trust` — `true` пропускает подтверждения. **Только для проверенного
  локального кода.**
- `headers` — кастомные заголовки HTTP/SSE.
- `includeTools` / `excludeTools` — белый/чёрный список tools сервера.
  `excludeTools` сильнее.

Глобально на все серверы:

- `mcp.allowed[]` — если задан, разрешены только перечисленные серверы.
- `mcp.excluded[]` — чёрный список (deny выигрывает у allow).

## OAuth

Удалённые MCP-сервера могут требовать OAuth 2.0. Поддержка:

- автоматический discovery эндпоинтов и dynamic client registration;
- браузерный flow;
- токены — `~/.qwen/mcp-oauth-tokens.json`;
- `redirectUri` по умолчанию `http://localhost:7777/oauth/callback` —
  для облачных сетапов нужно указать публично доступный.

Провайдеры: `dynamic_discovery`, `google_credentials`,
`service_account_impersonation`.

## CLI и слэш

```bash
qwen mcp add | remove | list
```

```text
/mcp           # статус серверов и список tools
/mcp auth      # OAuth-вход
```

## Имена и коллизии

При коллизии tool из разных серверов добавляется префикс: `<server>__<tool>`.

## Установка готовых MCP через extensions

Дословный пример из доки:

```bash
qwen extensions install https://github.com/github/github-mcp-server
```

Это подключит GitHub MCP-сервер как extension.

## Полезные ссылки

- [docs/developers/tools/mcp-server.md](https://github.com/QwenLM/qwen-code/blob/main/docs/developers/tools/mcp-server.md)
- [docs/users/configuration/settings.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/configuration/settings.md)
