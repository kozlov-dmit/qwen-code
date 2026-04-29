# Источники и верификация

## Источники

Документ собран строго по официальным источникам QwenLM:

- [QwenLM/qwen-code — README](https://github.com/QwenLM/qwen-code)
- [docs/users/extension/introduction.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/extension/introduction.md)
- [docs/users/features/skills.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/skills.md)
- [docs/users/features/sub-agents.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/sub-agents.md)
- [docs/users/configuration/settings.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/configuration/settings.md)
- [docs/developers/tools/mcp-server.md](https://github.com/QwenLM/qwen-code/blob/main/docs/developers/tools/mcp-server.md)
- [QwenLM/qwen-code AGENTS.md](https://github.com/QwenLM/qwen-code/blob/main/AGENTS.md)
- [packages/core/src/skills/bundled/qc-helper/SKILL.md](https://github.com/QwenLM/qwen-code/blob/main/packages/core/src/skills/bundled/qc-helper/SKILL.md)
- [.qwen/skills/qwen-code-claw/SKILL.md](https://github.com/QwenLM/qwen-code/blob/main/.qwen/skills/qwen-code-claw/SKILL.md)
- [QwenLM/qwen-code-examples](https://github.com/QwenLM/qwen-code-examples)
- [DeepWiki: QwenLM/qwen-code](https://deepwiki.com/QwenLM/qwen-code)

Официальный сайт документации — `https://qwenlm.github.io/qwen-code-docs/`
(те же страницы лежат в `docs/` основного репозитория).

## Как верифицировать у себя

Если установить qwen у себя локально:

1. `npm i -g @qwen-code/qwen-code && qwen --version`
2. `qwen` — запустить интерактив, затем последовательно:
   `/help`, `/skills`, `/agents`, `/extensions`, `/mcp` — убедиться, что
   команды живые.
3. Создать `.qwen/skills/demo/SKILL.md` с frontmatter `name` + `description`,
   запустить `/skills` и попросить агента «used demo skill».
4. `qwen extensions install owner/repo` — проверить установку из git.
5. Положить тестовый `mcpServers` в `.qwen/settings.json`, перезапустить,
   `/mcp` должен показать tools с префиксом сервера.

## Замечание

Документ был подготовлен на основе чтения документации, без live-проверки
`qwen --help` (локальный репозиторий на момент составления был пустым
placeholder'ом). Все факты и примеры взяты из перечисленных выше файлов
официального репозитория.
