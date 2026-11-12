---
name: delegate
description: Splits a large task into independent subtasks, runs each in its own sub-agent (in parallel where possible), verifies the combined result, and compiles one recap covering what got done, what didn't, blocking decisions with a suggestion for each, assumptions taken, and next actions. Use when the user runs /delegate or explicitly asks to break work into sub-agents or run it in parallel.
when_to_use: 'User says things like "pecah task jadi sub-agent", "kerjakan paralel", "fan out", or "bagi ke beberapa agent", or hands over a multi-part task they want delegated.'
argument-hint: <task description>
allowed-tools: Bash(git branch:*), Bash(git status:*), Bash(git rev-parse:*)
---

# Delegate: split → run sub-agents → verify → recap

Task: $ARGUMENTS
(If empty, use the task from the user's latest message.)

Starting state (baseline for step 5; files listed here were already changed **before** this run, so don't attribute them to sub-agents):
- Branch: !`git branch --show-current 2>/dev/null || echo "(not a git repo)"`
- HEAD: !`git rev-parse --short HEAD 2>/dev/null || echo "-"`
- Already changed: !`git status --short 2>/dev/null | head -50 || true`

Copy this checklist into your working notes and tick items off as you go:
```
- [ ] 1 Gate: is splitting worth it?
- [ ] 2 Plan written (subtasks, waves, scope)
- [ ] 3 Briefs sent (one message per wave)
- [ ] 4 Results collected; gap check done
- [ ] 5 Combined result verified
- [ ] 6 Recap delivered
```

## 1. Gate: is splitting worth it?
Sub-agents cost a lot. Multi-agent runs use roughly 15× the tokens of a chat, and every brief loses some context. Scale the run to the task:

| Task shape | Agents |
|---|---|
| Small, or each step needs the previous result | **0**: do it yourself |
| 2–3 clearly independent parts | 2–3 |
| Broad (many modules or sources, independent angles) | 3–5 per wave, max 2 waves |

Don't split when the parts share heavy context or edit the same files. Coding in one tight area is usually a poor fit, while independent modules, research angles, or a separate review pass are a good fit. If you don't split, say why in one line, do the work yourself, and still finish with the recap in step 6.

For runs bigger than 5 agents per wave, suggest the Workflow tool instead and ask the user first.

## 2. Plan
Read just enough context to find the real seams, without doing the work. For each subtask, define:
- **Goal**: one sentence.
- **Scope**: the files or dirs it may **write**. No two subtasks write the same file, and generated or shared files (lockfiles, migrations index, routes registry) go to exactly one owner.
- **Done when**: checkable criteria.
- **Wave**: if B needs A's output, B goes in a later wave.
- **Agent**:
  - `Explore` for read-only lookup (add `effort: "lower"` if mechanical). Explore skips the project's CLAUDE.md, so copy any relevant project rules into its brief.
  - `general-purpose` for changes.
  - A project-specific agent when its description matches exactly.

Pick a run directory: the session scratchpad directory if the system prompt lists one, otherwise `${TMPDIR:-/tmp}/delegate-<YYYYMMDD-HHMM>/`. Write the plan there as `delegate-plan.md` so it survives context compaction. Sub-agents' long outputs go to the same directory. Show it as a short table (subtask | agent | wave | writes). Then **start right away**. Ask first only if an ambiguity changes the split, or a step is destructive or outward-facing.

Worktree caveat: `isolation: "worktree"` branches from the **default branch**, not the current HEAD. Use it only when the subtask doesn't need uncommitted or feature-branch work. Otherwise keep the write scopes disjoint in the shared tree.

## 3. Brief each sub-agent
Sub-agents see none of this conversation, and they **cannot ask the user questions**. The brief is all they have. Use [brief-template.md](brief-template.md) and fill in every field. In particular:
- Context the agent needs, plus what's already ruled out.
- Effort budget: stop as soon as done-criteria are met, and don't keep searching for things that may not exist.
- Decisions: if the choice is reversible and low-risk, take the safest default and record it as an **assumption**. If it isn't, stop that part, continue the rest, and report it as a **blocking decision**.
- Long output (logs, findings > ~40 lines) goes to a file in the run directory. The report links it instead of pasting it.
- Helper files (scratch scripts, raw API/data dumps) are allowed only in the run directory, prefixed with the subtask id (`A-*`, `B-*`). Raw dumps stay there and are never copied into the repo; in step 5 they double as evidence for spot-checks.
- Don't spawn further sub-agents.
- **No git state changes.** Agents share one working tree, so `git stash`, `checkout`, `switch`, `reset`, `restore`, `clean`, `rebase`, `commit`, or `push` could wipe another agent's work or the user's own uncommitted changes. Only read-only git commands (`status`, `diff`, `log`, `show`) are allowed.
- **Scoped tests only.** Agents run only the tests or checks for their own files (e.g. `go test ./pkg/x/...`, a single test file). No full-suite runs, builds, or dev servers, because parallel runs collide on caches, ports, and test DBs. The full run happens once in step 5.
- For code-changing subtasks, keep the template's **Code rules** block (surgical, minimal, test-first). Its "Done when" should name a test or check that must pass.

## 4. Run and collect
- Send all briefs of a wave **in one message** so they run concurrently. They run in the background, and you're notified on completion. Don't poll, and don't predict results.
- Background agents raise their permission prompts in the main session, labeled with the agent's name. Outside auto mode, warn the user in one line that several prompts may appear at once.
- **Stuck agent:** if every other agent in the wave has finished and one is still running long after, or is looping, stop it with `TaskStop` and record its subtask under "Belum dikerjakan" with the reason.
- **Gap check** after each wave: are the done-criteria met? Did anything come back off-target, thin, or contradictory? Fix it by continuing that same agent with `SendMessage` (it keeps its context), by doing the part yourself, or by adding one more subtask to the next wave. Don't run more than one extra round. After that, report what's left as not done.
- `SendMessage` and `TaskStop` may be deferred tools. If their schema isn't loaded, load it first with `ToolSearch` (`select:SendMessage,TaskStop`).

## 5. Verify the combined result
Reports can sound confident and still be wrong.
- Compare `git status` / `git diff` against the starting state: HEAD must be unchanged, and the changed files should be the "Already changed" list plus the planned write scopes. Confirm every claimed change exists, and flag anything outside the scopes. For files that were already dirty before the run, read the diff carefully to separate the user's edits from the agents' edits.
- Run the full relevant test, build, or lint once over the **combined** result. This is the only full run.
- For high-stakes output (many files, security, data migration), optionally run one independent read-only reviewer agent over the diff. Give it the code and the done-criteria, not your conclusions.
- Resolve conflicts between agents, or list them as blocking decisions.

## 6. Recap
Use exactly this structure, leaving out any empty section. Write it in the **user's language**: the headings below are Indonesian, so translate them (e.g. Summary / Done / Not done / Decisions needed / Assumptions / Verification / Next actions) when the user writes in another language.

```
## Ringkasan
<1–3 sentences. Status: ✅ selesai / 🟡 sebagian / ⛔ terblokir. Agents used: N in M waves.>

## Sudah dikerjakan
| # | Subtask | Hasil | File / bukti |
|---|---|---|---|

## Belum dikerjakan
- <item>: <reason: blocked by decision #k / failed (error) / out of scope>

## Keputusan yang dibutuhkan (blocking)
1. **<question>**
   - Opsi: A) … B) …
   - Saran: **<option>**, because <reason + consequence of each choice>
   - Memblokir: <which work waits on this>

## Asumsi yang diambil
- <assumption>: <why it was safe; how to reverse it if wrong>

## Verifikasi
- <command> → <pass/fail + key output>; say plainly if something wasn't run

## Next action
1. [<user|Claude>] <concrete step>, in priority order, 1–5 items
```

Rules:
- Report honestly. Failed counts as failed, and skipped gets named as skipped.
- Every blocking decision ends with one concrete suggestion.
- Put next actions that unblock the most work first.

If anything didn't go as planned, see [examples.md](examples.md) for good and bad splits.
