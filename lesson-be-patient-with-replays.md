---
name: lesson-be-patient-with-replays
description: "Don't declare a slow/long-running op (projection replay, catch-up, migration) failed from an early snapshot"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b358309a-4307-42c4-a471-adac67c7c0f4
---

During the Pistacia wallet projection rebuild (2026-06-26) I checked row counts ~7 min after restart, saw 0 rows, and wrongly concluded the replay was "stuck at the tail" — even claiming the runbook had a bug. It was actually replaying correctly from the start, just **slowly** (checkpoints were climbing 0→15.5bn; rows were trickling in). My wrong call caused an unnecessary pause + reset detour.

**Why:** a one-shot snapshot can't distinguish "not working" from "working slowly." Event-store replays / catch-up subscriptions / big migrations are often slow and sparse.

**How to apply:**
- Before declaring a long-running op failed, watch the **trend over time** (is the checkpoint/position/offset *climbing*? are rows *trickling in*?), not a single instant.
- Don't claim someone's runbook/tool is buggy until the diagnosis is verified against the actual mechanism — verify, then assert.
- Know where the signal lives: the Pistacia projections host has `serilog.consoleenabled=false`, so `kubectl logs` shows only the DbUp bootstrap — judge replay by **row counts + checkpoint position + Honeycomb traces**, not stdout. See [[pistacia-projection-rebuild-runbook]].
- When unsure, say "still progressing / inconclusive, checking again in N min" rather than calling it done or failed.

**REINFORCED 2026-06-29 (I repeated this exact mistake — twice in one op).** Running the v7 category-stream wallet rebuild ([[esdb-projection-category-stream-pattern]]), I twice cried "failing" from early snapshots: first "ledger is failing" (it was 9 rows, cold-start slow), then "balance is failing / 0 rows" — and spun up a whole Honeycomb/ESDB investigation. Minutes later the replay accelerated and finished perfectly: ledger 2621, balance 441, autotopup 58, **byte-for-byte identical to the pre-rebuild backup** (0 mismatches). The replay order is reliably **autotopup → ledger → balance**, non-linear (slow cold start, late burst). So on THIS specific rebuild: expect autotopup to finish first and ledger/balance to lag then catch up — do NOT diagnose failure until the checkpoint stops climbing for a sustained period. Watch the trend; verify-by-diff against the backup at the end.
