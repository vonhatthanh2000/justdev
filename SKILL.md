---
name: justdev
description: Structured multi-stage dev workflow for large backend/blockchain tasks, where files (not chat) carry the memory. Work is split into small sub-tasks; each one is built as an MVP first, stopped at a checkpoint for the user to confirm the direction, then hardened. Stages - plan, review, implement (MVP), harden, test, report, integrate, amend. Use whenever the user types /justdev, or asks to plan a big task then have agents implement, test and document it, or mentions a task folder with PLAN.md / PROGRESS.md / TASK.md, or confirms or rejects an MVP checkpoint, or wants to amend an in-flight plan, even if they don't say "justdev".
---

# justdev

A workflow for large tasks. It solves two problems:

1. **Long chats lose details.** Every stage writes its result to a file in the task folder, and the next stage starts with a fresh context and reads only those files. Nothing important should live only in chat.
2. **Big builds go the wrong way unseen.** Work is split into small sub-tasks. Each one is first built as an MVP (the simplest version that runs end to end), then the agent stops so the user can confirm the direction. Only after "confirm" does the agent harden it (interfaces, modules, spec files, edge cases).

## Usage

`/justdev <stage> [task-folder-or-description]`

| Stage | What it does | Read |
|---|---|---|
| `plan` | Interview the user, write PLAN.md (MVP + Hardening backlog per sub-task) and TASK.md | [references/plan.md](references/plan.md) |
| `review` | Independent review of PLAN.md, fix conflicts, user approves | [references/review.md](references/review.md) |
| `implement` | Build the **MVP only** of the next sub-task, then stop at the checkpoint | [references/implement.md](references/implement.md) |
| `harden` | After the user confirms the MVP: build the sub-task's Hardening backlog | [references/harden.md](references/harden.md) |
| `test` | Run tests scoped to what changed, including the spec files `harden` wrote | [references/test.md](references/test.md) |
| `report` | CHANGELOG + NOTES.md + improvements and next steps + Result lines in TASK.md | [references/report.md](references/report.md) |
| `integrate` | Short guide for frontend / integrating teams (MD + HTML) | [references/integrate.md](references/integrate.md) |
| `amend` | Change an in-flight plan without restarting; handles a rejected MVP | [references/amend.md](references/amend.md) |

Read only the reference file for the stage being run. If no stage is given, read the `Status:` line in the task's PROGRESS.md, tell the user which phase the task is in and what the next stage is, and ask which to run. If PROGRESS.md doesn't exist, infer the phase from which files are in the task folder (see "Status and history").

**Checkpoint replies.** If the status is `implement <NN> · awaiting user` and the user replies "confirm", follow "User's reply" in [references/implement.md](references/implement.md). If they say the MVP is wrong, follow "Rejected MVP" in [references/amend.md](references/amend.md).

## Flow

```
plan → review → for each sub-task 01, 02, …:
                  implement (MVP) → checkpoint → harden
                                     └─ rejected → amend → implement (MVP v2) → checkpoint
              → test → report → integrate
```

- There is one checkpoint per sub-task, after its MVP. Harden slices don't stop, except on drift.
- After two rejected MVPs for the same sub-task, stop patching and go back to the `plan` interview (see amend.md).
- `test`, `report` and `integrate` run once, after every sub-task is hardened.

## Task folder

Each task lives in `tasks/<YYYY-MM-DD-slug>/` at the repo root:

```
PLAN.md          source of truth: what to build (current state only), split into sub-tasks
TASK.md          for humans: business-worded outcome per sub-task, copied by hand into the company dashboard
PIVOTAL.md       for humans: pivotal choices (function, effect, trade-offs, scenarios). Agents build from PLAN.md, not this
PROGRESS.md      status line, sub-task states, checkpoint, done / in progress / next / gotchas, history of big changes
DEVIATIONS.md    where implementation departed from the plan, and why
TEST-RESULTS.md  what was run and what happened
NOTES.md         report: decisions, risks, not-tested, improvements, next steps
INTEGRATION.md   guide for integrating agents
INTEGRATION.html same guide for humans
```

Find the current task from the argument; if none is given, use the most recently modified folder under `tasks/` and tell the user which one you picked. Create files only when their stage runs, except PROGRESS.md and TASK.md, which the plan stage creates.

### File ownership

| File | Reader | Written by |
|---|---|---|
| PLAN.md | AI | plan, review (fixes), amend; implement and harden only tick checkboxes |
| TASK.md | the user / their leader | plan, amend, report |
| PROGRESS.md | AI + user | plan (creates), implement, harden, amend; every stage updates the status line and History |
| DEVIATIONS.md | AI + user | implement, harden, amend, test (behavior-changing fixes) |

**`implement` and `harden` never read TASK.md** (or PIVOTAL.md). PLAN.md is their only spec. TASK.md is business wording for humans; nothing may exist only there.

## Status and history

Both live in PROGRESS.md, so one file answers "where are we, and how did we get here?" for a fresh agent or the user.

**Status line**: the first line under the title, always kept current:

```
Status: <stage>[ <NN>] · <state> · next: <stage>[ <NN>] · updated <YYYY-MM-DD>
```

