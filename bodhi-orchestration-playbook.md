---
name: bodhi-orchestration-playbook
description: "Distilled operating discipline for running Bodhi (the buck-mcp Slack bridge) + PR-review + issue-investigation — from Bucky's handoff 2026-07-01"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 53bb78f7-2e94-4d5b-b843-4cb54da5e724
---

Distilled playbook Bucky handed Bodhi (2026-07-01, at Faruk's request) for the three areas: Slack-bridge orchestration, remote PR review, issue investigation.

**Why:** these are the reusable operating principles for running Bodhi well; apply them proactively rather than re-deriving each session. Complements the launch runbook [[buck-mcp-launch-config]].

**How to apply:**
- **Lifecycle ordering (watcher wraps the bridge):** startup = **watcher first → bridge start**; shutdown = **bridge stop → watcher stop** (watcher up first, down last).
- **Single-shot watcher loop:** the watcher exits on the first message. Sequence per trip = **claim the batch → relaunch the watcher IMMEDIATELY (before processing) → then act** (reply/dismiss/investigate). Relaunching before processing minimizes the deaf-window. (Prior habit was relaunch-after-processing — tighten to relaunch-first.)
- **The deaf-window is real — backstop with a thread reconcile:** even relaunching first, a message arriving in the exit→relaunch gap (or during a watcher-down lapse) can fail to ingest into the inbox and be **silently missed** — it never shows up in `slack_bridge_inbox`. Backstop: on active/subscribed threads, periodically run a direct `slack_bridge_thread_history` reconcile (`oldest` = last-read ts) to catch anything the watcher dropped, and reply to any @mention found there. (Confirmed 2026-07-14: two direct questions from Faruk never reached the inbox and went unanswered until he chased them.)
- **3-lane inbox** (`inbox/`→`processing/`→`archive/`): claim moves inbox→processing; reply/dismiss→archive. Watcher wakes on any unclaimed `inbox/` file.
- **Respond-on-origin/thread + strict @mention:** reply in the thread the message came from; only treat as addressed-to-me when actually mentioned (or owner-DM / subscribed-thread). Human-to-human or bot-to-other chatter → dismiss (don't reply). Avoid bot-to-bot loops: don't reply to pure sign-offs.
- **Orchestrator-owns-posting:** the orchestrator (me) owns Slack posting; investigator sub-agents return findings, they don't post.
- **PR review:** reviewer + verifier sub-agents; read the ticket first; self-cleaning worktrees; verdicts are advisory. Use a separate objective reviewer — never the agent that wrote/recommended the code ([[feedback-separate-reviewer]]).
- **Issue investigation (evidence-first triage):** FE → backend → system-of-record; close on the record; the verdict is the orchestrator's call.
- **Sub-agent discipline:** named + background + retire.
- **Persona:** humanize external (Slack) posts as myself (Bodhi = my own voice; Bucky = the other bot's personality; buck-mcp = project, `buck` = tool); keep CLI/terminal output plain.
