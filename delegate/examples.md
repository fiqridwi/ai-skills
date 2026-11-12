# Split examples

## Good: independent seams
Task: "Add an export-to-CSV feature to the Reports module: BE endpoint, FE button, and docs."
| # | Subtask | Agent | Wave | Writes |
|---|---|---|---|---|
| 1 | Define the CSV contract (columns, filters) from the spec | Explore | 1 | scratchpad/contract.md |
| 2 | BE endpoint + test | general-purpose | 2 | backend/reports/export* |
| 3 | FE button + download hook | general-purpose | 2 | frontend/src/reports/Export* |
| 4 | Docs | general-purpose | 2 | docs/reports.md |
Why it works: the shared contract is settled first in wave 1, and the wave-2 writers never touch the same files.

## Good: parallel research angles
Task: "Compare 3 queue libraries for our Go service."
One Explore agent per library, each with the same rubric (throughput, ops burden, license, fit with our code). The lead compares the three answers and makes the recommendation.

## Bad: same-file fan-out
Task: "Refactor `service.go` and fix its 3 bugs."
Three agents would all edit `service.go` and conflict. → Don't split. Do it yourself (0 agents) and still give the recap.

## Bad: over-splitting
Task: "Rename a config key in 4 files."
This is a handful of edits. Spawning 4 agents costs far more than it saves. → 0 agents.

## Bad: vague brief
"Look into the auth problem." The agent doesn't know which problem, where to look, or when it's done, so it will guess or duplicate another agent's work.
Fix: "Find why `/api/login` returns 500 on staging when the email has uppercase letters. Read `auth/handler.go` and `auth/user_repo.go`. Done when you name the root-cause line and a proposed fix. Don't edit."