- `<stage>` is one of `plan`, `review`, `implement`, `harden`, `test`, `report`, `integrate`, `done`. `implement` and `harden` carry the sub-task number, e.g. `implement 02`.
- `<state>` is one of `in progress`, `awaiting user` (plan waiting for approval, or an MVP waiting at its checkpoint), `blocked` (say on what), `complete`.
- Every stage sets it when it starts (`in progress`) and again when it ends (`complete`, with `next:`). Moving backwards counts: an amend, a rejected MVP or failing tests set the stage back, because the status must say where the work really is, not where it last got to.
- If the line is missing, infer the phase from the files: no PROGRESS.md means plan; PLAN.md without approval means review; then take the first sub-task that isn't hardened: unchecked MVP tasks in PLAN.md mean `implement <NN>`; MVP done but no "MVP vN confirmed" History line means `implement <NN> · awaiting user`; confirmed but unchecked backlog items mean `harden <NN>`. All sub-tasks hardened without TEST-RESULTS.md means test; TEST-RESULTS.md without NOTES.md means report; NOTES.md without INTEGRATION.md means integrate. Say it's an inference, and write the status line.

**History**: a `## History` section at the bottom of PROGRESS.md. Append one line per big event, never rewrite or delete old lines (this is the one place old state is kept on purpose):

```
YYYY-MM-DD · <stage> · <what happened>, and why if it isn't obvious
```

Log these:
- the plan was written, and the user approved it (end of review)
- a pivotal choice was confirmed or changed
- PLAN.md or TASK.md was updated, by any stage and however small: what changed and why (edits made in one pass can share a line)
- the review verdict, when it found something blocking
- an MVP was confirmed (`MVP v1 confirmed (01)`) or rejected (`MVP v1 rejected (01): <short reason>; fix forward | revert`)
- a sub-task was escalated back to plan after two rejected MVPs
- tests failed and work returned to harden
- the Goal or scope changed
- a significant deviation (point to its DEVIATIONS.md entry instead of repeating it)

Don't log routine progress (that is the Done list). Edits to other files don't need a line; edits to PLAN.md and TASK.md always do, except ticking a checkbox, which is routine progress. One line each, with the why mattering more than the what, since git already has the diff. When a stage rewrites the rest of PROGRESS.md, leave History untouched.

## Principles (why the stages are shaped this way)

- **Files carry memory.** A fresh agent that reads PLAN.md and PROGRESS.md can continue the work with no chat history. Write state down at the moment it happens, because agents recall it poorly later.
- **MVP first, structure later.** Prove the behavior end to end before building interfaces, modules and spec files around it. Structure built on the wrong behavior is the most expensive rework. The Hardening backlog in PLAN.md lists what is deferred, which is what makes skipping it safe.
- **Small sub-tasks, one checkpoint each.** An MVP should take about 10–15 minutes of agent work. The user sees it running before anything is hardened.
- **Stop on drift, don't improvise.** When the code or the situation contradicts the plan, stop and ask. A silent workaround is a direction change the user never approved.
- **Fresh context per stage.** Review, test, report and integrate should not inherit the reasoning of the stage before. A reviewer that saw the planner's thinking tends to agree with it. A fresh session or `/clear` is enough; a subagent is optional, never required. Plan is interactive, so it needs the user in the loop. Questions to the user belong in the plan stage, before PLAN.md is finished; later stages work from the plan and don't re-interview.
- **Work from the local code.** Read the working tree as it is now, committed and uncommitted, and never assume code is pushed. Don't fetch, pull or push unless the user asks.
- **Never commit.** No stage runs `git commit`, `git add`, `git stash`, `git reset`, `git checkout` over changes, or anything else that alters git history, the index or the working tree's tracked state. The user does all committing manually. Read-only git (`status`, `diff`, `log`, `rev-parse`) is fine. Reverting a rejected MVP is done by editing files, never by git.
- **For large tasks only.** This workflow is for work big enough to lose context. For small or one-file changes, skip it; the ceremony costs more than it saves.
- **One place per fact.** PLAN.md holds the current truth. When something changes, rewrite the affected text and delete what it replaces. Duplicated or superseded text is what creates conflicts. The change history lives only in PROGRESS.md's History section, never inside PLAN.md. The one exception is a sub-task's "Rejected attempts" lines, which stay because they tell the next MVP attempt what not to repeat.
- **Scoped by default.** Tests cover only what the diff touches. The full suite runs only when the user explicitly asks.
- **Short beats complete.** Every document here is read by someone busy. Specific and brief wins over thorough.
- **Don't go beyond the stage.** Each stage stops when its output is written. Suggest the next stage and let the user trigger it, since the user wants control at the boundaries (especially after review and at each checkpoint).

## Project test map

Stage `test` needs to know how to run scoped tests. Look for a "test map" in the repo's CLAUDE.md (module path to test command, e.g. `cargo test -p <crate>`, `jest --findRelatedTests`, `pytest <path>`). If it's missing, infer one from the repo's tooling, show it to the user, and offer to save it to CLAUDE.md.
