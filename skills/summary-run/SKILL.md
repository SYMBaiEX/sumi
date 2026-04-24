---
name: summary-run
description: Executes one configured summary task by ID. Invoked automatically by a scheduled trigger (cron) or manually when the user says "run summary <task-id>", "run my weekly summary", "run the team update", "generate my daily standup", "run the exec brief now", "draft this week's customer digest", or any variant referencing a specific summary task. The task ID is passed as an argument; the skill looks it up in ~/.claude-summaries/config.json, pulls from the configured sources for the cadence-appropriate window, consolidates using the configured template and notes preset, and delivers the draft to the configured destination (Slack / Gmail draft / local file). Do not use this skill before the user has created at least one task — if the config file is missing or empty, defer to the summary-agent skill to set things up first.
---

# Summary Run

Execute one configured summary task. Designed to run unattended on a schedule — assume no human is watching. Fail loud (visible error at the configured destination) rather than silently.

Full procedure lives at [`references/run-procedure.md`](references/run-procedure.md). Read that file for the complete step-by-step — the run logic lives there so this skill body stays scannable.

## Entrypoint

The task ID comes in as an argument (`$ARGUMENTS` when invoked by a scheduled trigger, or from the user's natural-language request).

1. Read `~/.claude-summaries/config.json`.
2. Locate `tasks[]` entry where `id === <task-id>`. If not found: write a "task not found" error to the default destination (Gmail draft-to-self if Gmail is connected, else local file at `~/.claude-summaries/drafts/_errors/YYYY-MM-DDTHH-MM.md`) and stop. Do not silently fail.
3. If the task has `paused: true`: stop without error.
4. Execute the pull → consolidate → deliver procedure described in [`references/run-procedure.md`](references/run-procedure.md).

## Disambiguating when the user is vague

If the user says "run my summary" with no task ID:

- If exactly **one** task exists, use it without asking.
- If **zero** tasks exist, defer to `summary-agent` to set one up.
- If **multiple** tasks exist, list them with display names + cadence labels and ask which one.

## Dry-run behavior

If invoked via "run the weekly summary" / "run summary now" / "dry run the team update", behave identically to a scheduled invocation — post to the real destination. Users treat the destination as a staging area; there's no benefit to redirecting to chat.

## Context hygiene

When drafting, use any relevant context the user has shared in this conversation or prior conversations (codenames, team names, stakeholders, projects, company terminology). Do not ask the user to restate these at run time — they should never see a prompt from a scheduled run.
