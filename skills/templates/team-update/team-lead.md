# Template: Team Update — Team Lead

A weekly update from a team lead's perspective. Focuses on team-level outcomes, cross-team dependencies, and the asks the lead needs from their stakeholders. Assumes the reader is a peer lead, a manager, or a stakeholder outside the team.

## Structure

```
*{{display_name}} — {{start_date}} → {{end_date}}*

*TL;DR*
{{One sentence. The single most important thing a reader needs to know.}}

*🏆 Wins*
• {{Team-level outcome that shipped or landed. Name who owned it if not the team lead.}}

*🛠️ In progress*
• {{Initiative underway, milestone, rough %. Owner if not the team lead.}}

*🚧 Blockers & risks*
• {{What's stuck, why, who can unstick it. If nothing, write "_Nothing this period_".}}

*📅 Meetings & decisions*  ← only if Calendar is a source
• {{Notable decision or outcome from this week's meetings.}}

*📣 Customer / external signal*  ← only if Gmail is a source
• {{Customer feedback, external mention, or inbound worth escalating.}}

*💬 Channel activity*  ← only if Slack is a source
• {{Key thread or signal from tracked channels.}}

*📈 Metrics that moved*
• {{Deltas. Don't include metrics that didn't move.}}

*🙋 Asks*
• {{Decisions, intros, reviews the lead needs from the reader.}}

*🔜 Next period*
• {{Top 3 commitments for the next period.}}
```

## Rules

- **Outcomes over activity.** "Shipped checkout v2 to 100%" beats "worked on checkout."
- **Name owners when not the lead.** Stakeholders should know who to thank or follow up with.
- **Trim to 5 bullets per section.** Keep the highest-impact.
- **Expand internal codenames** on first use — the reader may be outside the team.
- **Conditional sections drop if no source configured; empty-but-configured sections say `_Nothing this period_`.**
