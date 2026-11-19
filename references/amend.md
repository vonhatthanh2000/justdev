# Stage: amend

Goal: change an in-flight plan without restarting, including when the user rejects an MVP at a checkpoint. The user describes what changed and why. PLAN.md stays the single source of truth: edit it in place, never create PLAN-v2 or a side file. TASK.md follows PLAN.md: whenever a sub-task is added, split, renumbered or its outcome changes, update TASK.md in the same pass.

## First, classify the change

| Situation | Action |
|---|---|
| The user rejected an MVP at a checkpoint | **Rejected MVP** (below) |
| `implement` hit the MVP size limit | Split the sub-task (simple amend): rewrite it as two or more sub-tasks, each with its own MVP, How to verify and Hardening backlog. Renumber later sub-tasks in PLAN.md, TASK.md and PROGRESS.md together, and tell the user TASK.md's numbering changed so they can update the dashboard. |
| Implementation detail the plan didn't specify | No amend. The implementer decides and logs it in DEVIATIONS.md. |
| Small change **within the original Goal** (extra edge case, tweak to a task or criterion) that does not touch any Decision | Simple amend (below) |
| Touches a **Decision** or a **PIVOTAL.md** entry, or invalidates work already done | Amend + **full** review, mark invalidated tasks `[needs rework]` in PROGRESS.md. A pivotal change needs the user's explicit confirmation again. Update PLAN.md first (it's what gets implemented), then rewrite the matching PIVOTAL.md section in place (new function, effect, trade-offs and scenarios), not patched. |
| New feature or a different Goal | Not an amend. Create a new task folder with its own PLAN.md; add a "Related" line linking the two. |

Always ask yourself explicitly: does this contradict any line under Decisions or Pivotal? That one question separates a safe tweak from a hidden conflict. Tell the user how you classified it.

## Simple amend

1. Don't commit anything (the user commits manually). If the user wants a clean before/after in git, suggest they commit the current state first.
2. Rewrite the affected sections of PLAN.md to the new state. Delete superseded text; don't strike it through or leave it beside the new text. Put new scope in the right place: behavior the MVP must show goes in its MVP section; structure, specs and edge cases go in the Hardening backlog. If the change hits a sub-task that's already hardened, that sub-task needs a new MVP and checkpoint only if its approved behavior changes; otherwise add the item to its backlog and mark it `[needs rework]` in PROGRESS.md for harden.
3. Update TASK.md if a sub-task's outcome changed.
4. Add one line to PROGRESS.md's History: date, `amend`, what changed, why. (PLAN.md itself carries no change log.)
5. Run a delta review in a fresh context: the reviewer reads the changed sections and checks them against the rest of the plan for conflicts, duplicates and untestable criteria (see [review.md](review.md)).
6. Update PROGRESS.md if sub-tasks or tasks were added, removed or reordered, and set the status to where the work now is (usually `implement <NN> · in progress` or `harden <NN> · in progress`, or `review · awaiting user` for a full review).
7. Resume with `/justdev implement` or `/justdev harden`. Leave the changes uncommitted.

## Full-review amend

Same as above, but at step 5 run the complete review from [review.md](review.md) and ask the user to re-approve. Mark finished tasks that are now invalid as `[needs rework]` in PROGRESS.md so the implementer revisits them. Set the status to `review · awaiting user` until the user re-approves, and add a History line for the re-approval too.

## Rejected MVP

When the user says the MVP shown at a checkpoint is wrong. Version numbers count attempts per sub-task: v1, v2, …

**Escalation first.** Count the sub-task's "Rejected attempts" lines in PLAN.md. If this rejection makes two, record it (step 1), log it (step 5), then stop patching: set the status to `plan · in progress`, add a History line `escalated to plan after 2 rejected MVPs (<NN>)`, tell the user the requirement is probably unclear, and suggest `/justdev plan` on the same folder to re-run the interview for this sub-task. Don't build a v3.

Otherwise:

1. **Record why.** Under the sub-task's MVP in PLAN.md, add one line to **Rejected attempts**: `MVP v1 rejected: expected X, got Y, because Z.` Take X and Y from the user's words and the checkpoint. If the cause (Z) isn't clear, ask; it's what keeps v2 from repeating the mistake.
2. **Update the MVP section** (tasks, How to verify) to the corrected direction. Touch the Hardening backlog only if it's affected. If the correction contradicts a Decision or a PIVOTAL.md entry, this is also a full-review amend.
3. **Ask the user which case it is**, with your recommendation:
   - **Small miss**: fix forward on top of the current MVP.
   - **Wrong direction**: revert the MVP and rebuild.
4. **Update TASK.md** if the outcome changed.
5. **Log it.** In PROGRESS.md: set the sub-task line to `MVP v2 in progress (v1 rejected)`, mark the affected MVP tasks not done (untick them in PLAN.md too, and give changed or new tasks an unchecked box), and add a History line `MVP v1 rejected (<NN>): <short reason>; fix forward | revert`. For a wrong direction, also add a DEVIATIONS.md entry for the direction change.
6. **Revert, if it's a wrong direction.** Undo only this sub-task's MVP, by editing files, never with git (`checkout`, `reset`, `stash`, `restore` are all off limits). Use the files PROGRESS.md lists for this MVP and `git diff` to see them: delete files the MVP created, restore edited files to their content before the MVP. If the user committed before the MVP started, diffing against that commit shows exactly what to undo. Don't touch earlier sub-tasks' work. If you can't tell precisely what a file looked like before the MVP (its changes are mixed with uncommitted earlier work), ask the user instead of guessing. Then run a delta review of the changed MVP section in a fresh context (see [review.md](review.md)), since the direction changed.
7. **Build v2.** Set the status to `implement <NN> · in progress · next: harden <NN>` and run implement again ([implement.md](implement.md)), ending with the checkpoint again. For a small miss you can continue straight into implement in this session; after a wrong direction, run it after the delta review.

## Rule

The change must be written to PLAN.md before any code for it is written. If it only exists in chat, the next fresh agent won't know about it.
