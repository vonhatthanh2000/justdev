# Stage: plan

Goal: turn the user's description into a PLAN.md that a fresh agent can implement without asking anything. This stage is interactive and runs in the main session.

## Steps

1. Create `tasks/<YYYY-MM-DD-slug>/` (slug: short, kebab-case, from the goal).
2. Explore the relevant code first so your questions are informed, not generic.
3. Interview the user until nothing is ambiguous. Ask the questions that change what gets built: scope edges, data formats, failure behavior, security or auth, compatibility, what must not change. Group questions, don't drip them one at a time, and propose a default for each so the user can just say "yes". Challenge assumptions that look risky.
4. Write PLAN.md using the template below. Stop and show the user. Don't start implementing.

## Template

```markdown
# <Task title>

## Goal
One or two sentences: the outcome, not the method.

## Non-goals
What this task deliberately does not do.

## Decisions
Final answers only, one line each, with a short reason. No debate history.

## Constraints
Hard limits: compatibility, performance, security, deadlines, things not to touch.

## Tasks
- [ ] 1. Small, ordered, each independently verifiable
- [ ] 2. ...
(Group into slices if the work is large; each slice should fit in one context.)

## Acceptance criteria
Testable statements. "Withdrawing above the daily cap returns error X", not "handles limits well".

## Files / modules expected to change
Paths or modules, used later to scope tests.

## Test scope
Which modules/tests matter for this change.

## Amendments
(empty at first)
```

## Quality bar

- Every acceptance criterion could be checked by a test or a command.
- Each fact appears once. If two sections say the same thing, keep one.
- Open questions are resolved with the user before finishing. Don't leave "TBD" in Decisions.
- For blockchain work, make the on-chain/off-chain boundary, account and authority model, and failure/finality handling explicit, because these are the usual sources of rework.

End by telling the user the next step: `/justdev review`.
