# 2f. SubAgents

## Что это и зачем

Изолированный AI-агент со своим системным промптом, набором tools, моделью
и approval-режимом. Хорош, чтобы делегировать узкую задачу (тестирование,
код-ревью, поиск багов) и не засорять основной контекст.

Используйте subagent, когда:

- задача требует много дополнительного исследования (сэкономит токены
  основного диалога);
- нужно ограничить набор инструментов (например, «только чтение, без
  редактирования исходников»);
- хочется выделенную модель (`model` поле) под конкретный профиль работы.

## Где живут

- `.qwen/agents/` — проектные.
- `~/.qwen/agents/` — личные.
- `agents/` внутри extension.

Файлы — `.md` с YAML-frontmatter или `.yaml`.

## Поля frontmatter

Подтверждено в `docs/users/features/sub-agents.md`:

- `name` — слаг.
- `description` — actionable-описание, по которому main-агент решит
  делегировать.
- `tools` — белый список инструментов.
- `disallowedTools` — чёрный список.
- `model` — какая модель запускает агента.
- `approvalMode` — `auto-edit`, `plan`, `default`, `yolo`.

Тело Markdown — детальный системный промпт с инструкциями.

## Минимальный пример

```markdown
---
name: test-engineer
description: Reproduces bugs via E2E testing and verifies fixes. Read-only — does not edit source code.
tools:
  - read_file
  - glob
  - grep
  - shell
disallowedTools:
  - write_file
  - edit
model: qwen3-coder-plus
approvalMode: plan
---

You are a test engineer. Your job:
1. Read the failing scenario from the user.
2. Reproduce it via headless or interactive E2E.
3. Report observed vs expected with logs.
4. After a fix is applied, re-run and verify.
```

## Виды

- **Named** — стартует с чистого листа, со своим system prompt и пустой
  историей.
- **Fork** — наследует историю и system prompt родителя; параллельная работа
  с shared prompt-кешем. **Важное ограничение:** результаты forka видны в
  UI прогресса, но **не возвращаются автоматически в основной диалог** —
  родитель не может действовать на их вывод напрямую.

## Управление

- `/agents create` — мастер создания.
- `/agents manage` — список, включение/отключение, редактирование.

Extension-агенты появляются в менеджере под секцией «Extension Agents».

## Полезные ссылки

- [docs/users/features/sub-agents.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/sub-agents.md)
- [QwenLM/qwen-code AGENTS.md](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md) — упоминает референсный subagent `test-engineer`.
