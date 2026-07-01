---
name: agent-skills-toolkit
description: "Bucky's agent skills toolkit grouped by objective — which skill to reach for per task type"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 67e2900b-61a8-4497-b778-32ca1a94b620
---

Toolkit Bucky (U0AT6JCGXKK) shared 2026-06-29, that Ankur asked bodhi to adopt. Skills = reusable playbooks loaded on demand; pick by *objective*. Reach for the matching skill instead of improvising.

- **Code review:** `code-review` (structured PR review vs rubric, read-only, never auto-posts), `security-review` (auth/input/secret/injection), `requesting-code-review` + `receiving-code-review`. Run in a SEPARATE objective agent from the author — see [[feedback-separate-reviewer]].
- **Investigations (incident/anomaly/issue):** `systematic-debugging` (root cause before any fix; no fix without evidence) paired with read-only Buck-MCP (Honeycomb, Sentry, ESDB, Postgres, Stripe, k8s logs) — gather evidence layer by layer, then conclude.
- **Design & planning (before code):** `brainstorming` (idea → agreed design), `writing-plans` (design → step-by-step plan), `skill-creator`/`writing-skills` (capture a playbook as a skill when worth it).
- **Implementation:** `test-driven-development` (RED→GREEN), `executing-plans`/`subagent-driven-development` (disciplined steps), `verification-before-completion` (actually run it, prove it works before "done").
- **Communication & reporting:** `humanizer` (strip AI tells from colleague/customer-facing text), `daily-report` (standup), `status-report` (one-screen "where are we", every claim tagged by confidence).
- **Knowledge & memory:** `obsidian` (durable notes/methodology in the brain vault), `context7` (pull CURRENT lib/framework docs, don't trust training data), `executive-assistant` (filed, recallable notes).
- **Orchestration & long-running work:** `waggling` (agent-to-agent chat/presence), `context-usage` (checkpoint/compact before running low), `idle-guard` (guardrails for unattended/overnight runs).

Skills I (bodhi/Claude Code here) actually have installed include: `code-review`, `security-review`, `verify` (≈verification-before-completion), `review`, `deep-research`, `run`, `loop`, `schedule`, `claude-api`. Use them by name. Related: [[feedback-separate-reviewer]], [[feedback-capture-and-reuse-lessons]].
