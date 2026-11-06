# Stage: test

Goal: verify the change, running only tests related to it. Use a fresh context so the tester doesn't inherit the implementer's assumptions.

## Scope

1. Get the changed files from the local working tree: use the `Base:` commit recorded in PROGRESS.md. `git diff --name-only <Base>` lists committed and uncommitted changes to tracked files; add new untracked files from `git status --porcelain`. Nothing needs to be pushed. Ignore everything under `tasks/` (PLAN.md, PROGRESS.md and the other workflow files change constantly but aren't code, and they map to no test); for example `git diff --name-only <Base> -- . ':(exclude)tasks'`, and filter `tasks/` out of the untracked list too. If there's no `Base:` line, say so and ask the user for the starting commit instead of guessing.
2. Find the **affected modules**: the ones that use the changed code (callers, importers, dependent crates or packages, programs that call a changed instruction). Look it up rather than guess, e.g. `cargo tree -i <crate>`, grep for the changed symbols, or the test runner's own related-files option. Keep this to direct dependents unless a change to a public interface or shared type clearly reaches further.
3. Map the changed code and the affected modules to test commands using the test map in CLAUDE.md (see SKILL.md). Examples: `cargo test -p <crate>`, `jest --findRelatedTests <files>`, `vitest related <files>`, `pytest <path>`, program tests for the changed on-chain module.
4. Also cover the plan's "Test scope" and acceptance criteria. If a criterion has no test, say so (and write a test if it's small and clearly within scope).

So the scope is exactly two things: the code implemented in this task, and the modules affected by it. Nothing else. **Run the full project suite only if the user explicitly asks.** If you suspect the impact goes beyond the affected modules, don't widen the run: name the risk in the results instead.

## Run and retry

- Run the scoped tests and record results.
- On failure, work out whether the code or the test is wrong. Fix clear, small problems and rerun. Limit this to 2-3 attempts in total.
- After that, stop and escalate: report what fails, what you tried, and your best diagnosis. Don't loop, and don't weaken or delete tests to make them pass.

## TEST-RESULTS.md

```markdown
# Test results
## Scope
Changed files / modules and the commands run.
## Results
Pass/fail per command, with the key output of any failure.
## Fixed during testing
What was changed and why (also add to DEVIATIONS.md if it changes behavior).
## Not covered
Acceptance criteria or areas with no test, and wider risks not run.
```

Status and history in PROGRESS.md: set `test · in progress` when you start. If everything passes, set `test · complete · next: report` and suggest `/justdev report`. If failures remain after your attempts, set `implement · in progress` (the work is back in implementation), add a History line saying what failed and why it returned, and suggest `/justdev implement`.
