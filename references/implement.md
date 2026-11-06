# Stage: implement

Goal: build what PLAN.md says, in a way that survives a context reset at any moment.

## Start

Read PLAN.md and PROGRESS.md (if it exists) and nothing else from earlier work. PLAN.md is the whole spec; don't build from PIVOTAL.md, which is a human decision record. Decisions in PLAN.md are fixed, so don't re-open them. If PROGRESS.md exists, resume from "next"; don't redo finished tasks. If the working tree has uncommitted changes you didn't make, tell the user before touching it.

**Record the base commit.** If PROGRESS.md has no `Base:` line, add one directly under the status line the first time you start: `Base: <output of git rev-parse HEAD>` (before you change anything). The test stage diffs against it, so keep it as is. On a resume, never overwrite it.

## Work in slices

Take the tasks in order. A slice is a small group of tasks that leaves the code in a working state. For each slice:

1. Implement it.
2. Do a quick sanity check (build/typecheck/the nearest test), not the whole test stage.
3. Update PROGRESS.md.
4. Stop there. **Don't run `git commit`, `git add`, or anything that changes git history or the index; the user commits manually.** Leave the changes in the working tree.

Without commits, PROGRESS.md is the checkpoint, so keep it accurate after every task. At a sensible slice boundary you may suggest the user commit if they want a restore point, but don't do it for them.

## PROGRESS.md

Update it after every task, not at the end. A context that is already overloaded can't be trusted to save state later. Keep it short:

```markdown
# Progress
Status: implement · in progress · next: test · updated 2026-11-06
Base: <commit sha before any changes>
## Done
- [x] Task 3: <what> (files touched)
## In progress
- Task 4: <current state, what's left>
## Next
- Task 5, 6
## Gotchas
- Things the next agent must know (surprising behavior, workarounds, open questions)
## History
(existing lines stay exactly as they are; see SKILL.md)
```

Set the status to `implement · in progress` when you start (including when resuming after an amend or failed tests) and refresh the date as you go. Never rewrite or drop History lines when you update the rest of the file. Add one when a significant deviation happens, pointing at its DEVIATIONS.md entry.

## DEVIATIONS.md

The moment you do something the plan didn't say or contradicts it, append an entry: what, why, and impact. Write it immediately; recalling it later is unreliable, and the report and integration stages depend on it.

- Small implementation detail the plan left open: log it, carry on.
- A change that alters a Decision, acceptance criterion, or interface: stop and use `/justdev amend` instead of silently diverging.

## Rules

- Stay inside the plan's scope. Notice-but-don't-fix unrelated problems; put them in PROGRESS.md gotchas for the report.
- If the context is getting long, make sure PROGRESS.md is current and tell the user they can `/clear` and re-run `/justdev implement`. The files are enough to continue.
- When all tasks are done, set the status to `implement · complete · next: test`, say so, and suggest `/justdev test`.
