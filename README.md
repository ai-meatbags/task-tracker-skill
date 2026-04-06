# task-tracker-skill

`task-tracker` is a standalone skill for running delivery work from project task artifacts: creating feature dossiers, decomposing work into tasks, resuming execution, handling testing and retro stages, and keeping task state aligned with project policy.

## Works Together With `project-tracker-skill`

This skill is designed to work in tandem with [`project-tracker-skill`](https://github.com/ai-meatbags/project-tracker-skill.git).

Responsibility split:
- `project-tracker-skill` bootstraps the project process layer: `rep.config.json`, `tasks/_policies/*`, templates, and baseline artifact structure.
- `task-tracker-skill` operates on top of that layer: it reads the project policy set and uses it to create, execute, resume, test, and close tasks safely.

## Required Project Contract

Before using `task-tracker`, the target project is expected to already contain the policy and template layer created or maintained by `project-tracker-skill`, including:
- `rep.config.json`
- `tasks/_policies/dev-plan.md`
- `tasks/_policies/arch-patterns.md`
- `tasks/_policies/project-patterns.md`
- project-local templates referenced from `rep.config.json`

If that contract is missing or invalid, `task-tracker` should stop and route the workflow back to `project-tracker`.

## Repository Contents

- `SKILL.md` — the skill definition and operating rules.

