---
name: debug-repro
description: Build a minimal reproducible example for a bug. Triggers on "minimal repro", "isolate the bug", "reproduce this", "собери репро". Produces a self-contained script or test that fails deterministically.
argument-hint: '<bug-description-or-issue-link>'
allowedTools:
  - read_file
  - glob
  - grep
  - shell
  - write_file
---

# Minimal reproducible example

Reduce a bug report to the smallest deterministic failing case.

## Inputs

- `args`: free-form bug description, stack trace, or link to issue.

## Steps

1. **Parse the report**: extract symptom, expected, suspected component,
   environment hints (versions, OS, env vars).
2. **Delegate context research in parallel** (see `## Delegation`). While
   subagents work, prepare the repro skeleton (`.qwen/scratch/repro/`).
3. **Build the repro**:
   - one file if possible (script, single test, single curl);
   - hardcode all inputs; remove network/external state;
   - include exact commands to run.
4. **Run it**. Iterate until it fails reliably **on a fresh shell**
   (cleared env, clean working tree).
5. **Reduce**: delete code/inputs and re-run after each cut. Stop when
   removing anything makes the bug disappear.
6. **Document**: write `REPRO.md` next to the script with: command,
   expected, actual, environment, version pins.

## Delegation

Two subagents, both **read-only and named**, run **in parallel** — they
explore independent areas and return small structured reports, so the
main skill can focus on building/reducing the repro.

| Subagent | Task | Why parallel & isolated |
|---|---|---|
| `bug-history-scout` | `git log -S`, `git log --grep`, recent commits to suspected files; related closed issues / PRs (via `shell` + `gh` or available MCP) | Large search surface; produces a short list of "looked at this before" pointers |
| `env-probe` | Versions of language/runtime/deps, lockfile state, OS, relevant env vars, container/sandbox hints | Independent of git history; needs different tools and avoids polluting history search |

If one of the two areas is irrelevant to the report (e.g. pure-logic bug
with no env angle) — skip that subagent.

Each subagent must return:

```json
{
  "findings": [ { "kind": "...", "evidence": "...", "next_step": "..." } ]
}
```

Do **not** delegate the actual reduction — it is iterative and depends on
running the repro after each cut, so a single sequential agent (this
skill) is faster.

## Output

A directory under `.qwen/scratch/repro/<short-slug>/` containing:

- the failing script/test,
- `REPRO.md` with run instructions, expected vs actual, environment,
- a one-paragraph summary in chat with the path and the failing command.

## Notes

- Never include real secrets in the repro — replace with placeholders.
- If the bug needs network — record a fixture (e.g. saved response) and
  replay locally; the goal is determinism.
- Stop reducing once the repro is < ~50 lines or further cuts hide the
  bug — diminishing returns.
