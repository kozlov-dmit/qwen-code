# Qwen Code CLI — пособие для разработки

Практическое руководство по `@qwen-code/qwen-code` (форк Google Gemini CLI от
QwenLM, оптимизирован под модели Qwen). Все факты сверены с официальными
источниками; список ссылок — в конце [`10-sources.md`](./10-sources.md).

## Содержание

1. [Краткое описание возможностей](./01-overview.md)
2. Подробные инструкции по созданию:
   - [a. Skills (`SKILL.md`)](./02a-skills.md)
   - [b. Extensions (`qwen-extension.json`)](./02b-extensions.md)
   - [c. Rules / Memory (`QWEN.md`)](./02c-rules-memory.md)
   - [d. Tools (custom commands, permissions)](./02d-tools.md)
   - [e. MCP](./02e-mcp.md)
   - [f. SubAgents](./02f-subagents.md)
3. [Best practices](./03-best-practices.md)
4. [Готовые компоненты, которые реально помогают](./04-recommended.md)
5. [Советы по организации и настройке](./05-organization.md)

Источники и как верифицировать у себя — [`10-sources.md`](./10-sources.md).

## Кому полезно

Разработчикам, которые планируют использовать Qwen Code CLI для проектирования,
анализа, разработки и тестирования — индивидуально или в команде.

## Замечание о подтверждённости данных

Документ собирался не по «общему знанию», а по конкретным файлам репозитория
[QwenLM/qwen-code](https://github.com/QwenLM/qwen-code) и
[QwenLM/qwen-code-examples](https://github.com/QwenLM/qwen-code-examples).
Если фичи нет в источниках — её нет и в этом гайде.
