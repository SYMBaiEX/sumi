# Summary Agent

Manage many recurring summary tasks — daily, weekly, monthly, or any custom cadence — drawn from the tools you're already connected to, delivered where you want to review them.

Create a standup digest that runs every morning. A team update that posts every Friday. A monthly exec brief. A weekly customer pulse. Each task is independent: its own cadence, its own sources, its own template, its own destination.

## How it works

1. **Install** the plugin in your agent runtime (Claude Code, Claude Desktop, Codex, Cursor, Cline, Continue — any that read `AGENTS.md` or `.claude-plugin/`).
2. **Say "set up summary agent"**. The manager skill walks you through creating your first task: name, cadence, sources, template, tone/notes preset, delivery destination.
3. **The run skill fires** on your configured schedule, pulls from the tools you picked, consolidates using your template, and drops the draft where you said to deliver it.
4. **You review the draft** — edit, forward, ignore, regenerate. The plugin never sends anything on your behalf.

Later, say "set up summary agent" again to create another task, edit an existing one, or delete one. You can have as many tasks as you want, each on its own cadence.

## Zero-config MCP, zero secrets

The plugin **does not bundle any MCP servers, tokens, or credentials**. It rides on whatever connectors your runtime already has — Google Drive, Gmail, Google Calendar, Slack. If a connector isn't enabled, the related option simply isn't offered. When an example needs a user-specific value (like your work email domain), the plugin asks the connector at the moment it's needed and substitutes it in — nothing personal is ever stored in the plugin itself.

This is why the plugin is safe to publish publicly and share across any company — nothing company-specific lives in the code.

## Install

Installing in any one of your agent runtimes makes the plugin available across every other agent runtime on your machine (except Claude, which manages its own cache). Do this once in whichever runtime you use first.

### Claude Code / Claude Desktop (easiest)

```
/plugin marketplace add SYMBaiEX/summary-agent
/plugin install summary-agent@summary-agent
set up summary agent
```

### Codex CLI / Codex Desktop

```bash
git clone https://github.com/SYMBaiEX/summary-agent.git ~/.codex/summary-agent
```

Then in Codex: `set up summary agent`.

### Cursor / Cline / Continue / any AGENTS.md-aware agent

Clone the repo into the runtime's global agents path or add as a workspace, then invoke `set up summary agent` and `run summary <task-id>` as natural-language commands. The `AGENTS.md` at the repo root is the entry point.

## Use

| What you want | What you say |
|---|---|
| Create your first / another task | `set up summary agent` |
| List, edit, delete, or run tasks | `set up summary agent` (shows manager menu when tasks exist) |
| Run a specific task now | `run summary <task-id>` |
| Run "the" summary (when only one exists) | `run my summary` |

## Sources

Pick any combination at setup time:

| Source | Good for |
|---|---|
| **Google Drive** | Project plans, weekly docs, design specs you update |
| **Gmail** | Customer signal, external mail, anything you star or label |
| **Google Calendar** | Meetings held + upcoming commitments |
| **Slack** | Channel activity, decisions, blockers — if Slack is connected |

Multi-select. At least one source per task.

## Templates

Preset two-level picker plus a Custom scaffold.

| Category | Variants |
|---|---|
| **team-update** | ic, team-lead, cross-functional |
| **exec-brief** | weekly, monthly |
| **customer-digest** | support, success |
| **standup** | daily-async, weekly-recap |
| **Custom** | Scaffolds a blank template at `~/.claude-summaries/templates/<task-id>.md` that you edit to taste |

Templates are read fresh each run, so edits take effect immediately.

## Notes / tone presets

Pick one per task — steers tone and emphasis without changing structure:

- Emphasize numbers and metrics
- Casual tone, short bullets
- Focus on customer quotes
- Highlight cross-team dependencies
- Strict executive brevity
- No notes / just the facts
- Other (custom freeform text)

## Destinations

Pick one per task:

1. **Slack channel** — if Slack is connected. Usually a private channel or DM-to-self.
2. **Gmail draft to yourself** — unsent draft in your inbox. Open Friday morning, review, forward.
3. **Local markdown file** — `~/.claude-summaries/drafts/<task-id>/YYYY-MM-DD.md`. Always-available fallback.

Smart default based on what you have connected; you can always override.

## Where things live

| Path | What |
|---|---|
| `~/.claude-summaries/config.json` | Your task list (chmod 600). |
| `~/.claude-summaries/plugin/` | Canonical git clone used for cross-runtime symlinks. |
| `~/.claude-summaries/drafts/<task-id>/` | Local drafts (if you chose the file destination). |
| `~/.claude-summaries/templates/<task-id>.md` | Your edited custom template (if you chose Custom). |

Nothing leaves your machine beyond the connector calls your runtime already makes.

## Platforms

| Runtime | Skills | Autonomous scheduled runs |
|---|---|---|
| Claude Code | ✅ | ✅ via native scheduled triggers |
| Claude Desktop | ✅ | ❌ manual — say "run summary \<id\>" when you want it |
| Codex CLI / Desktop | ✅ via AGENTS.md | ❌ manual — or wire a shell cron (example in `AGENTS.md`) |
| Cursor / Cline / Continue | ✅ via AGENTS.md | ❌ manual |
| Any MCP-capable agent | ✅ — follow the SKILL.md files directly | depends on runtime |

## Privacy

- Nothing leaves your machine except the connector calls your runtime already makes.
- No data is stored outside `~/.claude-summaries/` (identifiers + queries, never message bodies or doc contents).
- Draft is delivered only to the destination you configured. The plugin never @-mentions anyone or posts elsewhere.

## Sharing with teammates

Teammates install the same way, answer setup prompts using *their* connectors and *their* permissions. No shared tokens, no service accounts, no workspace-admin coordination beyond what each teammate's own Slack/Google policies already require.

## Customizing

- **Template**: edit the preset at `skills/templates/<cat>/<var>.md` in the plugin dir (changes on your machine), or pick Custom at setup time to get your own editable copy at `~/.claude-summaries/templates/<task-id>.md`.
- **Notes guidance**: the mapping from preset key → guidance string lives in `skills/summary-run/references/run-procedure.md`. Edit locally if you want different tuning.

## Related

- **[SYMBaiEX/weekly-update](https://github.com/SYMBaiEX/weekly-update)** — the single-purpose plugin that preceded this one. Still works, still maintained. Use that if you only need one weekly update; use `summary-agent` if you want multiple summary tasks of varying cadences.
