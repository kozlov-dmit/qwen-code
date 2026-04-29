---
name: code-review-self
description: Use before opening a PR to self-review the diff. Triggers on "review my changes", "before pr", "self review", "проверь мои изменения". Performs structured review covering correctness, tests, security, docs, and produces a blocker/nit/question report.
argument-hint: '[<base-branch=origin/main>]'
allowedTools:
  - read_file
  - glob
  - grep
  - shell
---

# Self code review

Pre-PR review of the current working branch against a base branch.

## Inputs

- `args` (optional): base branch. Default `origin/main`.

## Steps

1. Resolve diff range: `git fetch <base-remote>` then
   `git diff <base>...HEAD --stat` and `git diff <base>...HEAD`.
2. Build a file list and a short summary (purpose, scope, risk).
3. **Delegate four checks** (see `## Delegation` below) — in parallel
   if the runtime supports it.
4. Aggregate subagent reports. Drop duplicates. Keep severity in each item.
5. Output a final report in the format under `## Output`.

## Delegation

Spin up **named** subagents — they return structured reports back to this
skill. Run them **in parallel** when the runtime allows; the four areas are
independent and benefit from isolated context.

| Subagent | Scope | Why isolated |
|---|---|---|
| `correctness-reviewer` | Logic bugs, edge cases, error paths, off-by-one, concurrency | Largest context — own session keeps main clean |
| `tests-reviewer` | Coverage of new/changed behavior, brittle tests, missing negative cases | Different mental model; reads test files only |
| `security-reviewer` | Secrets, injection, authz, unsafe deserialization, dep risks | Separate checklist; benefits from read-only mode |
| `docs-reviewer` | Public API changes, README/CHANGELOG, comments WHY vs WHAT | Tiny scope, trivially parallelizable |

For each subagent send: the diff range, the changed-files list, and a
focused prompt with the scope above. Require each to answer in the same
JSON shape:

```json
{
  "items": [
    { "severity": "blocker|nit|question", "file": "...", "line": 0, "msg": "..." }
  ]
}
```

Skip a subagent only if its scope is clearly empty (e.g. no test files
touched and no behavior changes — skip `tests-reviewer`).

## Output

```
## Self review of <branch> vs <base>

### Summary
- Files changed: N
- Risk: low|medium|high
- One-line purpose

### Blockers
- file:line — message

### Nits
- file:line — message

### Questions
- file:line — message
```

## Notes

- Do **not** edit code in this skill — review only.
- If any subagent fails, surface the failure as a question, do not silently
  drop its area.
