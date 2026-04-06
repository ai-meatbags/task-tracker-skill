# task-tracker-skill

`task-tracker` is a skill for running project work from persistent task artifacts instead of chat context.

It helps an agent:
- start new work from a feature-level artifact
- decompose work into executable tasks
- pick the next task based on dependencies and status
- resume current work from saved state
- move tasks through execution, testing, and retro
- surface tasks that can start in parallel

## When To Use It

Use `task-tracker` when your project already keeps delivery state in files and you want the agent to work from that state reliably across long threads, pauses, and handoffs.

This skill is a good fit for:
- feature work with multiple tasks
- resumable execution
- policy-driven delivery flows
- multi-agent task execution

## Commands

### New work

- `новая задача`
- `new task`

Starts from a feature dossier, prepares a preview, applies the simple-vs-feature gate, and asks for confirmation before creating artifacts.

Preview replies:
- `делай`
- `делай, но поправь: ...`
- `отмена`

### Execute a task

- `сделай задачу N`
- `сделай PREFIX-N`
- `сделай следующую задачу`
- `го след задачу`
- `го след таску`

Resolves the requested task or finds the next executable one from project state.

### Resume work

- `текущая задача`
- `current task`
- `продолжай задачу`
- `continue task`

Restores execution context from task artifacts, not from the chat.

### Retro inbox

- `разбери ретро-инбокс`
- `review retro inbox`

Processes retro findings and routes them into project patterns, follow-up tasks, upstream candidates, defers, or rejects.

### Help

- `task tracker help`

## How It Works

`task-tracker` reads the project policy layer first, then uses it to decide:
- which artifacts to create
- how to interpret task status
- how to pick the next task
- when testing starts
- when retro is required
- which tasks can run in parallel

For new work, it shows a preview first and waits for confirmation.

For execution and resume, it recovers state from project artifacts and updates that state as work progresses.

## Project Requirements

`task-tracker` expects the target project to already have:
- a machine-readable project config
- a project policy layer
- feature and task artifacts with persistent state
- templates or conventions for delivery artifacts

The exact contract is defined in [`SKILL.md`](./SKILL.md).

## Companion Skill

If you also need to set up or maintain the project-side delivery structure, use [`project-tracker-skill`](https://github.com/ai-meatbags/project-tracker-skill.git).

Typical split:
- `project-tracker-skill` sets up and maintains the project delivery layer
- `task-tracker-skill` runs day-to-day task execution on top of that layer

## Repository Contents

- `SKILL.md` — skill definition and operating rules

