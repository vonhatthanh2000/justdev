# Stage: implement (MVP)

Goal: build the **MVP only** of the next sub-task, the simplest version that runs end to end, then stop at the checkpoint so the user can confirm the direction before anything is hardened. Everything in the sub-task's Hardening backlog is deferred on purpose; that is `harden`'s job.

## Start

Read PLAN.md and PROGRESS.md (if it exists), and DEVIATIONS.md if it exists, and nothing else from earlier work. PLAN.md is the whole spec. **Don't read TASK.md or PIVOTAL.md**: both are written for humans, and building from them leads to building something PLAN.md doesn't say. Decisions in PLAN.md are fixed, so don't re-open them.

**Pick the sub-task**: the first one in PROGRESS.md's Sub-tasks list whose MVP isn't confirmed. Build only its `### MVP` section. If PROGRESS.md shows that MVP in progress, resume from "next"; don't redo finished tasks. If the sub-task has "Rejected attempts" lines in PLAN.md, read them first: they say what the user didn't want, so don't repeat it.

**Check the working tree.** Uncommitted changes from earlier sub-tasks are expected (PROGRESS.md lists them). If there are changes neither this workflow nor PROGRESS.md accounts for, tell the user before touching anything. If earlier sub-tasks are still uncommitted, suggest the user commit them first: then this MVP can be reverted cleanly if it's rejected. Don't commit for them.

**Record the base commit.** If PROGRESS.md has no `Base:` line, add one directly under the status line the first time you start: `Base: <output of git rev-parse HEAD>` (before you change anything). The test stage diffs against it, so keep it as is. On a resume or a later sub-task, never overwrite it.

Set the status to `implement <NN> · in progress · next: harden <NN>`.

## MVP rules

- **Only the MVP section's tasks.** Happy path only. Anything listed in the Hardening backlog is skipped, even if it looks quick.
- **No new interfaces, traits, abstract classes or layers.** Call things directly.
- **No spec (test) files.**
- **Hardcoding, inline logic and a simple structure are fine** (one file, a constant instead of config, a stubbed dependency). Keep a running list of everything you hardcode or fake; it goes into the checkpoint.
- **Throwaway check script.** If the MVP has no visible output (a background job, a pure function, an on-chain instruction), you may write a quick script or curl call that prints the result. It is not a spec file. Prefer `tasks/<folder>/check-<NN>.<ext>` when it can run from there; otherwise use a clearly named file (e.g. `check-<NN>-throwaway.<ext>`). Record its path in PROGRESS.md so `harden` removes it or turns it into a spec.
- **Size limit: about 10–15 minutes of work or a handful of files.** If it is going over (more files than the plan names, tasks growing, a missing piece that needs its own build), stop at a working point, update PROGRESS.md, tell the user, and suggest `/justdev amend` to split the sub-task. `plan` should have split it; don't push through.
- **Drift stop.** If anything contradicts the plan (the code isn't shaped the way PLAN.md assumes, a Decision can't hold, the How to verify can't work as written), stop and ask. Don't improvise a workaround. A small implementation detail the plan simply left open is not drift: decide it, log it in DEVIATIONS.md, carry on.

## Work in tasks

For each MVP task:

1. Implement it.
2. Do a quick sanity check (build/typecheck), not the test stage.
3. **Tick it in PLAN.md** (`- [ ] 2.` → `- [x] 2.` under the sub-task's MVP) and update PROGRESS.md. Do both right away, not in a batch at the end. Ticking is the only edit you make to PLAN.md; if the task's text is wrong, that's a drift stop.
4. **Don't run `git commit`, `git add`, or anything that changes git history or the index; the user commits manually.** Leave the changes in the working tree.

## PROGRESS.md

Update it after every task, not at the end. A context that is already overloaded can't be trusted to save state later. Keep it short:

```markdown
# Progress
Status: implement 02 · in progress · next: harden 02 · updated 2026-11-19
Base: <commit sha before any changes>
## Sub-tasks
- 01 – <heading>: hardened
- 02 – <heading>: MVP v2 in progress (v1 rejected)
- 03 – <heading>: not started
## Done
- [x] 02 task 1: <what> (files touched, and whether each was created or edited)
## In progress
- 02 task 2: <current state, what's left>
## Next
- 02 task 3, then checkpoint
## Checkpoint
(written at the checkpoint, see below)
## Gotchas
- Things the next agent must know (surprising behavior, workarounds, open questions)
## History
(existing lines stay exactly as they are; see SKILL.md)
```

List files touched per task, created vs edited: if the MVP is rejected, `amend` uses this list to revert it. Refresh the status date as you go. Never rewrite or drop History lines when you update the rest of the file. Add one when a significant deviation happens, pointing at its DEVIATIONS.md entry.

## DEVIATIONS.md

The moment you do something the plan didn't say, append an entry: what, why, and impact. Write it immediately; recalling it later is unreliable, and the checkpoint, report and integration stages depend on it. Anything that contradicts the plan is a drift stop (above), not a deviation to log and continue.

## Checkpoint

When the MVP tasks are done:

1. **Run the MVP yourself**, using the sub-task's "How to verify" command from PLAN.md. Compare with the expected output. If it fails, fix it within the MVP scope (a couple of attempts at most); if it still fails, present the failure plainly, never as a success. If it can't run here (it needs credentials, a deployed network or a device you don't have), say so and ask the user to run it.
2. **Write the checkpoint to PROGRESS.md** so a fresh agent can pick it up after `/clear`. Replace the previous checkpoint section:
   ```markdown
   ## Checkpoint
   - Sub-task: 02 · MVP v2
   - Command: `<exact command>`
   - Result: matches expected | differs: <how>
   - Hardcoded / faked: <list>
   - Check script: <path, or none>
   ```
3. Set the status to `implement <NN> · awaiting user · next: harden <NN>` and update the sub-task line (`MVP v2 awaiting confirm`).
4. **Show the user**, briefly:
   ```
   Checkpoint: 02 – <heading> · MVP v2

   Output:
   <the actual output of the run, trimmed to the relevant lines>

   Rerun it yourself:
   <command>

   Hardcoded / faked:
   - <item>

   Deviations from the plan:
   - <item, or "none">

   Reply "confirm" to harden this, or tell me what's wrong.
   ```
5. **Wait.** Nothing gets hardened until the user replies "confirm".

## User's reply

- **"confirm"** (or an unambiguous equivalent): add a History line `MVP v<N> confirmed (<NN>)`, set the sub-task line to `MVP v<N> confirmed`, set the status to `implement <NN> · complete · next: harden <NN>`, and suggest `/justdev harden`. Mention this is a good moment for the user to commit a restore point.
- **The MVP is wrong**, in any degree, including "almost, but X should be Y": follow "Rejected MVP" in [amend.md](amend.md). Don't patch it on the spot; the reason must be recorded in PLAN.md first.
- **A question**: answer it and keep waiting. Don't treat a question as a confirmation.

## Rules

- Stay inside the MVP's scope. Notice-but-don't-fix unrelated problems; put them in PROGRESS.md gotchas for the report.
- If the context is getting long, make sure PROGRESS.md is current and tell the user they can `/clear` and re-run `/justdev implement`. The files are enough to continue.
