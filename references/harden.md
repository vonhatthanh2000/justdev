# Stage: harden

Goal: turn a sub-task's confirmed MVP into production-quality code by working through its Hardening backlog (interfaces, modules, spec files, error handling, edge cases), **without changing the behavior the user approved**.

## Start

1. Read PLAN.md (the sub-task's MVP and Hardening backlog, plus Decisions, Constraints and the sub-task's acceptance criteria), PROGRESS.md (especially the Checkpoint section: the hardcoded/faked list and the check script path) and DEVIATIONS.md. **Don't read TASK.md or PIVOTAL.md**; PLAN.md is the only spec.
2. **Confirmation gate.** PROGRESS.md's History must have `MVP v<N> confirmed (<NN>)` for this sub-task. If it doesn't, don't harden: show the user the checkpoint from PROGRESS.md and ask them to confirm. If they reply "confirm", record it as described in implement.md's "User's reply", then continue. Never harden an unconfirmed MVP.
3. Set the status to `harden <NN> · in progress` and the sub-task line to `hardening`. If PROGRESS.md shows hardening already in progress, resume from "next".

## Work in slices

Group the backlog into slices, each leaving the code working. A typical order:

1. **Structure**: interfaces, modules, folder layout. A pure refactor; behavior stays identical.
2. **Replace the fakes**: every item on the checkpoint's hardcoded/faked list gets its real implementation, unless the backlog says it stays.
3. **Error handling, edge cases, validation, config.**
4. **Spec files** covering the sub-task's acceptance criteria and the edge cases from step 3.
5. **Clean up**: remove the throwaway check script, or turn it into a spec. Docs the backlog asks for.

For each slice:

1. Implement it.
2. **Re-run the MVP's "How to verify"** from PLAN.md. It must still give the expected output: that is the behavior the user approved, and the refactor must not break it. If it fails, fix it before the next slice.
3. Do a quick sanity check: build/typecheck and the spec files you just wrote. The formal run is the test stage.
4. **Tick the finished backlog items in PLAN.md** (`- [ ] H3.` → `- [x] H3.`) and update PROGRESS.md (Done, In progress, Next). Ticking is the only edit you make to PLAN.md.
5. Go straight on to the next slice. Harden slices don't stop for the user, except on drift.

**Don't run `git commit`, `git add`, or anything that changes git history or the index; the user commits manually.**

## Drift stop

Stop and ask, without improvising, when:

- a backlog item can't be done without changing the approved behavior (the How to verify output would have to change);
- something contradicts PLAN.md (a Decision, a Constraint, or the code isn't shaped as the plan assumes);
- the backlog turns out to be missing something essential that's bigger than an implementation detail.

Set the status to `harden <NN> · blocked` (say on what) while you wait. If the answer changes the plan, it goes through `/justdev amend`. Small implementation details the plan left open aren't drift: decide, log in DEVIATIONS.md, carry on.

## DEVIATIONS.md

Same rule as implement: the moment you do something the plan didn't say, append what, why and impact.

## End

When every backlog item is ticked in PLAN.md and How to verify still passes:

1. Check that no throwaway check script is left (or that it's now a real spec).
2. Collapse the sub-task's Done entries in PROGRESS.md into its Sub-tasks line (`02 – <heading>: hardened`) to keep the file short, and clear the Checkpoint section.
3. Set the status:
   - more sub-tasks left: `harden <NN> · complete · next: implement <NN+1>`, and suggest `/justdev implement`;
   - last sub-task: `harden <NN> · complete · next: test`, and suggest `/justdev test`.
4. Mention this is a good moment for the user to commit a restore point (before the next MVP starts, so it can be reverted cleanly if rejected).

If the context is getting long, make sure PROGRESS.md is current and tell the user they can `/clear` and re-run `/justdev harden`.
