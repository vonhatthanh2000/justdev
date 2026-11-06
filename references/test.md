# Stage: test

Goal: verify the change, running only tests related to it. Use a fresh context so the tester doesn't inherit the implementer's assumptions.

## Scope

1. Get the changed files: `git diff --name-only <base>...HEAD` (use the branch base, or the commit before the task started; check the first PLAN-related commit if unsure) plus uncommitted changes.
2. Map them to test commands using the test map in CLAUDE.md (see SKILL.md). Examples: `cargo test -p <crate>`, `jest --findRelatedTests <files>`, `vitest related <files>`, `pytest <path>`, program tests for the changed on-chain module.
3. Also cover the plan's "Test scope" and acceptance criteria. If a criterion has no test, say so (and write a test if it's small and clearly within scope).

**Run the full project suite only if the user explicitly asks.** Even if you suspect wider impact, name the risk in the results instead of running everything.

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

End by suggesting `/justdev report` (or a return to implement if failures remain).
