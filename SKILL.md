---
name: justdev
description: Structured multi-stage dev workflow for large backend/blockchain tasks, where files (not chat) carry the memory. Stages - plan, review, implement, test, report, integrate, amend. Use whenever the user types /justdev, or asks to plan a big task then have agents implement, test and document it, or mentions a task folder with PLAN.md / PROGRESS.md, or wants to amend an in-flight plan, even if they don't say "justdev".
---

# justdev

A workflow for large tasks. The core problem it solves: long chats lose details. So every stage writes its result to a file in the task folder, and the next stage starts with a fresh context and reads only those files. Nothing important should live only in chat.

## Usage

`/justdev <stage> [task-folder-or-description]`

| Stage | What it does | Read |
|---|---|---|
| `plan` | Interview the user, write PLAN.md (new task folder) | [references/plan.md](references/plan.md) |
| `review` | Independent review of PLAN.md, fix conflicts, user approves | [references/review.md](references/review.md) |
| `implement` | Build in slices, checkpoint in PROGRESS.md | [references/implement.md](references/implement.md) |
| `test` | Run tests scoped to what changed | [references/test.md](references/test.md) |
| `report` | CHANGELOG + NOTES.md + improvements and next steps | [references/report.md](references/report.md) |
| `integrate` | Short guide for frontend / integrating teams (MD + HTML) | [references/integrate.md](references/integrate.md) |
| `amend` | Change an in-flight plan without restarting | [references/amend.md](references/amend.md) |

Read only the reference file for the stage being run. If no stage is given, ask which one, or suggest the next one based on which files exist in the task folder.

## Task folder

Each task lives in `tasks/<YYYY-MM-DD-slug>/` at the repo root:

```
PLAN.md          source of truth: what to build (current state only)
PROGRESS.md      checkpoint: done / in progress / next / gotchas
DEVIATIONS.md    where implementation departed from the plan, and why
TEST-RESULTS.md  what was run and what happened
NOTES.md         report: decisions, risks, not-tested, improvements, next steps
INTEGRATION.md   guide for integrating agents
INTEGRATION.html same guide for humans
```

Find the current task from the argument; if none is given, use the most recently modified folder under `tasks/` and tell the user which one you picked. Create files only when their stage runs.

## Principles (why the stages are shaped this way)

- **Files carry memory.** A fresh agent that reads PLAN.md and PROGRESS.md can continue the work with no chat history. Write state down at the moment it happens, because agents recall it poorly later.
- **Fresh context per stage.** Review, test, report and integrate should not inherit the reasoning of the stage before. A reviewer that saw the planner's thinking tends to agree with it. Use a subagent for these stages when available; otherwise tell the user to run `/clear` first. Plan is interactive, so it runs in the main session. Implement can run in the main session or a subagent.
- **One place per fact.** PLAN.md holds the current truth. When something changes, rewrite the affected text and delete what it replaces. Duplicated or superseded text is what creates conflicts.
- **Scoped by default.** Tests cover only what the diff touches. The full suite runs only when the user explicitly asks.
- **Short beats complete.** Every document here is read by someone busy. Specific and brief wins over thorough.
- **Don't go beyond the stage.** Each stage stops when its output is written. Suggest the next stage and let the user trigger it, since the user wants control at the boundaries (especially after review).

## Project test map

Stage `test` needs to know how to run scoped tests. Look for a "test map" in the repo's CLAUDE.md (module path to test command, e.g. `cargo test -p <crate>`, `jest --findRelatedTests`, `pytest <path>`). If it's missing, infer one from the repo's tooling, show it to the user, and offer to save it to CLAUDE.md.
