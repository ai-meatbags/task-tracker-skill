---
name: task-tracker
description: >-
  Use when user asks to execute or prepare project tasks: "новая задача", "new task", "сделай задачу N",
  "сделай PREFIX-N", "сделай следующую задачу", "го след задачу", "го след таску",
  "текущая задача", "current task", "продолжай задачу", "continue task",
  "разбери ретро-инбокс", "task tracker help". Works from rep.config.json and tasks/_policies
  files and uses the feature dossier + state-artifact flow.
---

# Task Tracker

## Purpose

Single entry point for delivery after project bootstrap:
1. Start new work from a feature dossier.
2. Fill `Spec` from raw description when needed.
3. Decompose dossier/spec into tasks.
4. Execute tasks with state, testing, retro, and resume artifacts.
5. Maintain `local` or `local+linear` sync without using chat as source of truth.

## Project Policy Set

`task-tracker` works from one explicit project policy set resolved inside the selected project root:

1. `rep.config.json`
2. `tasks/_policies/dev-plan.md`
3. `tasks/_policies/arch-patterns.md`
4. `tasks/_policies/project-patterns.md`
5. project-local runtime templates from `paths.*_template_file`

Meaning of `project policy set` in this skill:
- `rep.config.json` owns machine-readable paths, naming, sync mode, status model, and defaults;
- `tasks/_policies/dev-plan.md` owns the process contract:
  - artifact model
  - feature/task flow and statuses
  - simple-vs-feature gate
  - dossier/state requirements
  - testing/retro/finish rules
  - sync behavior for `local` / `local+linear`
  - link and language rules for Linear/task artifacts
  - multi-agent execution policy
  - logging / blocked / required-task-set rules
- `tasks/_policies/arch-patterns.md` and `tasks/_policies/project-patterns.md` own technical constraints;
- project-local templates own runtime artifact shape.

Architecture traceability rule:
- for every non-trivial feature, the dossier must explicitly record which `AP-*` and `PP-*` rules are in scope;
- task decomposition must carry that traceability forward so each created task either:
  - lists the relevant `AP-*` / `PP-*` it must obey, or
  - explicitly states that no additional feature-specific architecture rules apply beyond the baseline policy set;
- do not rely on "policy set was loaded" as a substitute for writing the relevant rules into feature/task artifacts.

Load order:
1. resolve `target project root`;
2. read `rep.config.json`;
3. load the policy files named above;
4. load project-local templates from `rep.config.json`;
5. only then build previews, pick tasks, or mutate artifacts.

If any required part of the project policy set is missing or invalid, stop and route the user to `project-tracker`.

## Ownership Split

`task-tracker` is not the owner of project process policy.

`dev-plan.md` owns:
- what artifacts exist;
- what statuses and gates mean;
- when testing/retro/finish are allowed;
- how `local+linear` behaves;
- what formatting/language rules apply to task and Linear artifacts;
- what multi-agent execution is allowed.

`task-tracker` owns:
- command entrypoints (`new task`, `current task`, `continue task`, explicit task execution);
- read order and recovery from artifacts;
- preview/confirmation protocol;
- execution orchestration after confirmation;
- working-brief format shown to the user;
- stopping and escalating when policy is missing, invalid, or cannot be satisfied safely.

Rule:
- when `task-tracker` needs a rule meaning, it must read it from the resolved project policy set instead of restating or redefining it locally.

## Core commands

### `новая задача / new task`

Default entry point.

1. Read project config and policies.
2. Create or resolve a feature key.
3. Build a feature dossier preview first.
4. Apply simple-vs-feature gate.
5. Show proposed path:
   - tiny/simple change
   - non-trivial feature
6. Do not write until explicit confirmation.

Core rule:
- the first local artifact for any new request is always the feature dossier;
- that dossier is the persistent working file for the request and is filled in over time as `Context`, `Spec`, `Plan`, `Decisions`, `Tracking`, and `Log` evolve.

Preview response contract:
- explicitly say that nothing has been created yet;
- explicitly say what will be created after confirmation;
- end with a compact `Reply with one of:` block;
- never leave the user with only an implicit prompt like `если подтверждаешь, создам`.

Required reply options for preview:
- `делай` — accept the proposed path and create local artifacts, then Linear mirror if enabled;
- `делай, но поправь: ...` — accept the flow but change feature key, scope, title, first task, or wording;
- `отмена` — stop without writing anything.

Before confirmation:
- no files are created;
- no Linear mutations are made.

After confirmation:
- always create the feature dossier first;
- before creating executable tasks, resolve and write the architecture traceability set for the feature dossier:
  - relevant `AP-*`
  - relevant `PP-*`
  - any approved deviation that must be referenced instead of silently repeated;
