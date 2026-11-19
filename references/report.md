# Stage: report

Goal: tell the user what actually happened, from evidence rather than memory. Use a fresh context.

## Read

PLAN.md, TASK.md, the code diff for the task, DEVIATIONS.md, PROGRESS.md (gotchas and History, including rejected MVPs), TEST-RESULTS.md. Not the implementation chat.

## Write

**1. CHANGELOG entry.** Add to the repo's CHANGELOG.md (create it if missing, follow its existing style): date, what changed from the user's/consumer's point of view, breaking changes called out.

**2. NOTES.md** in the task folder:

```markdown
# Notes: <task>
## What was done
Short summary mapped to the plan's tasks.
## Decisions
Key decisions and why.
## Deviations
From DEVIATIONS.md, with impact.
## Risks and things to watch
What could bite someone: edge cases, migrations, security, performance, operational concerns.
## Not tested
From TEST-RESULTS.md: gaps and wider-impact risks not run.

## Improvements and next steps
### Improvements
Up to ~5, ranked. Each: what, why it matters (performance, security, cost, maintainability, DX), effort (S/M/L), priority (H/M/L). Mark correctness or security concerns clearly so they aren't buried.
### Tech debt / shortcuts
Where "good enough for now" was chosen, and the trade-off.
### Next steps
3-5 ordered follow-ups, each worded so it can be pasted as the goal of a new `/justdev plan`. Note dependencies ("do A before B").
```

**3. Result in TASK.md.** Under each sub-task block, add (or replace) a `## Result` with one line, in the same business wording as the Outcome: what was delivered, and plainly what is partial or not done.

```markdown
# 01 – <Heading>
## Outcome
- <unchanged>
## Result
- <what is now true for the user/business; note anything partial or not delivered>
```

Don't change the Outcome lines; they're the plan, the Result is what happened. Add a History line ("TASK.md results added").

## Rules

- Be specific and grounded in what you saw in the code, tests and deviations. "Add an index on `withdrawals.user_id` because the new query filters on it" is useful; "improve performance" is not.
- Don't pad. If there is nothing real to say under a heading, say so in one line.
- Don't hide bad news: failing or skipped tests, unresolved deviations and unverified claims belong in the report.

Set the status in PROGRESS.md to `report · in progress` when you start and, when done, to `report · complete · next: integrate` (or `done` if other teams don't consume this work). Suggest `/justdev integrate` if they do.
