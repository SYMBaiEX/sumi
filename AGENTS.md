# Summary Agent — Cross-Runtime Instructions

This file is the entry point for AGENTS.md-aware runtimes: Codex CLI / Desktop, Cursor, Cline, Continue, and any other agent that reads `AGENTS.md` from a workspace or global agents path. Claude Code and Claude Desktop discover the same capabilities via `.claude-plugin/` — no duplication needed.

The underlying logic lives in runtime-agnostic SKILL.md files. This file just points the agent at them.

---

## What this plugin does

Manages many recurring summary tasks of any cadence (daily, weekly, monthly, custom). Each task pulls from the user's connected tools (Google Drive, Gmail, Google Calendar, Slack), consolidates using a preset or custom template, applies a notes / tone preset, and delivers the draft to a destination the user picks (Slack channel, Gmail draft to self, or local markdown file).

The plugin rides entirely on whatever connectors the runtime already has — it bundles no MCP servers, no credentials, no secrets.

## Two skills

### Manager — `skills/summary-agent/SKILL.md`

Invoke when the user says:
- "set up summary agent"
- "create a summary task"
- "add a weekly team update"
- "add a daily standup digest"
- "add a monthly exec brief"
- "manage my summaries"
- "edit the team-weekly summary"
- "delete my standup summary"
- "run my weekly summary now"

**What it does:** detects connectors, walks the user through creating / editing / deleting summary tasks, writes `~/.claude-summaries/config.json`, and creates one scheduled trigger per task.

Full instructions: [`skills/summary-agent/SKILL.md`](skills/summary-agent/SKILL.md)

### Runner — `skills/summary-run/SKILL.md`

Invoke when the user says:
- "run summary team-weekly"
- "run my weekly summary"
- "run the standup digest"
- "generate this week's exec brief"
- "draft my customer digest now"

Also invoke automatically when a scheduled trigger fires with a task ID as its argument.

**What it does:** loads the task by ID, pulls from configured sources for the cadence-appropriate window, consolidates, and delivers to the configured destination.

Full instructions: [`skills/summary-run/SKILL.md`](skills/summary-run/SKILL.md)

Canonical schema + procedure: [`skills/summary-run/references/run-procedure.md`](skills/summary-run/references/run-procedure.md)

## Templates

Preset templates live under `skills/templates/<category>/<variant>.md`. Users can also pick `Custom`, which scaffolds `~/.claude-summaries/templates/<task-id>.md` from `skills/templates/custom.md` — the user edits the local file, the runner re-reads it each invocation.

Categories: `team-update`, `exec-brief`, `customer-digest`, `standup`. Add new categories by dropping a new directory under `skills/templates/` and re-listing variants in the manager skill.

## Config location

Per-user config at `~/.claude-summaries/config.json` (chmod 600). Schema in `run-procedure.md`. Never commit; it's ignored.

## Runtime-specific notes

- **Claude Code / Desktop**: install via `/plugin marketplace add SYMBaiEX/summary-agent` and `/plugin install summary-agent@summary-agent`. Setup creates native Friday/Daily/Monthly scheduled triggers via Claude Code's `schedule` tooling. The manager skill auto-enables marketplace auto-update.
- **Codex CLI / Desktop**: place this repo under a Codex scan path (e.g. `~/.codex/summary-agent`). Invoke with the trigger phrases above. Codex does not have a first-class cron yet — wire a shell-level cron pointing at `codex exec "run summary <task-id>"` if you want autonomous runs.
- **Cursor / Cline / Continue**: clone the repo globally or add as a workspace, then invoke by the trigger phrases. Same manual-scheduling note as Codex.
- **Any MCP-capable agent**: read the two SKILL.md files and follow them directly. The instructions are runtime-agnostic.

## Automatic cross-runtime availability

When the manager skill runs in any runtime, it clones this repo to a canonical local path at `~/.claude-summaries/plugin/` and symlinks from there into each other agent runtime's scan path on the same machine. Install once in Claude Code, switch to Codex, and the plugin is already there sharing the same config. On runtime-specific update cadences, the manager skill pulls the canonical copy and every symlinked runtime sees the latest.

## Scheduling without a native cron

For runtimes lacking native scheduling, use a shell-level cron pointing at a one-shot agent invocation:

```cron
0 9 * * 5  /path/to/codex exec "run summary team-weekly" >/dev/null 2>&1
```

One line per task. The run skill handles everything after — same path as an interactive invocation.

## Do not

- Store credentials, tokens, or API keys anywhere in this repo or the user's config file.
- Name any specific company, team, or domain in the repo. Infer user-specific values (email domain, team names, codenames) from the runtime's connectors or conversational context at draft time.
- @-mention anyone in Slack drafts — drafts are private artifacts until the user forwards them.
- Invent bullets that aren't grounded in real artifacts pulled from configured sources.
- Ask the user at run time for any input. If something is missing, fail loud at the destination so the user can fix it in the manager skill; never prompt a scheduled run.
