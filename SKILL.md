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

Read only the reference file for the stage being run. If no stage is given, read the `Status:` line in the task's PROGRESS.md, tell the user which phase the task is in and what the next stage is, and ask which to run. If PROGRESS.md doesn't exist, infer the phase from which files are in the task folder (see "Status and history").

## Task folder

Each task lives in `tasks/<YYYY-MM-DD-slug>/` at the repo root:

```
PLAN.md          source of truth: what to build (current state only)
PIVOTAL.md       for humans: pivotal choices (function, effect, trade-offs, scenarios). Agents build from PLAN.md, not this
PROGRESS.md      status line, checkpoint (done / in progress / next / gotchas), history of big changes
DEVIATIONS.md    where implementation departed from the plan, and why
TEST-RESULTS.md  what was run and what happened
NOTES.md         report: decisions, risks, not-tested, improvements, next steps
INTEGRATION.md   guide for integrating agents
INTEGRATION.html same guide for humans
```

Find the current task from the argument; if none is given, use the most recently modified folder under `tasks/` and tell the user which one you picked. Create files only when their stage runs, except PROGRESS.md, which the plan stage creates so that status and history exist from the first moment.

## Status and history

Both live in PROGRESS.md, so one file answers "where are we, and how did we get here?" for a fresh agent or the user.

**Status line**: the first line under the title, always kept current:

```
Status: <stage> · <state> · next: <stage> · updated <YYYY-MM-DD>
```

- `<stage>` is one of `plan`, `review`, `implement`, `test`, `report`, `integrate`, `done`.
- `<state>` is one of `in progress`, `awaiting user` (e.g. plan waiting for approval), `blocked` (say on what), `complete`.
- Every stage sets it when it starts (`in progress`) and again when it ends (`complete`, with `next:`). Moving backwards counts: an amend or failing tests sets the stage back (e.g. `implement · in progress`), because the status must say where the work really is, not where it last got to.
- If the line is missing, infer the phase from the files: no PROGRESS.md means plan; PLAN.md without approval means review; unchecked tasks mean implement; all tasks done without TEST-RESULTS.md means test; TEST-RESULTS.md without NOTES.md means report; NOTES.md without INTEGRATION.md means integrate. Say it's an inference, and write the status line.

**History**: a `## History` section at the bottom of PROGRESS.md. Append one line per big event, never rewrite or delete old lines (this is the one place old state is kept on purpose):

```
YYYY-MM-DD · <stage> · <what happened>, and why if it isn't obvious
```

Log these:
- the plan was written, and the user approved it (end of review)
- a pivotal choice was confirmed or changed
- PLAN.md was updated, by any stage and however small: what changed and why (edits made in one pass can share a line)
- the review verdict, when it found something blocking
- tests failed and work returned to implement
- the Goal or scope changed
- a significant deviation (point to its DEVIATIONS.md entry instead of repeating it)

Don't log routine progress (that is the Done list). Edits to other files don't need a line; edits to PLAN.md always do. One line each, with the why mattering more than the what, since git already has the diff. When a stage rewrites the rest of PROGRESS.md, leave History untouched.

## Principles (why the stages are shaped this way)

- **Files carry memory.** A fresh agent that reads PLAN.md and PROGRESS.md can continue the work with no chat history. Write state down at the moment it happens, because agents recall it poorly later.
- **Fresh context per stage.** Review, test, report and integrate should not inherit the reasoning of the stage before. A reviewer that saw the planner's thinking tends to agree with it. A fresh session or `/clear` is enough; a subagent is optional, never required. Plan is interactive, so it needs the user in the loop. Questions to the user belong in the plan stage, before PLAN.md is finished; later stages work from the plan and don't re-interview.
- **Work from the local code.** Read the working tree as it is now, committed and uncommitted, and never assume code is pushed. Don't fetch, pull or push unless the user asks.
- **Never commit.** No stage runs `git commit`, `git add`, `git stash`, `git reset`, `git checkout` over changes, or anything else that alters git history, the index or the working tree's tracked state. The user does all committing manually. Read-only git (`status`, `diff`, `log`, `rev-parse`) is fine.
- **For large tasks only.** This workflow is for work big enough to lose context. For small or one-file changes, skip it; the ceremony costs more than it saves.
- **One place per fact.** PLAN.md holds the current truth. When something changes, rewrite the affected text and delete what it replaces. Duplicated or superseded text is what creates conflicts. The change history lives only in PROGRESS.md's History section, never inside PLAN.md.
- **Scoped by default.** Tests cover only what the diff touches. The full suite runs only when the user explicitly asks.
- **Short beats complete.** Every document here is read by someone busy. Specific and brief wins over thorough.
- **Don't go beyond the stage.** Each stage stops when its output is written. Suggest the next stage and let the user trigger it, since the user wants control at the boundaries (especially after review).

## Project test map

Stage `test` needs to know how to run scoped tests. Look for a "test map" in the repo's CLAUDE.md (module path to test command, e.g. `cargo test -p <crate>`, `jest --findRelatedTests`, `pytest <path>`). If it's missing, infer one from the repo's tooling, show it to the user, and offer to save it to CLAUDE.md.
