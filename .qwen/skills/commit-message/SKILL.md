---
name: commit-message
description: Generate a Conventional Commits message from the currently staged changes. Triggers on "commit message", "conventional commit", "сгенерируй коммит". Returns a message ready to paste into `git commit -m`.
argument-hint: '[<scope-hint>]'
allowedTools:
  - read_file
  - shell
---

# Commit message generator

Produces a single Conventional Commits message describing the staged diff.

## Inputs

- `args` (optional): scope hint, e.g. `cli`, `core`, `docs`. If omitted —
  inferred from changed paths.

## Steps

1. `git diff --cached --stat` and `git diff --cached` to read staged changes.
   If nothing is staged — stop and tell the user to `git add` first.
2. Classify the change type by content:
   - `feat` — new user-visible capability
   - `fix` — bug fix
   - `docs` — only docs / comments
   - `refactor` — no behavior change
   - `perf`, `test`, `build`, `ci`, `chore`
3. Pick a scope: hint > top changed dir > omit.
4. Compose the subject (≤ 72 chars, imperative mood, no trailing period).
5. If the change touches > 5 files or has non-trivial logic — add a body
   with 1–3 short bullets (WHY, not WHAT).
6. Detect breaking changes (removed public API, schema migration, env var
   rename) → add `BREAKING CHANGE:` footer.

## Output

Return exactly one fenced block, ready to paste:

```
<type>(<scope>): <subject>

<optional body bullets>

<optional BREAKING CHANGE: ...>
```

Then a one-line preview of the `git commit` command.

## Why no subagent here

Single short task on already-fetched diff. Spawning a subagent costs more
in latency and tokens than it saves — keep it inline.

## Notes

- Never invent issue numbers or co-authors.
- If the diff mixes unrelated concerns — refuse and ask to split into
  multiple commits.