- if tiny/simple path:
  - if the first implementation step is executable now, create one task in `todo`;
  - if a tiny/simple task file is created, create `TASK.state.json` with it;
  - otherwise keep the work in the feature dossier until the first executable task is clear or an explicit `blocked` task is needed;
  - do not create `TASK.testing.md` yet;
  - do not create `TASK.retro.md` yet;
- if feature path:
  - create `FEATURE.state.json`;
  - create or update dossier `Spec` and `Plan`;
  - do not treat dossier `Spec`/`Plan` as ready for decomposition until the relevant architecture rules are written explicitly;
  - create required tasks only when each task is executable enough to own cleanly;
  - during decomposition, every created task must inherit only the relevant subset of feature-level `AP-*` / `PP-*` rules instead of a blind dump of the whole policy catalog;
  - if several dependency-free tasks have disjoint write boundaries, explicitly surface them as a parallel-start package instead of collapsing them into a single implied next task;
  - for every created task also create `TASK.state.json`;
  - update dossier `Tracking` so it remains the human-readable source of truth for:
    - feature anchor issue id/url when `local+linear` is enabled
    - child task keys
    - required task set source of truth
  - if a known stop-factor means a task is not executable yet, keep it in dossier planning or create it directly in `blocked` with `blocked_reason`;
  - do not create testing/retro artifacts before those stages are reached;
  - create one feature anchor issue in Linear if mode is `local+linear`;
  - sync dossier `Tracking`, dossier/spec/plan, and tasks milestones to the feature anchor issue.

## Simple vs feature gate

Use the simple-vs-feature gate from the resolved `tasks/_policies/dev-plan.md`.

Skill responsibility:
- apply that gate before any writes;
- show the chosen path in preview;
- if evidence is ambiguous, ask the user instead of inventing a local reinterpretation.

## Feature dossier contract

Feature dossier schema is owned by the project policy set and project-local templates.

Skill responsibility:
- create the dossier first when the project policy requires it;
- fill and update the sections required by the project policy;
- treat the dossier as the human-readable coordination artifact for the request.

Minimum architecture write gate for dossier design:
- before a feature can move from dossier design into task decomposition, the dossier must name the architecture rules actually governing the feature;
- the expected form is:
  - relevant baseline rules from `arch-patterns.md` as `AP-*`
  - relevant project-local rules from `project-patterns.md` as `PP-*`
  - explicit note when there is no project-local override/deviation;
- if the relevant rules cannot yet be named confidently, stop decomposition and keep the work in dossier/spec clarification instead of inventing tasks.

## State artifacts

### `FEATURE.state.json`

Shape, authority, and update order are owned by `tasks/_policies/dev-plan.md` and the project-local template.

Skill responsibility:
- create/update the file only when the project policy says it should exist;
- never invent extra semantics outside the project policy set.

### `TASK.state.json`

Shape, authority, dependency semantics, blocked semantics, and update order are owned by the project policy set and project-local templates.

Skill responsibility:
- treat `TASK.state.json` as the machine-readable execution source of truth;
- recover execution state from it before chat context;
- rewrite it only according to the project policy set.

Process-surface mutation policy during ordinary task execution:
- do not mutate:
  - `rep.config.json`
  - managed sections in `AGENTS.md`
  - `tasks/_policies/dev-plan.md`
  - `tasks/_policies/arch-patterns.md`
  - `tasks/_templates/*`
- allowed project-level mutations:
  - append-only entries to `tasks/_inbox/retro-inbox.md`
  - `tasks/_policies/project-patterns.md` only through the retro routing flow or when the task itself is explicitly a process-maintenance task
- if execution reveals a bad base policy or template, route it into retro or a follow-up instead of editing it inline.

## Task execution

### Triggers
- `сделай задачу N`
- `сделай PREFIX-N`
- `сделай следующую задачу`
- `го след задачу`
- `го след таску`
- `текущая задача / current task`
- `продолжай задачу / continue task`

### Resolve task

1. Read config and local artifacts first.
2. Resolve explicit task if user named one.
3. For “next task”:
   - consider only tasks in `todo`;
   - treat a dependency as resolved only when the referenced task has `status=done`, regardless of `resolution`;
   - choose only tasks whose `depends_on_task_keys` are all resolved by that rule;
   - exclude tasks with unresolved dependencies from auto-pick;
   - if exactly one executable task exists, auto-pick it;
   - if several executable tasks exist and their reserved write scopes are disjoint, do not silently collapse them into one next task; show them as a parallel-start package and explicitly recommend multi-agent execution;
   - if several executable tasks exist but they are not safe to run in parallel, explain the contention and then choose or ask for a single task;
