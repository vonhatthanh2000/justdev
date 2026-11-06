# Stage: review

Goal: an independent check of PLAN.md before any code is written. Run it in a fresh context (a subagent, or after `/clear`). The reviewer reads PLAN.md and the relevant code, not the planning conversation, because independence is the point.

## Check for

- **Contradictions** between sections (a Decision vs a Task, a Constraint vs a Criterion).
- **Duplicates**: the same fact stated in more than one place, which will drift apart later.
- **Vague wording** that two engineers would implement differently ("handle errors properly", "fast enough").
- **Untestable acceptance criteria.**
- **Missing edge cases**: empty/zero/max values, failure and retry paths, concurrency, permissions, and for blockchain: failed or dropped transactions, finality, account size and cost limits, upgrade authority.
- **Ordering problems**: a task that depends on a later task.
- **Plan vs reality**: files, functions, or interfaces the plan assumes that don't exist in the code.
- **Scope drift**: tasks that don't serve the Goal.
- **Pivotal gaps**: PLAN.md is what gets implemented, so judge it on its own. If PIVOTAL.md exists, read it only to cross-check. Flag a pivotal choice (costly to reverse, or widely depended on) that is buried in Decisions or Tasks, or implied but never confirmed. Don't make the call yourself; raise it as a question for the user so it can get a PIVOTAL.md entry after explicit confirmation. Also flag any PIVOTAL.md choice whose outcome is missing from or contradicted by PLAN.md, and entries missing one of the four parts (function, effect, trade-offs, scenarios).

## Output

1. Report the issues first, grouped by severity (blocking / should fix / minor), each with the section it's in and a concrete suggested fix.
2. Then apply the fixes to PLAN.md in place, rewriting the text and deleting what it replaces. Where a fix would change the user's intent rather than clarify it, don't decide silently: list it as a question for the user.
3. Show the user a short summary of what changed and any open questions.

## Approval

This is the user's single approval gate. Ask them to approve the plan (or answer the open questions) before implementation starts.

Keep the status current in PROGRESS.md: `review · in progress` while reviewing, `review · awaiting user` while waiting for approval. On approval, set `review · complete · next: implement` and add a History line ("plan approved"). If the review found blocking issues or changed PLAN.md, also add a History line saying what changed and why. Then suggest `/justdev implement`.
