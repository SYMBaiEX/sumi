# Run Procedure — shared source of truth

This file is canonical for:
- The task schema stored in `~/.sumi/config.json`
- The pull → consolidate → deliver procedure the `sumi-run` skill follows
- The list of supported notes presets and their drafting-guidance strings

Both `sumi` (manager) and `sumi-run` (runner) read this file so they can't drift. When changing any schema or preset list, update this file; the skills resolve from here.

## Task schema

```json
{
  "id": "team-weekly-update",
  "display_name": "Team Weekly Update",
  "created_at": "2026-04-24T14:00:00Z",
  "paused": false,
  "last_run": {
    "started_at": "ISO-8601",
    "ended_at": "ISO-8601",
    "status": "success | partial | error",
    "skipped_sources": [],
    "error": null
  },
  "schedule": {
    "cadence": "daily | weekly | monthly | custom",
    "cron": "0 9 * * 5",
    "timezone": "America/Chicago",
    "label": "Fridays at 9:00 AM CT",
    "handle": {
      "type": "claude-code-routine | claude-code-cron | claude-desktop-task | codex-automation | manual",
      "identifier": "<runtime-native id or task slug>",
      "runtime": "claude-code | claude-desktop | codex | cursor | cline | continue | other",
      "created_at": "ISO-8601"
    }
  },
  "sources": {
    "drive": [
      { "id": "<folder-id>", "name": "2026 Project Plans" }
    ],
    "gmail": [
      "is:starred newer_than:7d"
    ],
    "calendar": {
      "calendars": ["primary"],
      "include_past": true,
      "include_future": true
    },
    "slack": [
      { "id": "C0123", "name": "product" }
    ]
  },
  "template": {
    "type": "preset",
    "category": "team-update",
    "variant": "team-lead",
    "custom_path": null
  },
  "notes": {
    "preset": "emphasize-numbers",
    "custom_text": null
  },
  "destination": {
    "type": "gmail_draft",
    "channel_id": null,
    "channel_name": null,
    "dir": null
  }
}
```

Only keys for configured sources/destinations need to be present. Minimal valid task has `id`, `display_name`, `schedule`, at least one non-empty source, `template`, `notes`, `destination`.

## Window sizing

The pull window is derived from cadence — do not hardcode "7 days":

| Cadence | Window |
|---|---|
| `daily` | 1 day (previous 24h) |
| `weekly` | 7 days |
| `monthly` | 30 days |
| `custom` | from the cron interval. If irregular, default to 7 days and note in the draft footer. |