4. Scope rule:
   - if current task belongs to a feature folder, prefer that feature scope;
   - explicit feature/task in user request wins over auto-scope.

### Resume commands

`текущая задача / current task` and `продолжай задачу / continue task` must recover state from artifacts, not chat.

Resume order:
1. `TASK.state.json`
2. task file
3. feature dossier
4. `TASK.testing.md`
5. `TASK.retro.md`

If several tasks are active:
- auto-select only when exactly one task is active;
- otherwise show candidates ordered by:
  - `in_progress`
  - `testing`
  - `need_changes`
  - `need_retro`
  - `blocked`
- within the same status prefer most recently updated `TASK.state.json`.

If several dependency-free `todo` tasks are executable now:
- group them by safe parallel lanes when reserved write scopes are disjoint;
- explicitly say which tasks can start in parallel and which remain sequential or blocked by shared ownership;
- when the project/feature already allows multi-agent execution, recommend the parallel package as the first-class path instead of only a singular "next task".

Migration rule for upgraded projects:
- do not invent dependencies from general prose;
- if explicit task-key evidence exists, populate `depends_on_task_keys`;
- otherwise leave it empty;
- if a prose-only dependency graph is important for execution order, treat it as a blocking upgrade follow-up until it is converted.

### Execution gate

Before explicit `делай`:
- no code changes;
- no status updates.

After explicit `делай`:
- move task to `in_progress`;
- append task log;
- keep updating `TASK.state.json` and task log after meaningful blocks.

### Testing gate

Testing policy is owned by the resolved `tasks/_policies/dev-plan.md`.

Skill responsibility:
1. move the task into the testing stage defined by project policy;
2. create or update the testing artifact required by the project template;
3. resolve commands in the order defined by project policy;
4. allow only policy-permitted exits from testing.

### Retro gate

Retro policy is owned by the resolved `tasks/_policies/dev-plan.md`.

Skill responsibility:
1. execute retro as part of close-out when the project policy requires it;
2. create or update the retro artifact required by the project template;
3. append inbox proposals when the project policy requires them;
4. close the task only after the policy-defined retro conditions are satisfied.

Human async review command:
- `разбери ретро-инбокс / review retro inbox`

Routing options:
- `в project-patterns / apply to project-patterns`
- `создать follow-up task / create follow-up task`
- `кандидат в upstream / upstream candidate`
- `отложить / defer`
- `отклонить / reject`

## Feature finish

Feature finish policy is owned by the resolved `tasks/_policies/dev-plan.md`.

Skill responsibility:
- check finish conditions against the project policy set before moving a feature to `finish`;
- if the policy conditions are not yet satisfied, surface the missing items explicitly.

## Multi-agent protocol

Multi-agent policy is owned by the resolved `tasks/_policies/dev-plan.md`.

Skill responsibility:
- detect when the project policy allows or prefers parallel execution;
- present the safe parallel-start package to the user instead of collapsing it into one implied next task;
- keep coordination artifacts aligned with the project policy set.

## Write gates

### Creation commands
Before explicit confirmation:
- no file writes;
- no Linear mutations.

After confirmation:
- write local artifacts first in `local`;
- in `local+linear`, create/sync Linear records only after the local artifact set is well-defined;
- if Linear mutation fails in `local+linear`, do not leave partially-created mirrored state without reporting the gap.

### Execution commands
Before `делай`:
- no code changes

After `делай`:
- local state/artifacts update first;
- Linear comments/status mirror last.

## What to show in the working brief

When preparing execution, always include:
1. task key and title;
2. canonical artifact references formatted according to the resolved project policy set;
3. current status and next gate;
4. last meaningful log lines;
5. applied rules from `arch-patterns` / `project-patterns`;
6. requirements from feature dossier;
7. testing expectations;
8. risks and open questions;
9. if relevant, explicit separation between:
   - tasks that can start now in parallel
   - tasks that must stay sequential
10. if relevant, a direct recommendation to launch the parallel package with multi-agent execution;
11. confirmation prompt: `делай?`

When showing a creation/decomposition preview instead of an execution brief, always include:
1. proposed path (`tiny/simple` or `feature`);
2. exact artifact references that will be created after confirmation, formatted according to the resolved project policy set;
3. explicit statement that nothing has been created yet;
4. explicit statement whether Linear will be touched after confirmation;
5. the architecture traceability plan:
   - which `AP-*` / `PP-*` will be written into the dossier before decomposition
   - whether any created tasks will need their own narrower rule subsets
6. reply options block:
   - `делай`
   - `делай, но поправь: ...`
   - `отмена`
