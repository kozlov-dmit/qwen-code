---
name: refactor-stepwise
description: Refactor code in small verifiable steps with tests passing between each. Triggers on "refactor", "extract module", "rename across project", "split this function". Produces a step plan, then executes it iteratively.
argument-hint: '<target-file-or-symbol>'
allowedTools:
  - read_file
  - glob
  - grep
  - edit
  - write_file
  - shell
---

# Stepwise refactor

Behavior-preserving refactor in small commits, each verified by tests.

## Inputs

- `args`: target file, symbol, or short description ("split UserService").

## Steps

1. **Read the target** and immediate neighbours. Note public surface and
   any tests covering it.
2. **Delegate the impact map** to a `refactor-analyst` subagent (see
   `## Delegation`). Wait for its report before planning.
3. **Plan**: decompose into 2–6 atomic steps. Each step must:
   - keep the public API behavior unchanged,
   - be independently revertable,
   - have a clear "tests still green" check.
4. **For each step**:
   1. apply the edit,
   2. run focused tests (`shell` — pick the narrowest test command),
   3. if green → continue; if red → revert and re-plan.
5. After the last step — full test run, then summary of what changed.

## Delegation

One subagent, **named** `refactor-analyst`, **read-only**:

| Why isolated | Why not parallel |
|---|---|
| Impact analysis (call-sites, transitive imports, tests touching the symbol) is large in tokens but produces a small report. Doing it in a separate context keeps the main session focused on edits. | The actual refactor is sequential — each step depends on the previous file state. Parallelizing edits would race on the same files. |

Send the analyst:
- target symbol/file,
- list of test directories,
- request: "return JSON with `callers[]`, `tests[]`, `risk_notes[]`".

Skip delegation only if the impact is obviously local (single file, no
external imports, < 50 lines).

## Output

```
## Refactor plan: <target>

Impact (from refactor-analyst):
- callers: N files, K sites
- covering tests: ...
- risks: ...

Steps:
1. ...
2. ...

Execution:
- step 1: applied, tests green (npm test -- src/foo)
- step 2: ...
```

## Notes

- Never combine refactor with feature changes in the same skill run.
- If a step turns out to need a behavior change — stop, surface it, ask
  the user to confirm before proceeding.
- Commit after each green step (or at least at safe checkpoints) so a
  bad step can be rolled back individually.
