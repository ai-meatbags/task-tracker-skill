# task-tracker-skill

Policy-driven task orchestration for coding agents.

`task-tracker` helps an agent move work forward from persistent project artifacts instead of relying on chat history. It is built for teams that want delivery to survive context loss, handoffs, retries, and long-running feature work.

## What It Does

- Starts new work from a feature-level artifact instead of jumping straight into code.
- Decomposes scoped work into executable tasks with explicit state.
- Resumes the current task from files, not from conversation memory.
- Enforces execution, testing, and retro gates through project-owned policy.
- Surfaces parallel-start work when tasks are independent and safe to split.

## Why It Exists

Most agent workflows break down once the conversation gets long, another person takes over, or the implementation pauses for a day. `task-tracker` treats the project itself as the source of truth:

- plans live in artifacts
- execution state lives in artifacts
- testing and close-out live in artifacts
- the agent reads those artifacts before acting

That makes task execution more repeatable and easier to audit.

## Product Model

`task-tracker` is intentionally not a project bootstrapper.

It does not invent your delivery process, folder layout, or artifact semantics. Instead, it reads a project-defined policy layer and uses that contract to decide:

- what artifacts exist
- how tasks are selected
- when testing starts
- when retro is required
- what “done” means
- which work can run in parallel

This makes the skill flexible, but it also means the project must already expose a compatible process contract.

## Best Fit

This skill is useful when you want:

- longer-lived feature work with resumable state
- explicit task artifacts instead of ad hoc chat instructions
- consistent testing and close-out behavior
- multi-agent execution with clear task boundaries
- a delivery workflow that is owned by the project, not hardcoded into the agent

It is a poor fit for:

- tiny one-off edits with no artifact model
- repos that do not maintain structured task state
- teams expecting a zero-config task manager

## Current Contract

The current open-source version expects the target project to already expose:

- a machine-readable project config
- a process policy layer
- task and feature artifacts with persistent state
- templates or conventions for runtime delivery artifacts

If that layer is missing or invalid, `task-tracker` should stop rather than improvising process rules.

## Repository Contents

- `SKILL.md` — the operating contract for the skill

## Important Note

This repository is open source and standalone, but the skill is not fully generic yet.

The current `SKILL.md` still targets a specific policy-driven artifact format. So the honest product position is:

- portable repository
- reusable skill
- not plug-and-play for arbitrary repos without a compatible project contract

If you want broader adoption, the next real product step is not another README pass. It is extracting the policy contract into a more neutral public spec or adding adapters for multiple project layouts.
