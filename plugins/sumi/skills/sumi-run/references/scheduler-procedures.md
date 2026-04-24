# Scheduler Procedures

Canonical reference for how to create / list / update / delete a scheduled summary task in each supported runtime. The manager skill reads this file; the run skill doesn't need it.

Every runtime has a native scheduling primitive as of 2026 — this plugin never touches OS-level cron / launchd / Task Scheduler. The native primitive is preferred because it integrates with the runtime's own auth, connectors, permissions, notifications, and run-history UI.

## Scheduler-handle schema

Stored inside each task in `config.json` under `task.schedule.handle`. The handle shape varies per runtime — the `type` field discriminates.

```json
{
  "type": "claude-code-cron | claude-code-routine | claude-desktop-task | codex-automation | manual",
  "identifier": "<runtime-native ID or task slug>",
  "runtime": "claude-code | claude-desktop | codex | cursor | cline | continue | other",
  "created_at": "ISO-8601"
}
```

`type: manual` means the runtime has no programmatic scheduler and the user was given UI instructions to create one by hand. In that case, `identifier` is informational (e.g. the cron expression displayed to the user) and deletion is also manual.

---

## Claude Code CLI

**Primary primitives:**

| Cadence | Use | Why |
|---|---|---|
| `daily` / `weekly` / `monthly` / anything ≥ 1h and durable | Routines (cloud) | Persistent, survives laptop close, max 1 resource. Created via `/schedule` or natural language. |
| Short custom cron (<1h interval) or short-lived polling | `CronCreate` | Session-scoped with 7-day expiry. Only suitable when user wants something ephemeral or has intervals CronCreate supports that Routines doesn't. |

**Create (Routine):** in the current session, instruct the runtime: *"Create a routine named `summary-<task-id>` that runs `/sumi-run <task-id>` with cron `<cron>` in timezone `<tz>`. Use default connectors."*

Store in handle: `type: "claude-code-routine"`, `identifier: <routine name or ID echoed back by the runtime>`.

**Create (CronCreate fallback):** call the `CronCreate` MCP tool with `{cron, prompt: "/sumi-run <task-id>", recurring: true}`. Capture the 8-char job ID.

Store in handle: `type: "claude-code-cron"`, `identifier: <8-char job ID>`.

**List:** use `CronList` (session tasks) and `/schedule list` for routines.

**Delete:** for Routines, instruct the runtime to delete by name. For CronCreate, `CronDelete <id>`.

**Gotcha:** CronCreate tasks expire 7 days after creation. If the user picks CronCreate and the task outlives a week, the run stops firing. The manager skill should warn on CronCreate selection and recommend a Routine for durable tasks.

---

## Claude Desktop

