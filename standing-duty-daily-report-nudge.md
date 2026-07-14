---
name: standing-duty-daily-report-nudge
description: "Standing duty: Bodhi pokes Ankur on Slack DM D0BBTUXCATZ every 4h
  to send his daily report (session-only cron, re-bind after restart)"
metadata:
  node_type: memory
  type: project
---

Ankur asked (2026-07-13) that Bodhi **poke him on Slack DM channel `D0BBTUXCATZ` every 4 hours** with a short nudge to send his daily report. He chose "just a nudge" — no activity summary/dashboard, only the reminder.

**Mechanism — session-only local cron** (NOT a cloud schedule): posting the nudge requires the LOCAL buck MCP / Slack bridge running in the Bodhi session; a cloud routine has no access to it and cannot post. So this runs as an in-session `CronCreate` job.

- Cron: `7 */4 * * *` (every 4h at :07, off-minute to avoid the fleet :00 pileup), recurring, session-only.
- Action per fire: `slack_bridge_post` to `D0BBTUXCATZ` a short varied friendly nudge; if the buck session is inactive, `start_session` first.
- Original job id (2026-07-13 session): `73961b82`.

**Re-bind after restart/compaction (MUST):** a session-only cron dies with the session and is invisible to the auto-summary, so it silently lapses. Once the Slack bridge is re-armed (see [[buck-mcp-launch-config]] / [[bodhi-orchestration-playbook]]), re-create the cron with the same `7 */4 * * *` schedule and nudge prompt.

**Stop:** `CronDelete <jobid>`, or when Ankur says stop. Auto-expires after 7 days regardless — re-create if still wanted.
