---
name: context-summary-methodology
description: How to write a context-summary/checkpoint so a fresh instance survives context compaction without losing the thread — adopt for multi-step/investigation/bridge work
metadata: 
  node_type: memory
  type: methodology
  originSessionId: 53bb78f7-2e94-4d5b-b843-4cb54da5e724
---

Context-summary = checkpoint (ONE file). Write it before compaction so a fresh instance resumes in minutes instead of re-deriving. From Bucky's handoff 2026-07-01 (Faruk: "one of the most important to adopt"). Apply proactively; complements [[bodhi-orchestration-playbook]] and [[buck-mcp-launch-config]].

**Core rules:**
- **Map, not transcript. Enough beats minimal** — under-saving is a false economy (next context drifts + re-derives, costing more than a fuller summary). Target ~500–1000 lines for substantial work. Inline every owner directive, `file:line`, decision, and the *why*; breadcrumb deep detail (point to runbook/spec).
- **NEVER overwrite — always a NEW version** (`…-{N}.md`→`{N+1}`, or SemVer; MINOR per checkpoint, PATCH for a fix). It's your last safe recovery point; overwriting destroys the fallback state. List existing versions and bump to next free path before writing; latest = source of truth; keep all in one dedicated folder.
- **Keep it current** — refresh to a fresh version *just before* compacting; never write-once-and-forget (post-write work would be lost).
- **Standing duties — ALWAYS breadcrumb (MUST READ):** a role that runs across turns/sessions (inbox/bridge orchestration, a watcher, a cron/idle-guard, a held lock, on-call) is **invisible to the auto-summary** and silently lapses after compaction → fresh agent resumes deaf. For each: breadcrumb (point to runbook, don't inline), mark MUST READ at top of read-order + in the pre-compact reminder, and STATE THE RE-BIND ACTION (e.g. reconnect + relaunch the watcher). Also breadcrumb the agent's own always-loaded memory store as MUST READ.
- **Threshold protocol:** <60% keep working; 60–79% don't *start* big work without checkpointing; **≥80% controlled wind-down** (finish small unit → fresh versioned checkpoint → hand off); **≥90% hard stop**. Checkpoint more often near the limit; **cross-review the checkpoint with a peer before compacting**.
- **When to write:** before any compaction with non-trivial state; before switching machines/sessions; as a hand-off to another agent; after each important turn (decision/delivery/role change). NOT for routine one-shot sessions.

**9-section template:** (1) Read order for fresh-context recovery (#1 = the summary itself; include MUST-READ own-memory + standing-duty runbooks); (2) Curated code reading list (`file:line`, grouped — often the most valuable section); (3) Session outcomes (numbered); (4) External-system changes (tickets/memory/deploys/flag flips); (5) Code/test posture (branch, last test run, flakes); (6) Items remaining / sequencing; (7) Open items / decisions pending; (8) Methodology lessons captured; (9) Pre-compact reminder (where to start + every standing duty + its first re-bind action).

**Anti-patterns:** dumping the full session; skipping the curated reading list (biggest miss); stale `file:line` (verify before writing); restating the plan (link it); burying decisions in prose (numbered bullets); writing once then compacting stale.
