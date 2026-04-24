---
name: sumi-run
description: Run one summary task by ID. Invoked by scheduled triggers (Routines, Desktop scheduled tasks, Codex Automations) and by users who want to fire a task on demand.
argument-hint: "<task-id>"
---

Invoke the `sumi-run` skill at `skills/sumi-run/SKILL.md` with `$ARGUMENTS` as the task ID. If no argument and only one task exists, use it. If multiple tasks and none specified, list them and ask which one.
