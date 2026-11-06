# Stage: implement

Goal: build what PLAN.md says, in a way that survives a context reset at any moment.

## Start

Read PLAN.md and PROGRESS.md (if it exists) and nothing else from earlier work. If PROGRESS.md exists, resume from "next"; don't redo finished tasks. If the working tree has uncommitted changes you didn't make, tell the user before touching it.

## Work in slices

Take the tasks in order. A slice is a small group of tasks that leaves the code in a working state. For each slice:

1. Implement it.
2. Do a quick sanity check (build/typecheck/the nearest test), not the whole test stage.
3. Update PROGRESS.md.
4. Commit with a message that names the slice.

Committing per slice gives a second checkpoint in git and makes amendments and rollbacks cheap.

## PROGRESS.md

Update it after every task, not at the end. A context that is already overloaded can't be trusted to save state later. Keep it short:

```markdown
# Progress
## Done
- [x] Task 3: <what> (commit abc123)
## In progress
- Task 4: <current state, what's left>
## Next
- Task 5, 6
## Gotchas
- Things the next agent must know (surprising behavior, workarounds, open questions)
```

## DEVIATIONS.md

The moment you do something the plan didn't say or contradicts it, append an entry: what, why, and impact. Write it immediately; recalling it later is unreliable, and the report and integration stages depend on it.

- Small implementation detail the plan left open: log it, carry on.
- A change that alters a Decision, acceptance criterion, or interface: stop and use `/justdev amend` instead of silently diverging.

## Rules

- Stay inside the plan's scope. Notice-but-don't-fix unrelated problems; put them in PROGRESS.md gotchas for the report.
- If the context is getting long, make sure PROGRESS.md is current and tell the user they can `/clear` and re-run `/justdev implement`. The files are enough to continue.
- When all tasks are done, say so, and suggest `/justdev test`.
