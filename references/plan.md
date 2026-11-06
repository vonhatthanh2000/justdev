# Stage: plan

Goal: turn the user's description into a PLAN.md that a fresh agent can implement without asking anything. This stage is interactive and runs in the main session.

## Steps

1. **New task:** create `tasks/<YYYY-MM-DD-slug>/` (slug: short, kebab-case, from the goal) and its PROGRESS.md with the title, `Status: plan · in progress · next: review · updated <date>`, and an empty `## History` (format in SKILL.md).
   **Re-entering an existing task** (the folder exists, e.g. review sent you back or the user wants to refine the plan): don't create anything. Read the existing PLAN.md (and PIVOTAL.md) and PROGRESS.md, set the status to `plan · in progress`, and continue the interview from what's already decided, asking only about what's open or changing. Edit PLAN.md in place, rewriting affected text and deleting what it replaces. Never overwrite PROGRESS.md or touch existing History lines; only append.
2. Explore the relevant code first so your questions are informed, not generic.
3. **Grill the user until nothing is unclear.** Whenever something is ambiguous, ask instead of assuming: a wrong guess costs far more in rework than a question costs now. Follow the approach of the `grill-with-docs` skill: invoke the `grilling` skill for the interview, and also `domain-modeling` when the task introduces or renames domain terms or makes a decision worth recording as an ADR. (`grill-with-docs` itself is user-invoke-only and just calls those two.) If they aren't installed, do the same by hand using the rules below.
   - Ask what changes what gets built: scope edges, data formats, failure behavior, security or auth, compatibility, what must not change.
   - Explore the code first and answer what you can from it yourself; only ask the user what the code can't tell you.
   - Ask in small related groups, and give a recommended answer for each so the user can reply "yes". Keep going until the answers stop surfacing new questions.
   - Challenge assumptions that look risky, including the user's.
4. **Surface pivotal decisions.** While interviewing, watch for choices that are expensive to reverse or that much of the project will depend on (see "What counts as pivotal" below). Don't settle these quietly inside a normal answer: stop, present them as their own question, and get an explicit confirmation.
5. Write PLAN.md, and `PIVOTAL.md` if there were pivotal choices. Stop and show the user. Don't start implementing.
   - **PLAN.md is the only thing implementing agents build from.** Everything they need must be in it, including the outcome of every pivotal choice, written as an actionable line under Decisions (what to build, with any resulting constraints, tasks or acceptance criteria placed in their own sections).
   - **PIVOTAL.md is for humans only**: the record you and the user read to make and revisit big decisions. Agents don't implement from it, so nothing may exist only there.

## Template

```markdown
# <Task title>

## Goal
One or two sentences: the outcome, not the method.

## Non-goals
What this task deliberately does not do.

## Decisions
Final answers only, one line each, with a short reason. No debate history. Includes the confirmed outcome of each pivotal choice as an actionable line (e.g. "Use X for Y; do not Z"), so the implementer needs nothing else.

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
```

PLAN.md carries no change log. Changes to it are recorded in PROGRESS.md's History.

## What counts as pivotal

A choice is pivotal when getting it wrong would be costly: a large part of the work gets redone, or the project ends up worse off in a way that's hard to undo. Judge this from the task and the code in front of you; don't match against a fixed list of topics. The tests to apply:

- **Reversibility:** how hard and expensive is it to change after the work is built on top of it?
- **Reach:** how much of the project (modules, interfaces, data, other teams) depends on it?
- **Visibility:** does it become a contract that outside parties rely on?

If a choice scores high on any of these, treat it as pivotal.

For each one, present it as its own question: the options (2-3), your recommendation, and the trade-offs that matter for that particular choice. Wait for the user's explicit answer. Don't mark it confirmed on a vague reply or an assumption; if the user is unsure, say what you'd need to find out (a spike, a benchmark) and record it as an open question, not as a decision. When one is confirmed, add a History line to PROGRESS.md right away (choice and outcome in one line).

Ordinary choices (naming, minor structure, small library calls) are not pivotal; they go in Decisions. Keep the list short, since an overlong one means the signal is lost.

**PIVOTAL.md vs ADRs.** PIVOTAL.md is about this one task only. An ADR (what `domain-modeling` writes) records a decision about the whole system. They don't overlap: if a choice made here also changes how the whole system works, record it as an ADR where the repo keeps its ADRs, and have the PIVOTAL.md entry link to it instead of repeating it. Keep the ADR/glossary output out of the task folder.

## PIVOTAL.md template

One file per task, one section per pivotal choice. It's written for humans, to help the user decide and to remind the team later why it was chosen, so it can be explanatory. Write it so a reader who wasn't in the interview understands what was chosen and what it will mean in practice. Keep each section to roughly half a page; concrete beats comprehensive. Agents don't implement from this file.

```markdown
# Pivotal choices: <task title>

## 1. <Choice name>
**Chosen:** what we're going with, and over which alternative(s), in one or two lines.
**Status:** confirmed by user, <date>.

**Main function**
What this does in the system, in plain words. What problem it solves.

**Effect on the project**
How it shapes the rest of the work: which modules, interfaces, data or infrastructure depend on it; what it makes easy; what it makes hard to change later.

**Trade-offs**
What we gain and what we give up, along whichever dimensions matter for this choice. Name the cost honestly; if the downside is only "more code", say what code.

**Scenarios**
Concrete situations that can happen once it's applied. Cover a normal case, a stress or edge case, and a failure case. Each is one line of "when X happens, then Y":
- Normal: ...
- Load / edge: ...
- Failure: ... (and how the system behaves or recovers)

## 2. <Next choice>
...
```

The scenarios matter most: they make the user picture the consequences before confirming. Base them on how the real system would behave (using the code you explored), not on generic statements. If a scenario implies behavior the code must have (e.g. a failure that must be handled), put that into PLAN.md as a task or acceptance criterion, because the scenario text itself isn't read by the implementer.

## Quality bar

- Every acceptance criterion could be checked by a test or a command.
- Each fact appears once. If two sections say the same thing, keep one.
- Open questions are resolved with the user before finishing. Don't leave "TBD" in Decisions or PIVOTAL.md.
- Every PIVOTAL.md entry has an explicit user confirmation behind it, and its outcome is already stated in PLAN.md. Deleting PIVOTAL.md would not change what an implementer builds.
- If the task has no pivotal choices, skip PIVOTAL.md.
- For blockchain work, make the on-chain/off-chain boundary, account and authority model, and failure/finality handling explicit, because these are the usual sources of rework.

When PLAN.md is written, add a History line ("plan written"), set the status to `plan · complete · next: review`, and tell the user the next step: `/justdev review`.

Every update to PLAN.md made in this stage is logged, not just the first draft. When re-entering, add a History line for each update you make (what changed and why; edits made in the same pass can share one line), including small ones. The history is how the user and later agents see how the plan evolved, and PLAN.md itself keeps only the current text.
