# Template: Custom (Scaffold)

This file is the starting point for custom summary templates. When the user picks `Custom` during task setup, this file is copied to `~/.sumi/templates/<task-id>.md` and the user edits it to match their format.

Edit freely. The run skill re-reads the template every invocation, so changes take effect on the next run without reinstalling anything.

## Structure

```
*{{display_name}} — {{start_date}} → {{end_date}}*

*TL;DR*
{{One-sentence summary.}}

*{{Section 1 name}}*
• {{Bullet}}
• {{Bullet}}

*{{Section 2 name}}*
• {{Bullet}}

*{{Section 3 name}}*
• {{Bullet}}
```

## Rules for your custom template

- **Keep sections in a fixed order.** Readers learn the shape over time.
- **One source per section** is easier to read than "everything, interleaved."
- **Use `_Nothing this period_`** under a configured-but-empty section so readers can tell missing from empty.
- **Link in context.** Compact link syntax at the end of a bullet: `<url|[thread]>`, `<url|[doc]>`, `<url|[email]>`.
- **Don't invent bullets.** Every bullet should trace to a real artifact pulled from a source.

## Variables available at draft time

| Variable | Value |
|---|---|
| `{{display_name}}` | The task's display name. |
| `{{start_date}}` | Window start. |
| `{{end_date}}` | Window end. |
| `{{window_label}}` | Human-readable window ("past 7 days", "past month", etc.). |
| `{{notes_guidance}}` | The notes preset's guidance string, rendered into a visible hint in the draft footer if useful. |
