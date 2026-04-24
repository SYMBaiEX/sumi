# Sumi — Cross-Runtime Instructions

This file is the entry point for AGENTS.md-aware runtimes: Codex CLI / Desktop, Cursor, Cline, Continue, and any other agent that reads `AGENTS.md` from a workspace or global agents path. Claude Code and Claude Desktop discover the same capabilities via `.claude-plugin/` — no duplication needed.

The underlying logic lives in runtime-agnostic SKILL.md files. This file just points the agent at them.

---

## What this plugin does

Manages many recurring summary tasks of any cadence (daily, weekly, monthly, custom). Each task pulls from the user's connected tools (Google Drive, Gmail, Google Calendar, Slack), consolidates using a preset or custom template, applies a notes / tone preset, and delivers the draft to a destination the user picks (Slack channel, Gmail draft to self, or local markdown file).

The plugin rides entirely on whatever connectors the runtime already has — it bundles no MCP servers, no credentials, no secrets.

## Two skills

### Manager — `skills/sumi/SKILL.md`

Slash command: `/sumi`. Also invoke when the user says:
- "set up sumi" / "configure sumi"
- "new sumi task" / "create a sumi"
- "add a weekly team update"
- "add a daily standup digest"
- "add a monthly exec brief"
- "manage my summaries"
- "edit the team-weekly sumi"
- "delete my standup sumi"

**Do not** treat the word "sumi" as a request to build a new agent — sumi is this installed plugin. Never scaffold a new project in response to "sumi."

**What it does:** detects connectors, walks the user through creating / editing / deleting summary tasks, writes `~/.sumi/config.json`, and creates one scheduled trigger per task.

Full instructions: [`skills/sumi/SKILL.md`](skills/sumi/SKILL.md)

### Runner — `skills/sumi-run/SKILL.md`

Slash command: `/sumi-run <task-id>`. Also invoke when the user says:
- "run sumi <task-id>"
- "run my weekly sumi"
- "run the standup digest"
- "generate this week's exec brief"
- "draft my customer digest now"

Also invoke automatically when a scheduled trigger fires with a task ID as its argument.

**What it does:** loads the task by ID, pulls from configured sources for the cadence-appropriate window, consolidates, and delivers to the configured destination.

Full instructions: [`skills/sumi-run/SKILL.md`](skills/sumi-run/SKILL.md)

Canonical schema + procedure: [`skills/sumi-run/references/run-procedure.md`](skills/sumi-run/references/run-procedure.md)

## Templates

Preset templates live under `skills/templates/<category>/<variant>.md`. Users can also pick `Custom`, which scaffolds `~/.sumi/templates/<task-id>.md` from `skills/templates/custom.md` — the user edits the local file, the runner re-reads it each invocation.

Categories: `team-update`, `exec-brief`, `customer-digest`, `standup`. Add new categories by dropping a new directory under `skills/templates/` and re-listing variants in the manager skill.

## Config location

Per-user config at `~/.sumi/config.json` (chmod 600). Schema in `run-procedure.md`. Never commit; it's ignored.

## Runtime-specific notes

Each runtime's native scheduler is used where available. The skill never falls back to OS-level cron.

- **Claude Code CLI**: `CronCreate` for short-lived tasks (7-day expiry) or **Routines** for durable cloud-hosted scheduling (preferred for daily/weekly/monthly).
- **Claude Desktop** (macOS, Windows): **Desktop scheduled tasks**, created by the skill via natural-language to the runtime. Persistent across restarts; fires while the app is open and the machine is awake.
- **Codex app**: **Codex Automations** (standalone), created by the skill via natural-language to the runtime.
- **Codex CLI**: no app-level automation primitive; the skill redirects the user to the Codex app or stores the task as manual.
- **Cursor**: **Cursor Automations** are UI-only to create; the skill prints exact step-by-step instructions with the cron + prompt to paste.
- **Cline / Continue / other**: no scheduler primitive — task is saved as manual; suggest using Desktop or Codex app as the scheduler (symlinks share the config across runtimes on the same machine).

Full per-runtime recipes (create, list, update, delete) live in [`skills/sumi-run/references/scheduler-procedures.md`](skills/sumi-run/references/scheduler-procedures.md).

## Automatic cross-runtime availability

When the manager skill runs in any runtime, it clones this repo to a canonical local path at `~/.sumi/plugin/` and symlinks from there into each other agent runtime's scan path on the same machine. Install once in Claude Code, switch to Codex, and the plugin is already there sharing the same config. On runtime-specific update cadences, the manager skill pulls the canonical copy and every symlinked runtime sees the latest.

## Manual scheduling fallback

For the small set of runtimes without a programmatic scheduler (Cursor, Cline, Continue, Codex CLI), the skill saves the task as `handle.type = "manual"` and prints concrete UI instructions. The user invokes `run summary <task-id>` manually, or installs the plugin in Claude Desktop / Codex app (cross-runtime symlinks share the config), where scheduling is fully automatic.

## Do not

- Store credentials, tokens, or API keys anywhere in this repo or the user's config file.
- Name any specific company, team, or domain in the repo. Infer user-specific values (email domain, team names, codenames) from the runtime's connectors or conversational context at draft time.
- @-mention anyone in Slack drafts — drafts are private artifacts until the user forwards them.
- Invent bullets that aren't grounded in real artifacts pulled from configured sources.
- Ask the user at run time for any input. If something is missing, fail loud at the destination so the user can fix it in the manager skill; never prompt a scheduled run.