**Primitive:** Desktop scheduled tasks (stored under `~/.claude/scheduled-tasks/<task-name>/SKILL.md` with the schedule metadata held in the app's state).

**Available on:** macOS, Windows. (Not Linux.)

**Create:** inside any Desktop session, instruct the runtime in natural language:

> "Create a Desktop scheduled task named `summary-<task-id>` with frequency `<daily|weekdays|weekly|hourly|manual|custom cron>`. When it runs, send this prompt: `/sumi-run <task-id>`. Working folder: use the plugin's install dir. Permission mode: auto-approve tools already allowed in this session."

Desktop will create the task and register the schedule. Capture the task name the runtime reports back.

Store in handle: `type: "claude-desktop-task"`, `identifier: "summary-<task-id>"`.

**List:** ask the runtime "show my scheduled tasks" in any Desktop session, or browse `~/.claude/scheduled-tasks/`.

**Delete:** instruct the runtime: "delete the `summary-<task-id>` scheduled task."

**Gotcha:** Desktop scheduled tasks only fire while the app is running and the machine is awake. For always-on scheduling, use a Routine (cloud) instead — the skill should explain this to the user if they pick Desktop on a laptop that sleeps often.

**Catch-up:** Desktop auto-runs one catch-up on resume for missed runs in the last 7 days. The run skill is designed to handle that — its rolling window is defined from `now`, not from the "missed" scheduled time, so catch-up runs produce a reasonable draft for the current period.

---

## Codex (app)

**Primitive:** Codex Automations — standalone (fresh run each time) or thread (wakes the same conversation). Configurable with cron syntax.

**Create:** inside any Codex thread, instruct the runtime in natural language:

> "Create a standalone Codex automation named `summary-<task-id>` with cron `<cron>` in timezone `<tz>`. When it fires, run: `run summary <task-id>`."

Codex's runtime handles creation (this is documented behavior — skills can create and update automations). Capture the identifier Codex echoes back.

Store in handle: `type: "codex-automation"`, `identifier: "summary-<task-id>"`.

**Use standalone over thread automation.** Standalone runs get a fresh context and a clean draft each period. Thread automations preserve the previous run's context and tend to accumulate clutter across weeks.

**List:** ask the runtime "list my automations" or browse the app's Automations panel.

**Delete:** instruct the runtime: "delete the `summary-<task-id>` automation."

**Gotcha:** Codex CLI (as opposed to the Codex app) does not currently have first-class scheduling; the automation primitive is an app-level feature. If setup was invoked from Codex CLI, the skill should either (a) redirect the user to the Codex app to complete scheduling, or (b) fall back to "manual" scheduling and tell the user how to paste the cron/prompt into Codex's UI.

---

## Cursor

**Primitive:** Cursor Automations (UI-only creation as of April 2026).

**Create:** the skill cannot create a Cursor Automation programmatically. Instead, the skill prints step-by-step instructions the user follows in the Cursor web UI:

> 1. Open [cursor.com/automations](https://cursor.com/automations) (or Cursor IDE → Automations)
> 2. Click **New Automation**
> 3. Trigger: **Cron schedule** → paste `<cron>` (timezone: `<tz>`)
> 4. Prompt: paste exactly: `run summary <task-id>`
> 5. Tools: allow whatever MCP/Slack/Gmail/Drive tools the task needs
> 6. Save

Store in handle: `type: "manual"`, `runtime: "cursor"`, `identifier: "cron=<cron>;prompt=run summary <task-id>"`.

**Update / delete:** user does this in the Cursor UI. The skill's edit-task and delete-task flows print the equivalent instructions.

---

## Cline / Continue / other IDE extensions

**No native scheduler primitive for this use case.** Cline's parallel-agents feature and Continue's CI/CD background agents are not cron-style schedulers.

**Fallback:** `type: "manual"`. Tell the user plainly: "This runtime doesn't have a native scheduler. Your task is saved; invoke with `run summary <task-id>` when you want it, or install the plugin in Claude Desktop or Codex on the same machine (cross-runtime symlinks share the config), where scheduling works automatically."

---

## Decision tree (manager skill uses this at step 3i)

```
If runtime is Claude Code CLI:
  If cadence ∈ {daily, weekly, monthly} OR interval ≥ 1h:
    → Create Routine (recommend; explain "runs in the cloud, persistent")
    If user prefers local → Create Desktop task (if Desktop installed) OR CronCreate with 7-day expiry warning
  Else (sub-hour custom interval):
    → CronCreate, warn about 7-day expiry

If runtime is Claude Desktop:
  → Create Desktop scheduled task (natural-language to runtime)
  If cadence ≥ 1h AND user wants always-on → offer Routine as alternative

If runtime is Codex app:
  → Create standalone Codex Automation (natural-language to runtime)

If runtime is Codex CLI:
  → Instruct user to open Codex app to finish scheduling, OR manual

If runtime is Cursor:
  → Print UI instructions, store as manual

If runtime is Cline / Continue / other:
  → Manual. Suggest installing in a runtime with scheduling (Desktop, Codex app).
```

## Edit and delete

All edit/delete operations go through the same runtime-specific primitive. The manager skill reads `task.schedule.handle.type` to pick the path.

For `type: "manual"`, edit/delete prints UI instructions for the user rather than automating anything.
