# 2a. Skills (`SKILL.md`)

## Что это и зачем

Skill — модульная капабилити: папка с `SKILL.md` (YAML-frontmatter +
Markdown) и опциональными `reference.md`, `examples.md`, `scripts/`,
`templates/`. Skill **model-invoked**: модель сама решает, когда подгрузить,
опираясь на `description`. Это удобнее, чем «всегда в памяти», и сильно
дешевле по токенам.

Используйте skills, когда нужно:

- закодировать регулярную процедуру (deploy, миграция, аудит);
- упаковать узкое экспертное знание (PDF-обработка, генерация изображений);
- собрать переиспользуемый «рецепт», который агент должен подхватывать
  только по триггеру.

## Где живут

- `~/.qwen/skills/` — личные, доступны во всех проектах.
- `.qwen/skills/` — проектные, коммитятся в git и доступны команде.
- `skills/` внутри extension (имя поддиректории настраивается полем `skills`
  в `qwen-extension.json`).
- Bundled — `packages/core/src/skills/bundled/` внутри самого qwen-code.

## Минимальный пример

Формат frontmatter подтверждён в `qc-helper/SKILL.md`:

```markdown
---
name: pdf-processor
description: Use when the user asks to extract text/tables from PDF files. Triggers on "pdf", "extract pdf", "parse pdf".
argument-hint: '<path-or-url>'
allowedTools:
  - read_file
  - glob
  - shell
---

# PDF processor

## Instructions
1. Locate the PDF (path or URL).
2. If URL — download via `web_fetch`, save in `./tmp/`.
3. Run `pdftotext` через shell, верни текст.

## Examples
- "extract pdf ./report.pdf" — return plain text
- "parse pdf https://..." — download then parse
```

Поля frontmatter, явно подтверждённые в репо:

- `name` — слаг, имя папки совпадает с этим полем.
- `description` — описание + триггер-ключи; именно по нему модель решает,
  подключать ли skill.
- `argument-hint` — подсказка к аргументу (для UI и автокомплита).
- `allowedTools` — белый список инструментов, доступных skill.

## Управление

Слэш-команда `/skills` показывает все обнаруженные skills (личные,
проектные, из extensions, bundled). Активация — автоматическая, по описанию.

## Как добиться, чтобы Qwen Code выбирал skill, а не делал сам

Модель решает «подключить skill или справиться напрямую» по `description`
во frontmatter. Если описание расплывчатое — агент чаще игнорирует skill и
решает задачу собственными tool-ами (read_file, shell, edit), теряя
проверенный сценарий. Несколько правил, которые повышают точность выбора:

1. **Description = «когда вызывать» + триггеры.** Начинайте с фразы
   `Use when ...` / `Триггерится на ...` и явно перечисляйте ключевые
   слова и формулировки запроса пользователя. Пример из `qc-helper`:
   `Use when the user asks about Qwen Code itself, settings.json, ...`.
   Эти токены модель сопоставляет с пользовательским сообщением.
2. **Перечисляйте задачи, а не возможности.** Вместо
   `Helps with PDFs` пишите конкретные действия: `extract text from PDF`,
   `parse tables from PDF`, `summarize PDF report`. Чем ближе формулировка
   к тому, как пользователь спрашивает, тем выше совпадение.
3. **Добавляйте антипримеры.** Секция вида
   `Do NOT use for: ... (use <other-skill> instead)` снимает конфликты
   между похожими skills и не даёт модели подхватить не тот рецепт.
4. **Один skill — одна капабилити.** Если description перечисляет 5
   разнородных задач, модель не уверена в применимости и выбирает
   «сделать самой». Дробите на отдельные skills с узкими триггерами.
5. **Не дублируйте триггеры между skills.** Пересекающиеся ключевые слова
   создают неоднозначность; при необходимости в `description` явно
   различайте их («for backend bugs only — frontend bugs go to ui-bugfix»).
6. **Имя skill = слаг триггера.** `pdf-extract` лучше, чем `helper-1`:
   модель учитывает имя при ранжировании.
7. **Узкий `allowedTools`.** Если skill ограничен read_file/shell, модель
   видит, что без него многих tool-ов не получит, и охотнее активирует.
8. **Проверяйте на реальных формулировках.** Запустите `/skills`, затем
   задайте 5–10 пользовательских реплик, которыми обычно начинается
   задача. Если skill не подхватывается — добавляйте недостающие
   триггеры в `description` и повторяйте.
9. **Закрепляйте «обязательные» skills через QWEN.md.** Для skills,
   которые должны включаться всегда (`qc-helper`, внутренний
   `release-checklist` и т. п.), добавьте в `QWEN.md` строку вида
   `Always invoke /<skill> for <task>` — память подхватится в системный
   промпт и будет жёстким напоминанием.
10. **`/memory refresh` после правок.** Изменили `description` или
    `QWEN.md` — обновите память, иначе старая версия так и будет
    конкурировать с новой.

Быстрый чек-лист «почему skill не сработал»:

- description слишком общий или без триггер-слов;
- триггеры пересекаются с другим skill;
- имя файла не совпадает с `name` (skill не загрузился — проверьте `/skills`);
- skill лежит вне `~/.qwen/skills/`, `.qwen/skills/` или `skills/`
  активного extension;
- `allowedTools` пустой/слишком широкий — модель не видит выгоды;
- сессия не перечитала память (`/memory refresh`).

## Полезные ссылки

- [docs/users/features/skills.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/skills.md)
- [packages/core/src/skills/bundled/qc-helper/SKILL.md](https://github.com/QwenLM/qwen-code/blob/main/packages/core/src/skills/bundled/qc-helper/SKILL.md)
- [.qwen/skills/qwen-code-claw/SKILL.md](https://github.com/QwenLM/qwen-code/blob/main/.qwen/skills/qwen-code-claw/SKILL.md)
