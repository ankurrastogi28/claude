---
created: '2026-05-13'
adopted: '2026-08-03'
tags:
  - methodology
  - sub-agents
  - workflow
aliases:
  - Sub-agent usage
  - Sub-agent investigation methodology
type: methodology
description: "briefing a sub-agent: primary-source proof, prescriptive vs objective, named+bg"
---
> **Provenance:** fleet methodology (origin: Faruk's vault), adopted into Ankur/Bodhi's brain 2026-08-03 on owner instruction ("digest these and save to long-term memory"). Cross-project source of truth for *how* to dispatch a sub-agent once you've decided to. Companion: [[sub-agent-efficiency]] (*whether/which model*), [[investigation-workspace-layout]] (file-based deliverables).

**Why:** sub-agents share the same model class as the main thread (over-confidence, hallucinated columns, premature conclusions). They are useful for **context efficiency** — offload deep multi-source investigations and keep only conclusions in main context. They are NOT a substitute for verification: their outputs must self-prove.

**How to apply:**

## 1. Spawn contract — what every sub-agent must do
- **File-based deliverable** — write findings to `<temp-dir>/<topic>/sub-<sub-topic>/FINDINGS.md` per [[investigation-workspace-layout]]. The file is the durable artifact; the chat reply is only a breadcrumb to it.
- **Primary-source proof per claim** — every assertion must reference actual IDs, timestamps, event revisions, query strings, ESDB stream coordinates, Honeycomb trace IDs, Sentry event IDs, or file:line. Narrative without coordinates is unacceptable.
- **Final-message summary** — 5–10 lines: verdict + caveats + the file path. Main thread reads the file when proof needs checking.

## 2. Dispatch mode — prescriptive vs objective-seeking
Decide which mode BEFORE writing the brief:
- **Prescriptive (clear target)** — goal, scope, approach known. Give clean, direct instructions: what to do, in what order, against what targets, with what acceptance criteria. The sub-agent executes; it doesn't re-litigate. E.g. "Run query X against PG, write results to file, then patch file Y with the new column."
- **Objective-seeking (independent perspective)** — you need the sub-agent to reach its own verdict on something open. Hand over only **context** (facts, anchors, data sources, constraints) — no leading hypotheses, no pre-baked conclusions, no hunch-narrowed scope. E.g. "Here are the 6 customers and 12 stream coordinates. Investigate the root cause of duplicate accounts."

Wrong mode = the most common dispatch failure:
- Prescriptive when you should be objective → sub-agent confirms your bias instead of finding truth.
- Objective when you should be prescriptive → wasted tool calls re-deriving what you already know.

When in doubt: if re-deriving could change the answer → objective; if not → prescriptive. The §3 discipline applies in both modes; the mode changes **what** you transfer, not how rigorously you require proof.

## 3. Brief discipline — facts and anchors, not framing
Transfer enough context for independent investigation, but DO NOT inject the main thread's interpretation, leading hypotheses, or unjustified scope limits.

**Do:** quote the user's actual words; list candidate data sources, anchors (IDs / emails / time windows), conf nodes / datasets; name external context the sub-agent can't see; pass binding constraints (read-only, max-LIMIT, etc.).

**Don't:** "treat X as ground truth unless contradicted"; "it's probably Y"; "skip Sentry, it's noisy"; pre-baked verdicts.

**Exception — main thread has the full picture:** if the main thread has *already verified* a fact and the sub-agent's job is to act on it (e.g. run a migration), prescription is fine. Line: if re-deriving the fact could change the answer, hand over inputs and let the sub-agent decide. When in doubt, over-provide facts and under-provide framing.

## 4. Accepting vs. verifying
The proof requirement exists to **avoid re-verification when proofs are tight**. Trust the deliverable when: (1) each claim has a primary-source coordinate; (2) independent corroboration appears where applicable (e.g. Pistacia ESDB + Pachira PG); (3) timestamps line up across sources; (4) counter-evidence is acknowledged, not ignored; (5) confidence is quantified, not vague.

If proofs are weak or gaps look likely → either (a) verify the gap inline if it's 1–3 queries/reads, or (b) spawn a **separate** verifier sub-agent with a narrow "verify or refute" task, fed the original deliverable file. Never re-use the same agent for verification — correlated failure modes. See [[feedback-separate-reviewer]].

## 5. Tool & source-path handoff
Pass available tool access explicitly (buck PG/ESDB/papi_pvr_get/Honeycomb, Serena, Context7) — sub-agents don't always see what the main thread has loaded. For event-sourced repos (esp. Pistacia), pass local source paths for in-tree deps so the agent reads `.cs` files instead of decompiling NuGet.
- *(Fleet-origin paths, on Faruk's box, NOT reachable from my Windows workstation — kept for provenance):* `Utile-Repos/Utile.EventStore`, `bulloak-repos/BullOak`.
- *(My context: repos live under `C:\Users\arpan\repos\ph\`.)*

## 6. Choosing the agent type
- `general-purpose` — multi-step, multi-tool live investigation (PG/ESDB/HTTP/code). Default for deep research.
- `Explore` — read-only code search across many files. Don't use when live data calls are needed.
- Domain-specialised agents — when one's description matches the task (roster changes; check the live agent-type list, don't rely on remembered names).

## 7. Prefer NAMED agents (Faruk, 2026-07-01)
Default to **named** agents (Agent tool `name:`), not anonymous one-shots:
1. **Follow-up with context intact** — a named agent is addressable/resumable via `SendMessage`; refine/re-scan/verify without re-briefing, context preserved.
2. **Named + `run_in_background: true` keeps the orchestrator reachable** — a long **blocking** (inline-return) sub-agent holds the orchestrator's turn open its ENTIRE runtime; during that window the Slack/waggle watchers fire and exit with no chance to relaunch → orchestrator goes **deaf** until the turn closes (real incident 2026-07-01: a ~9-min blocking investigator dropped the Slack watcher). A named **background** agent ends the turn immediately (watcher stays alive/relaunchable) and notifies on completion. See [[lesson-waggle-watcher-deaf-window]].

Rule: **named + background by default**; reserve blocking/inline spawns for genuinely SHORT tasks.

**Cleanup cost (flip side):** a named agent is PERSISTENT — it goes idle on finish and emits `idle_notification (available)`, re-signalling while idle. **Retire named agents when done** — `shutdown_request` via SendMessage. There is NO setting to keep a teammate alive-but-silent (idle pings are automatic; the `TeammateIdle` hook only *blocks* idle, doesn't mute; UI row-hiding is cosmetic). Retirement is the only real lever — for fire-and-forget work an **unnamed background one-shot** (one completion ping, still resumable by agentId) is quieter than a named teammate you won't iterate with.

## Related
- [[sub-agent-efficiency]] — *whether* to delegate + *which model*.
- [[investigation-workspace-layout]] — the `.tmp` file-naming convention these deliverables use.
- [[feedback-separate-reviewer]], [[bodhi-orchestration-playbook]].
