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

## Полезные ссылки

- [docs/users/features/skills.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/skills.md)
- [packages/core/src/skills/bundled/qc-helper/SKILL.md](https://github.com/QwenLM/qwen-code/blob/main/packages/core/src/skills/bundled/qc-helper/SKILL.md)
- [.qwen/skills/qwen-code-claw/SKILL.md](https://github.com/QwenLM/qwen-code/blob/main/.qwen/skills/qwen-code-claw/SKILL.md)
