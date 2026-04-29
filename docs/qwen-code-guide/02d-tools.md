# 2d. Tools (custom commands, permissions, DiscoveredTools)

В Qwen Code «свой tool» — это, как правило, одно из трёх:

1. **Custom slash-command** (Markdown-шаблон) — самый простой способ.
2. **Subagent** — см. раздел [SubAgents](./02f-subagents.md).
3. **MCP-сервер** — настоящий программный tool. См. [MCP](./02e-mcp.md).

## Custom slash-commands

Раскладка:

```
.qwen/commands/
├── deploy.md            # /deploy
├── tests/
│   └── unit.md          # /tests:unit
└── gcs/
    └── sync.md          # /gcs:sync
```

Markdown-файл — это prompt-шаблон с заменой `{{args}}`. Подкаталоги
становятся неймспейсами через `:`. TOML-формат **deprecated**, при
обнаружении предлагается авто-миграция.

Пример `.qwen/commands/deploy.md`:

```markdown
---
description: Deploy current branch to staging
argument-hint: '<env=staging>'
---

Run `npm run deploy -- --env={{args}}` and report logs.
Validate the deploy by curling /healthz endpoint.
```

## Built-in tools

Файловая система: `read_file`, `write_file`, `edit`, `glob`, `grep`, `ls`.

Системные: `shell`, `web_fetch`, `web_search`, `save_memory`, `todo_write`.

Полный список — `docs/users/tools/*` в репозитории.

## Permissions

Новая система разрешений (старые `tools.allowed/exclude` авто-мигрируют):

```json
{
  "permissions": {
    "allow": ["Bash(git *)", "Read(/path/**)"],
    "ask":   ["Bash(git push *)"],
    "deny":  ["Bash(rm -rf *)", "Read(.env)"]
  }
}
```

Правила:

- `deny` сильнее `allow` — даже если паттерн есть в обоих списках, итог
  блокировка.
- Шаблоны вида `Bash(git *)`, `Read(.env)`, `Read(**/secrets/**)` — глобы
  по аргументам инструмента.
- Минимальный безопасный baseline: `Bash(rm -rf *)`, `Bash(curl *)`,
  `Read(.env)`, `Read(**/secrets/**)` в `deny`.

## Approval mode и sandbox

```json
{
  "tools": {
    "approvalMode": "auto-edit",
    "sandbox": "docker"
  }
}
```

- `tools.approvalMode`: `default` (спрашивать), `plan` (read-only анализ),
  `auto-edit` (редактировать без подтверждения, остальное спрашивать),
  `yolo` (без подтверждений).
- `tools.sandbox`: `docker`, `podman`, macOS seatbelt — изоляция шеллов и
  файловых операций.

## Полезные ссылки

- [docs/users/configuration/settings.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/configuration/settings.md)
- `docs/users/tools/` в репозитории QwenLM/qwen-code.
