# justdev

A Claude Code skill for running large backend / blockchain tasks as a staged workflow. Long chats lose details, so every stage writes its result to a file and the next stage starts fresh from those files. Work is split into small sub-tasks, and each one is built as an MVP first: the agent stops so you can confirm the direction before it hardens anything.

## Install

```bash
# all projects
git clone git@github.com:vonhatthanh2000/justdev.git ~/.claude/skills/justdev

# or one project only
git clone git@github.com:vonhatthanh2000/justdev.git <project>/.claude/skills/justdev
```

Restart Claude Code (or start a new session), then run `/justdev`.

## Usage

`/justdev <stage> [task-folder-or-description]`

| Stage | What it does |
|---|---|
| `plan` | Interview you, then write `PLAN.md` (MVP + Hardening backlog per sub-task) and `TASK.md` in a new task folder |
| `review` | Independent review of the plan; fix conflicts; you approve once |
| `implement` | Build the **MVP only** of the next sub-task, run it, then stop at the checkpoint |
| `harden` | After you reply "confirm": interfaces, modules, spec files, error handling, edge cases |
| `test` | Run only the tests related to what changed (full suite only if you ask) |
| `report` | CHANGELOG entry + `NOTES.md` with improvements and next steps + Result lines in `TASK.md` |
| `integrate` | Short guide for frontend / integrating teams (`INTEGRATION.md` + `.html`) |
| `amend` | Change an in-flight plan without restarting; handles a rejected MVP |

Typical run:

```
plan → review → for each sub-task: implement (MVP) → checkpoint → harden
              → test → report → integrate
```

At each checkpoint the agent shows the output of the run, the command to rerun it, what it hardcoded or faked, and any deviations. Reply **"confirm"** to harden it, or say what's wrong: `amend` records why, asks whether to fix forward or revert, and builds MVP v2. After two rejected MVPs for the same sub-task it goes back to `plan`. Each stage suggests the next one and stops; you trigger it.

## Task folder

Each task lives in `tasks/<YYYY-MM-DD-slug>/` in your repo:

```
PLAN.md  TASK.md  PIVOTAL.md  PROGRESS.md  DEVIATIONS.md  TEST-RESULTS.md
NOTES.md  INTEGRATION.md  INTEGRATION.html
```

## Tips

- Run `/clear` (or use a subagent) between stages so review, test, report and integrate are independent of earlier reasoning.
- Add a **test map** to your project's `CLAUDE.md` so `test` knows which command covers which module, e.g. `programs/foo/ → cargo test -p foo`. If it's missing, the skill infers one and asks before saving it.
- `PLAN.md` always holds the current truth. Changes go through `amend`, which edits it in place.
- `TASK.md` is business-worded (no file or class names), one block per sub-task, ready to copy into your dashboard by hand.
- Commit after each confirmed or hardened sub-task: the agent never commits, and a commit before each MVP makes a rejected one easy to revert.

## Layout

```
SKILL.md          entry point and shared rules
references/       one file per stage
```
