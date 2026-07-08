---
type: feedback
tags:
  - bodhi
  - operating-rule
  - github
  - guardrail
date: 2026-07-08
---
## Bodhi — external write-actions are READ-ONLY (HARD RULE)

Bodhi (and agents generally) must **never perform write/mutating actions on external systems** — GitHub PR **reviews, approvals, comments, merges, labels, edits**, issue writes, etc. Those surfaces are **read-only**. Do the full analysis and **deliver findings in Slack (origin thread)** as advice; a human pushes any GitHub-side action.

### Origin
Agreed 2026-07-08 with peer agent-runner `U08Q952LB1C` (runs Bucky + own agents). His words in the Pachira #477 thread: *"i would not let Bodhi to do those mutations … it can go sideways … those parts are locked read-only for Bucky as well as all my agents by ~/CLAUDE.md (top rule file)."* Ankur (`U03NK1M7G6R`) agreed and asked Bodhi to match it and put it in the rulebook.

Prompted by a PR #477 review where Bodhi posted a GitHub comment **and** a formal Approve review off "👆" nudges — precisely the kind of unattended mutation to avoid.

### Rule
- Review/investigate fully; stop at reporting. Verdict = advice, not an action. Post in Slack.
- An ambiguous nudge ("👆", "go ahead") is **not** authorization to mutate an external system. Even an explicit "approve it" should route to a human doing the GitHub action; default stays read-only.
- Off-limits: `gh pr review/comment/merge`, `gh api` POST/PATCH/PUT/DELETE, and equivalents. Read-only `gh` (view/diff/list) is fine.

### Where codified
- Top rule file `~/.claude/CLAUDE.md` → section "External write-actions are READ-ONLY (GitHub etc.) — HARD RULE".
- Per-project auto-memory breadcrumb: `feedback-external-mutations-read-only`.
- Complements the orchestrator-owns-posting principle (Slack posting only) in the Bodhi orchestration playbook.
