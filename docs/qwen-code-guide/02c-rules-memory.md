# 2c. Rules / Memory (`QWEN.md`)

## Что это и зачем

`QWEN.md` — «инструкционный контекст» / «память» агента: файл с правилами
стиля, архитектуры, конвенций, который Qwen Code держит в системном
промпте. Аналог `CLAUDE.md` / `AGENTS.md`.

Используйте, чтобы зафиксировать:

- кодстайл и принятые конвенции;
- структуру проекта и правила навигации;
- запреты (например, «не трогать legacy-папку»);
- ссылки на дизайн-доки и важные ADR.

## Иерархия загрузки

- `~/.qwen/QWEN.md` — глобально на все проекты.
- `.qwen/QWEN.md` или корневой `QWEN.md` в проекте — командные правила (в
  git).
- Файлы в подкаталогах — локальный контекст для конкретного модуля.
- `contextFileName` из любого активного extension.

Имя настраивается через `context.fileName` в `settings.json`:

```json
{
  "context": {
    "fileName": ["QWEN.md", "AGENTS.md"]
  }
}
```

Так один и тот же файл правил подхватывают и Claude Code, и Gemini CLI —
без дубликатов.

## Управление

- `/memory show` — показать, что сейчас загружено.
- `/memory refresh` — перечитать файлы (после ручных правок).
- `/memory add` — добавить заметку в память.

Авто-память:

- `memory.enableManagedAutoMemory` (по умолчанию `true`) — фоновое извлечение
  памяти из диалогов.
- `memory.enableManagedAutoDream` (по умолчанию `false`) — авто-консолидация
  накопленной памяти.

## Референс

В корне самого `QwenLM/qwen-code` лежит подтверждённый `AGENTS.md` с
разделами `Common Commands`, `Code Conventions`, `Development Guidelines`,
ссылками на используемые skills (`/feat-dev`, `/bugfix`) и `.qwen/`-папки
(`design/`, `e2e-tests/`, `pr-drafts/` и т. п.). Хороший шаблон для своих
правил.

> Полная картина того, что и когда попадает в контекст модели —
> в разделе [6. Как Qwen Code заполняет контекст](./06-context-loading.md).

## Полезные ссылки

- [docs/users/configuration/settings.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/configuration/settings.md)
- [QwenLM/qwen-code AGENTS.md](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md)
