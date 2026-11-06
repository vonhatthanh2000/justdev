# Stage: amend

Goal: change an in-flight plan without restarting. The user describes what changed and why. PLAN.md stays the single source of truth: edit it in place, never create PLAN-v2 or a side file.

## First, classify the change

| Situation | Action |
|---|---|
| Implementation detail the plan didn't specify | No amend. The implementer decides and logs it in DEVIATIONS.md. |
| Small change **within the original Goal** (extra edge case, tweak to a task or criterion) that does not touch any Decision | Simple amend (below) |
| Touches a **Decision** or a **PIVOTAL.md** entry, or invalidates work already done | Amend + **full** review, mark invalidated tasks `[needs rework]` in PROGRESS.md. A pivotal change needs the user's explicit confirmation again. Update PLAN.md first (it's what gets implemented), then rewrite the matching PIVOTAL.md section in place (new function, effect, trade-offs and scenarios), not patched. |
| New feature or a different Goal | Not an amend. Create a new task folder with its own PLAN.md; add a "Related" line linking the two. |

Always ask yourself explicitly: does this contradict any line under Decisions or Pivotal? That one question separates a safe tweak from a hidden conflict. Tell the user how you classified it.

## Simple amend

1. Commit current state first, so `git diff` later shows exactly what changed.
2. Rewrite the affected sections of PLAN.md to the new state. Delete superseded text; don't strike it through or leave it beside the new text.
3. Add one line to PROGRESS.md's History: date, `amend`, what changed, why. (PLAN.md itself carries no change log.)
4. Run a delta review in a fresh context: the reviewer reads the changed sections and checks them against the rest of the plan for conflicts, duplicates and untestable criteria (see [review.md](review.md)).
5. Update PROGRESS.md if tasks were added, removed or reordered, and set the status to where the work now is (usually `implement · in progress`, or `review · awaiting user` for a full review).
6. Commit the amendment, then resume with `/justdev implement`.

## Full-review amend

Same as above, but at step 4 run the complete review from [review.md](review.md) and ask the user to re-approve. Mark finished tasks that are now invalid as `[needs rework]` in PROGRESS.md so the implementer revisits them. Set the status to `review · awaiting user` until the user re-approves, and add a History line for the re-approval too.

## Rule

The change must be written to PLAN.md before any code for it is written. If it only exists in chat, the next fresh agent won't know about it.
