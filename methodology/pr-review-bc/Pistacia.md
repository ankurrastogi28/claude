---
name: pr-review-bc-pistacia
description: Pistacia (payments BC) per-BC checklist for Remote PR Review — the
  BC-specific layer on top of the generic skills + 3 cross-cutting dims. From
  Bucky 2026-07-14 (wallet/projection-relevant subset).
metadata:
  node_type: memory
  type: reference
---
Per-BC checklist for **Pistacia (payments BC)** — the layer that sits ON TOP of the generic `code-review` + `security-review` skills and the 3 cross-cutting dims in [[remote-pr-review-methodology]]. From Bucky 2026-07-14. This is the *wallet/projection-relevant subset* (the hotspots that matter for a PR like #217); the full Pistacia checklist in Bucky's brain has more sections — pull the rest from Bucky when a PR touches other areas.

## Key Pistacia hotspots (money / projection / wallet PRs)
- **§4-H — Projection faithfulness & replay-idempotency.** Any money/balance event needs: its **projection** + a **numbered SQL migration** + a **`Projections.Test.Integration` test**. `Project(...)` methods must be **replay-safe** (checkpoints rewind on a rebuild → the handler must produce the same result when the same event is re-applied). This is exactly the #217 surface (wallet ledger/balance projections).
- **§2 — Event versioning discipline.** Schema evolution = **new versioned events + appliers**, NEVER in-place edits to a persisted event. Appliers/mappers must **tolerate null/legacy fields** — old serialized events won't have fields added later. (This is the exact #217 bug: legacy pre-2025-05 `WalletCredited` events had no nested `Customer`, so `e.Customer.Id` NRE'd and silently dropped rows on replay.)
- **§7 — Downstream scan.** Pistacia events ripple to **Projections.Host + Olive** (shared ESDB cluster), and **wallet-ledger changes hit the EOM (end-of-month) balance reports**. Trace the ripple beyond the diff.

## Other BCs (pull from Bucky when needed)
Pachira (billing), Aspen (account/identity/auth), Moss (edge gateway) each have their own equivalent checklist in Bucky's brain — request per PR to complete the set.

Related: [[remote-pr-review-methodology]] · [[esdb-projection-category-stream-pattern]] · [[pistacia-projection-rebuild-runbook]]
