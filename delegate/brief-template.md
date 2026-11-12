# Sub-agent brief template

Fill in every field, and write "none" instead of deleting one.

```
Goal: <one sentence>

Context:
- Why: <the larger task and where this piece fits>
- Read first: <paths, docs, specs>
- Already known / ruled out: <facts so the agent doesn't redo them>

Scope:
- You may WRITE only: <paths>. Read anything you need.
- Helper files (scratch scripts, raw API/data dumps) are allowed ONLY inside <run-dir>, named `<subtask-id>-*` (e.g. `C-items.json`). Never copy raw dumps anywhere else, and list them under Files changed.
- Do not: delete files, call external services, or spawn sub-agents<, plus task-specific limits>.
- Git: read-only commands only (status, diff, log, show). Never stash, checkout, switch, reset, restore, clean, rebase, commit, or push. Other agents and the user share this working tree.
- Tests: run only the tests/checks for your own files (<scoped command>). No full-suite runs, builds, or dev servers; the lead runs those once at the end.

Done when: <checkable criteria>
Budget: stop as soon as the done criteria are met. If something can't be found after a reasonable search, report it as not found instead of searching on.

Code rules (only for subtasks that change code; delete this block otherwise):
- Every changed line must trace to this subtask's goal. Don't refactor, reformat, or "improve" adjacent code. Mention unrelated issues in the report instead.
- Match existing style. Write the minimum code that meets "Done when", with no speculative features or abstractions.
- Remove only the imports/vars that YOUR change made unused.
- Prefer a verifiable goal: write or identify the test first, then make it pass.

Decisions: you cannot ask the user.
- Reversible and low-risk: pick the safest default and list it under Assumptions.
- Otherwise: stop that part, finish the rest, and list it under Blocking decisions with options and your recommendation.

Long output (more than ~40 lines of logs or findings): write it to <run-dir>/<subtask-id>.md and link the file.

Final report, under 300 words, using exactly these headings:
### Done
- <item> (<file path>)
### Not done
- <item>: <why>
### Blocking decisions
- <question> | options: A / B | recommendation: <choice + why>
### Assumptions
- <assumption>: <why it was safe>
### Verification
- <command> → <result>, or "not verified: <why>"
### Files changed
- <path>: <one-line summary>
```
