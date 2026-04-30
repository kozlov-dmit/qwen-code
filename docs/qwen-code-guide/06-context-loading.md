# 6. Как Qwen Code заполняет контекст

Что именно попадает в контекст модели, когда и в каком порядке, и чем
управлять. Все настройки и команды — из официальной документации
[`docs/users/configuration/settings.md`](https://github.com/QwenLM/qwen-code/blob/main/docs/users/configuration/settings.md)
и [`docs/users/features/skills.md`](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/skills.md).

## 6.1 Источники контекста

Полный набор того, что в принципе может оказаться в окне модели:

| Источник | Когда подгружается |
|---|---|
| Системный промпт qwen-code | Захардкожен, на старте |
| Контекст-файлы (`QWEN.md` и/или из `context.fileName`) — иерархия user/project/dirs/extensions | Старт сессии, по `/memory refresh` |
| Auto-memory (`memory.enableManagedAutoMemory`, default `true`) | Фоновое извлечение фактов из диалога |
| Auto-dream / consolidation (`memory.enableManagedAutoDream`, default `false`) | Фоновая консолидация и дедупликация |
| Skills — `name`+`description` всех обнаруженных | На старте (метаданные) |
| Skills — тело `SKILL.md` и опциональные `reference.md`/`examples.md`/`scripts/`/`templates/` | Только при активации skill моделью (model-invoked, по `description`) |
| MCP-tools и DiscoveredTools | Старт сессии (описания функций) |
| Subagents | Своя изолированная сессия; Named — с нуля, Fork — наследует историю и system prompt родителя |
| `@file` / `@dir` в реплике пользователя | По мере того как пользователь их упоминает |
| Tool calls / tool results | По мере исполнения; old results авто-чистятся |

## 6.2 Что происходит при старте сессии

1. Читаются настройки в порядке приоритета (см. 6.5). Финальный
   эффективный конфиг — итог их слияния.
2. Определяется список имён контекст-файлов: `context.fileName` (строка
   или массив, например `["QWEN.md", "AGENTS.md"]`); если не задан —
   подразумевается `QWEN.md`.
3. Иерархический поиск контекст-файлов:
   - `~/.qwen/<contextFileName>` — глобально;
   - вверх по дереву от `cwd` до корня проекта — каждый найденный файл;
   - дополнительно — все каталоги из `context.includeDirectories`
     (если `context.loadFromIncludeDirectories: true` — подключаются
     и в `/memory refresh`);
   - контекст-файлы из активных extensions
     (`qwen-extension.json:contextFileName`).
4. Контент склеивается в системный промпт.
5. Регистрируются tools: built-in + MCP-серверы + DiscoveredTools +
   skills (только метаданные `name`/`description`) + agents
   (метаданные).
6. Применяются `permissions.allow`/`ask`/`deny` и `tools.approvalMode`.

## 6.3 Что добавляется во время сессии

- Реплики пользователя и ассистента — в историю.
- Tool calls и tool results — в историю; авто-очистка старых результатов
  по таймауту простоя (см. 6.4).
- `@file` / `@dir` — содержимое файлов, отфильтрованное `.gitignore`
  (если `respectGitIgnore: true`) и `.qwenignore`
  (`respectQwenIgnore: true`).
- `save_memory` (built-in tool) и `/memory add` (`/remember`) —
  добавляют запись в персистентную память.
- Активация skill: тело `SKILL.md` и те вспомогательные файлы, на
  которые он ссылается (`reference.md`, `examples.md`, `scripts/`,
  `templates/`).
- Авто-память: фоновое извлечение фактов
  (`enableManagedAutoMemory: true`); консолидация — если
  `enableManagedAutoDream: true`.

## 6.4 Авто-сжатие и очистка

- **Авто-сжатие** срабатывает при достижении
  `model.chatCompression.contextPercentageThreshold` (default `0.7`,
  т. е. 70 % окна модели). Сжимается история диалога.
- **Ручное** — `/compress`.
- **`model.maxSessionTurns`** (default `-1`, без лимита) обрезает
  количество ходов user/model/tool.
- **Очистка старых tool-results** —
  `context.clearContextOnIdle.toolResultsThresholdMinutes` (default
  `60`, `-1` отключает), сохраняется
  `context.clearContextOnIdle.toolResultsNumToKeep` (default `5`).

## 6.5 Иерархия и приоритет настроек

От низшего к высшему (выигрывает верхнее):

1. Default hardcoded values
2. System defaults file
3. `~/.qwen/settings.json` (user)
4. `.qwen/settings.json` (project)
5. System settings file
6. Environment variables
7. CLI arguments

## 6.6 Команды управления контекстом

- `/memory show` — что сейчас загружено.
- `/memory refresh` — перечитать `QWEN.md`/`AGENTS.md`/
  `includeDirectories`.
- `/memory add` (или `/remember`) — добавить запись.
- `/dream` — связан с `enableManagedAutoDream` (консолидация).
- `/compress` — сжать историю.
- `/stats` — токены/ходы/использование.
- `/skills`, `/agents`, `/extensions`, `/mcp` — посмотреть, что
  зарегистрировано в контексте.

## 6.7 Файлы и фильтры

- `.qwenignore` — собственный игнор-лист qwen для tool-операций
  (`respectQwenIgnore`, default `true`).
- `.gitignore` — учитывается по умолчанию (`respectGitIgnore: true`).
- `enableRecursiveFileSearch` / `enableFuzzySearch` — ускоряют и
  расширяют `@`-completions; можно отключить ради производительности.

## 6.8 Практические следствия

- Большой `QWEN.md` = постоянно занятые токены. Разбивайте по
  подкаталогам; в корне держите ссылки.
- Не кладите секреты в `QWEN.md` — он попадает в каждый запрос модели.
- Регулярно проверяйте, что попадает в контекст: `/memory show` +
  `/stats`.
- Для skills главное — `description`: только по нему модель решает
  подгружать тело. Триггеры/keywords — обязательно.
- На длинных сессиях не дожидайтесь авто-компрессии в 70 %: ручной
  `/compress` после крупного исследования сохраняет качество ответов.
- Для воспроизводимости в команде — фиксируйте `context.fileName`,
  `context.includeDirectories`,
  `model.chatCompression.contextPercentageThreshold` в проектном
  `.qwen/settings.json`.

## 6.9 Полезные ссылки

- [docs/users/configuration/settings.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/configuration/settings.md)
- [docs/users/features/skills.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/skills.md)
- [docs/users/features/sub-agents.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/sub-agents.md)