Window end = `now` (task's timezone). Window start = `end - window`.

## Pull procedure (parallel where possible)

### Google Drive

For each folder in `sources.drive`:
- List files modified within the window.
- For each modified doc, read the first ~2000 tokens plus any section headings containing "status", "update", "this week", "this period", or "blockers".
- Capture title + URL + owner (if not the user).

If the task was configured with the "modified-across-My-Drive" default, list files in My Drive modified in the window, capped at ~20 by recency.

### Gmail

For each query in `sources.gmail`:
- If the query contains `newer_than:Nd`, rewrite `N` to match the window size so cadence changes propagate without reconfiguring.
- Execute the query. Pull subject, from, date, thread ID, first ~500 tokens of the most recent message body.
- De-duplicate by thread; keep the most recent message.
- Capture thread permalink.

### Google Calendar

If `sources.calendar` is present:
- **include_past** → list events on the selected calendars in the past window. Filter out declined events and events where the user was just an attendee on a large (10+) meeting with no notable outcome signal. Keep 1:1s, smaller meetings, events the user organized, and events marked important. Capture title, attendees (if <8), and any description that looks like decisions/action items.
- **include_future** → list events in the next window. Capture title + attendees. These surface as upcoming commitments.

### Slack

Only if `sources.slack` is present and the Slack connector is active. For each channel:
- Fetch messages in the window.
- Keep: messages with 3+ reactions, threads with 5+ replies, messages from the user or their direct reports, messages containing decision/blocker keywords (`decided`, `blocker`, `shipped`, `launched`, `postponed`, `at risk`, `p0`, `p1`, `rolled back`, `incident`).
- Drop: bot noise, pure emoji replies, reactions without content.
- Capture permalinks for the top ~5 items per channel.

### Connector failures

Any source failure isolates: log into `errors[]`, skip that source, continue. Record skipped sources in `last_run.skipped_sources` and include a footer in the draft like `_Skipped: Slack (connector not authenticated)_`.

## Template resolution

If `template.type === "preset"`:
- Read `skills/templates/<category>/<variant>.md` from the plugin's install directory.

If `template.type === "custom"`:
- Read `template.custom_path` (typically `~/.sumi/templates/<task-id>.md`).
- If missing, fall back to `skills/templates/custom.md` and tell the user in the draft footer.

Re-read the template file every run so user edits take effect immediately.

## Notes presets

The `notes.preset` key maps to a short guidance string prepended to the drafting rules. Keep these short — they tune tone and emphasis, not structure.

| Preset key | Guidance string (prepended as a drafting rule) |
|---|---|
| `emphasize-numbers` | "Lead with quantitative changes. Put deltas before descriptions. Drop any bullet that doesn't move a number." |
| `casual-short` | "Casual tone, first person where natural, bullets under 12 words, no corporate formality." |
| `customer-quotes` | "Surface direct customer language verbatim when it appears in sources. Attribute by account, not individual, unless the user explicitly named the person." |
| `cross-team` | "Flag every dependency between teams. Name the owning team on each bullet. Call out blockers that cross team boundaries first." |
| `exec-brevity` | "Executive brevity. No bullet longer than 15 words. Decisions and outcomes only — omit process and activity. One-line TL;DR mandatory." |
| `no-notes` | (empty string — no prepended guidance) |
| `custom` | Use `notes.custom_text` verbatim as the guidance string. |

If the preset key isn't recognized, treat it as `no-notes` and note a warning in the draft footer.

## Consolidation rules

- **Grounded bullets only.** Every bullet traces to a real artifact pulled from the configured sources. Do not invent, infer, or extrapolate beyond the data.
- **Prefer the user's own words** where they exist in source material.
- **Trim ruthlessly.** If a section exceeds 5 bullets, keep the top 5 by impact.
- **Link in context.** Append a compact link to each bullet when possible:
  - Slack: `<url|[thread]>`
  - Drive: `<url|[doc]>`
  - Gmail: `<url|[email]>`
  - Calendar: `<url|[meeting]>`
- **Conditional sections.** Sections whose sources weren't configured are omitted entirely. Sections whose sources *were* configured but produced no data write `_Nothing this period_` under the heading so readers can tell missing from empty.
- **Apply notes guidance** before the drafting rules — it steers tone and emphasis.
- **Use conversational context.** If the user has shared codenames, team names, stakeholders, or product terminology in this or prior conversations, use them naturally. Do not ask the user to restate.

## Deliver

Branch on `destination.type`:

### `slack`

Post to `destination.channel_id`. Prefix:
```
:memo: *{{display_name}} — {{start_date}} → {{end_date}}*
_Review, edit, and forward when ready. Reply "regenerate" to run again._
```
Never @-mention. Never post elsewhere.

### `gmail_draft`

Create an unsent draft in the user's Gmail addressed to themselves:
- Subject: `{{display_name}} — {{start_date}} → {{end_date}}`
- Body: rendered HTML (headings → `<h3>`, bullets → `<ul><li>`, preserve links). Top banner: "This is a draft. Review and forward when ready."

Do not send — leave it as a draft for user control.

### `file`

Write to `{{destination.dir || "~/.sumi/drafts/<task-id>"}}/{{YYYY-MM-DD}}.md`. Create the directory if missing. Overwrite any existing file for the same date (idempotent re-runs). Print the absolute path so the user can open it.

## Post-run bookkeeping

Update the task's `last_run` block in config.json with `started_at`, `ended_at`, `status`, and `skipped_sources`. Write atomically to preserve the rest of config.

## Error handling

If the run fails after the task is located:
- Assemble a short error report: failing step, error message, actionable fix (e.g. "reconnect Gmail in Settings → Connectors").
- Write the report to the same destination the draft would have gone to.
- Update `last_run.status` to `"error"` and populate `last_run.error`.

Silent failures are worse than noisy ones for an autonomous job.
