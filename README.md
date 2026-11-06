# justdev

A Claude Code skill for running large backend / blockchain tasks as a staged workflow. Long chats lose details, so every stage writes its result to a file and the next stage starts fresh from those files.

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
| `plan` | Interview you, then write `PLAN.md` in a new task folder |
| `review` | Independent review of the plan; fix conflicts; you approve once |
| `implement` | Build in slices, checkpoint in `PROGRESS.md`, log `DEVIATIONS.md` |
| `test` | Run only the tests related to what changed (full suite only if you ask) |
| `report` | CHANGELOG entry + `NOTES.md` with improvements and next steps |
| `integrate` | Short guide for frontend / integrating teams (`INTEGRATION.md` + `.html`) |
| `amend` | Change an in-flight plan without restarting |

Typical run: `plan` → `review` → `implement` → `test` → `report` → `integrate`. Each stage suggests the next one and stops; you trigger it.

## Task folder

Each task lives in `tasks/<YYYY-MM-DD-slug>/` in your repo:

```
PLAN.md  PROGRESS.md  DEVIATIONS.md  TEST-RESULTS.md
NOTES.md  INTEGRATION.md  INTEGRATION.html
```

## Tips

- Run `/clear` (or use a subagent) between stages so review, test, report and integrate are independent of earlier reasoning.
- Add a **test map** to your project's `CLAUDE.md` so `test` knows which command covers which module, e.g. `programs/foo/ → cargo test -p foo`. If it's missing, the skill infers one and asks before saving it.
- `PLAN.md` always holds the current truth. Changes go through `amend`, which edits it in place.

## Layout

```
SKILL.md          entry point and shared rules
references/       one file per stage
```
