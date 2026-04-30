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

## Подсказка skills через `QWEN.md`

`QWEN.md` — удобное место, чтобы напомнить агенту про важные skills,
которые он иначе пытается заменить ручной работой. Несколько шаблонов:

```markdown
## Skills routing
- For any task that mentions «release», «changelog», «bump version» —
  ALWAYS invoke `/release-checklist` skill before editing files.
- Reproducing a bug? Use `/bugfix` (reproduce-first). Не пытайся чинить
  напрямую без воспроизведения.
- E2E / browser-сценарии — только через `/e2e-testing`, не запускай
  Playwright руками.
- Вопросы про сам Qwen Code, settings.json, permissions — `/qc-helper`.
```

Что именно работает:

- **Глаголы императива** (`ALWAYS invoke`, `Do NOT edit until …`) —
  модель воспринимает их как правило, а не пожелание.
- **Привязка к триггер-словам.** Перечислите фразы пользователя,
  которые должны включать конкретный skill — это дублирует триггеры
  из `description` и повышает точность выбора.
- **Список «не делай сам».** Явно запрещайте обходные пути:
  «не запускай pytest вручную — используй `/e2e-testing`».
- **Регулярный `/memory refresh`** после правок и проверка через
  `/memory show`, что напоминалки реально загружены.

## Референс

В корне самого `QwenLM/qwen-code` лежит подтверждённый `AGENTS.md` с
разделами `Common Commands`, `Code Conventions`, `Development Guidelines`,
ссылками на используемые skills (`/feat-dev`, `/bugfix`) и `.qwen/`-папки
(`design/`, `e2e-tests/`, `pr-drafts/` и т. п.). Хороший шаблон для своих
правил.

## Полезные ссылки

- [docs/users/configuration/settings.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/configuration/settings.md)
- [QwenLM/qwen-code AGENTS.md](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md)
