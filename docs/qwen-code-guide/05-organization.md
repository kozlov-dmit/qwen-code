# 5. Советы по организации и настройке

## Двухуровневая раскладка

- **Личные дефолты** — в `~/.qwen/` (model, approval, permissions, любимые
  skills/agents).
- **Проектные правила и инструменты** — в `.qwen/` под git. Проектный
  `settings.json` точечно оверрайдит личный.

## Приоритет настроек

От высшего к низшему:

1. CLI args
2. env vars
3. system settings (`/etc/qwen-code/settings.json` на Linux)
4. project (`.qwen/settings.json`)
5. user (`~/.qwen/settings.json`)
6. defaults

## Минимальный starter `~/.qwen/settings.json`

```json
{
  "general": { "vimMode": true, "enableAutoUpdate": true },
  "model": { "name": "qwen3-coder-plus", "maxSessionTurns": 20 },
  "context": { "fileName": ["QWEN.md", "AGENTS.md"] },
  "memory": { "enableManagedAutoMemory": true },
  "tools": { "approvalMode": "auto-edit", "sandbox": "docker" },
  "permissions": {
    "allow": ["Bash(git *)", "Bash(npm *)"],
    "ask":   ["Bash(git push *)"],
    "deny":  ["Bash(rm -rf *)", "Read(.env)", "Read(**/secrets/**)"]
  }
}
```

## Секреты

- `~/.qwen/.env` — личные.
- `.qwen/.env` — проектные (через
  `qwen extensions settings set ... --scope workspace`).
- В git — никогда. Добавьте `.qwen/.env` в `.gitignore`.
- OAuth-токены MCP лежат в `~/.qwen/mcp-oauth-tokens.json` сами.

## Approval-профили под задачи

Удобно держать заготовки:

- `plan` — для аудита и архитектуры.
- `auto-edit` — повседневно.
- `yolo` — только в `tools.sandbox: "docker"` или CI.

## Память актуальной

После любых правок `QWEN.md` — `/memory refresh`. Регулярно
`/memory show`, чтобы видеть, что грузится.

## Гигиена сессии

- `/stats` — мониторит токены.
- `/compress` — сжимает историю до критичной точки.
- Длинные refactor'ы лучше делить на сессии.

## Поддержание актуальности

- `enableAutoUpdate: true` в `general`.
- Периодически `qwen extensions update --all` и `/mcp` после апдейтов
  (отвалившиеся сервера видны сразу).

## Шаринг по команде

Коммитьте в репозиторий проекта:

- `.qwen/skills/`
- `.qwen/agents/`
- `.qwen/commands/`
- `QWEN.md` (и/или `AGENTS.md`)
- проектный `.qwen/settings.json` (без секретов)

Это превращается в воспроизводимое рабочее окружение для агента наравне с
lockfile.

## Кросс-совместимость

Поскольку qwen-code умеет ставить расширения из Gemini Gallery и Claude
Marketplace и понимает `AGENTS.md`, можно переиспользовать инфраструктуру,
написанную для Claude Code / Gemini CLI, без дублирования.
