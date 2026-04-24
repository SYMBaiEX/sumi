# Template: Customer Digest — Support

A periodic digest for a support-facing audience. Surfaces customer issues, patterns across tickets, and items that need product/engineering escalation. Pulls primarily from Gmail (support inbox) and Slack (support channels).

## Structure

```
*{{display_name}} — {{start_date}} → {{end_date}}*

*TL;DR*
{{One sentence on the shape of the queue this period.}}

*🔥 Top issues by volume*
• {{Issue pattern — N tickets. Example ticket linked.}}

*📣 Direct customer quotes worth surfacing*
• "{{quote}}" — Account, date.

*🆘 Escalations to product / engineering*
• {{Escalation. Owner. Status.}}

*💡 Feature requests (clustered)*
• {{Request theme — N asks. Top requesting accounts.}}

*✅ Resolved this period*
• {{Notable fix or resolution. Customer impact.}}

*🚩 Risks to watch*
• {{Pattern that hasn't hit severity yet but is trending.}}

*🙋 Asks*
• {{Thing support needs from another team.}}
```

## Rules

- **Cluster by pattern, not by ticket.** Execs and product leads care about shapes, not individual tickets.
- **Quote customers verbatim** when possible. Attribute to the account, not the individual (unless the user said otherwise in conversation).
- **Name the escalation owner.** Support digests are most useful when they clearly hand off.
- **Surface churn signal explicitly** if present in source material — that's the highest-leverage thing support sees.
