# Sumi

> **/sumi** — manage many recurring summary tasks at any cadence, drawn from the tools you're already connected to, delivered where you want to review them. Uses each runtime's native scheduler so scheduled runs are fully autonomous.

Create a standup digest that runs every morning. A team update that posts every Friday. A monthly exec brief. A weekly customer pulse. Each task is independent: its own cadence, sources, template, notes, and delivery destination.

## How to use

- **`/sumi`** — open the manager. Creates your first task or shows the manager menu (create / edit / delete / run / list) if tasks exist.
- **`/sumi-run <task-id>`** — run a task immediately. Also what scheduled triggers invoke.

You can also say things naturally: "set up sumi", "new sumi task", "run the weekly sumi", "delete my standup sumi". The slash commands are the direct path and avoid any confusion with the word "agent."

## Install

Installing in any one of your agent runtimes makes the plugin available across every other agent runtime on your machine (except Claude, which manages its own cache). Do this once in whichever runtime you use first.

### Claude Code / Claude Desktop

```
/plugin marketplace add SYMBaiEX/sumi
/plugin install sumi@sumi
/sumi
```

### Codex CLI / Codex Desktop

Codex has its own plugin marketplace that reads `.codex-plugin/` + `.agents/plugins/marketplace.json`. Sumi ships both alongside the Claude Code manifest, so the standard Codex install flow works.

Start a Codex session first, then run these **as slash commands inside the session** (not as shell commands):

```
codex
```

Then inside Codex:

```
/plugin marketplace add SYMBaiEX/sumi
/plugins
```

`/plugins` opens the plugin directory — switch to the **Sumi** marketplace tab and install sumi. Once installed, type `/sumi` to set up.

**Alternative (if the marketplace command doesn't find the repo)**: place a personal marketplace pointer at `~/.agents/plugins/marketplace.json` that references this repo, or clone directly into the global skills path:

```bash
git clone https://github.com/SYMBaiEX/sumi.git ~/.codex/skills/sumi
```

Codex's global skills loader will discover the SKILL.md files at `~/.codex/skills/sumi/skills/` on next session. Then `/sumi` invokes the manager.

### Cursor / Cline / Continue / any AGENTS.md-aware agent

Clone the repo into the runtime's global agents path or add as a workspace, then invoke `/sumi` or `/sumi-run <id>`. The `AGENTS.md` at the repo root is the entry point.

## Zero-config MCP, zero secrets

The plugin **bundles no MCP servers, no tokens, no credentials**. It uses whatever connectors your runtime already has — Google Drive, Gmail, Google Calendar, Slack. If a connector isn't enabled, the related option simply isn't offered. User-specific values (like your work email domain) are inferred from the connector at the moment they're needed, never stored.

This is why sumi is safe to publish publicly and share across any company — nothing company-specific lives in the code.

## Native scheduling per runtime

Every major runtime ships a native scheduler as of 2026. Sumi uses the runtime's own primitive — no OS-level cron, no launchd, no Task Scheduler. Scheduled runs integrate with the runtime's auth, connectors, permissions, notifications, and run-history UI.

| Runtime | Native scheduler sumi uses |
|---|---|
| Claude Code CLI | **Routines** (cloud, durable) or `CronCreate` (session-scoped, 7-day) |
| Claude Desktop (macOS, Windows) | **Desktop scheduled tasks** (fires while app is open) |
| Codex app | **Codex Automations** (standalone) |
| Codex CLI | redirects to Codex app for scheduling, or saves as manual |
| Cursor | prints exact paste-in steps for **Cursor Automations** UI |
| Cline / Continue / other | manual — suggests using Desktop / Codex app alongside (config shared via symlinks) |

Full per-runtime scheduler recipes (create / list / update / delete) in [`skills/sumi-run/references/scheduler-procedures.md`](skills/sumi-run/references/scheduler-procedures.md).

## Sources

Pick any combination at setup time:

| Source | Good for |
|---|---|
| **Google Drive** | Project plans, weekly docs, design specs |
| **Gmail** | Customer signal, external mail, anything you star or label |
| **Google Calendar** | Meetings held + upcoming commitments |
| **Slack** | Channel activity, decisions, blockers — if Slack is connected |

Multi-select. At least one source per task.

## Templates

Two-level preset picker plus a Custom scaffold.

| Category | Variants |
|---|---|
| **team-update** | ic, team-lead, cross-functional |
| **exec-brief** | weekly, monthly |
| **customer-digest** | support, success |
| **standup** | daily-async, weekly-recap |
| **Custom** | Blank scaffold at `~/.sumi/templates/<task-id>.md` you edit to taste |

Templates are read fresh each run — edits take effect immediately.

## Notes / tone presets

Pick one per task. Steers tone and emphasis without changing structure:

- Emphasize numbers and metrics
- Casual tone, short bullets
- Focus on customer quotes
- Highlight cross-team dependencies
- Strict executive brevity
- No notes / just the facts
- Other (custom freeform text)

## Destinations

- **Slack channel** — if Slack is connected. Usually a private channel or DM-to-self.
- **Gmail draft to yourself** — unsent draft in your inbox.
- **Local markdown file** — `~/.sumi/drafts/<task-id>/YYYY-MM-DD.md`. Always-available fallback.

Smart default based on what's connected; override anytime.

## Where things live

| Path | What |
|---|---|
| `~/.sumi/config.json` | Your task list (chmod 600). |
| `~/.sumi/plugin/` | Canonical git clone used for cross-runtime symlinks. |
| `~/.sumi/drafts/<task-id>/` | Local drafts (if you chose the file destination). |
| `~/.sumi/templates/<task-id>.md` | Your edited custom template (if you chose Custom). |

Nothing leaves your machine beyond the connector calls your runtime already makes.

## Privacy

- No data stored outside `~/.sumi/` (identifiers + queries only, never message bodies or doc contents).
- Draft delivered only to the destination you configured. Never @-mentions, never posts elsewhere.

## Sharing with teammates

Teammates install the same way, answer setup prompts using *their* connectors and *their* permissions. No shared tokens, no service accounts, no workspace-admin coordination beyond what each person's own tool policies already require.

## Related

- **[SYMBaiEX/weekly-update](https://github.com/SYMBaiEX/weekly-update)** — the single-purpose weekly-update plugin that preceded sumi. Still works. Use it if you only need one weekly update; use sumi if you want multiple tasks at varying cadences.
