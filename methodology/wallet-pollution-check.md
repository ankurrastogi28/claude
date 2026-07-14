---
name: wallet-pollution-check
description: "How to detect wallet-projection pollution (rows keyed by a legacy account id instead of a 3PL id) — the NOT-IN-known-3PL-ids query, the canonical prod 3PL ids, and the ~2-week re-check Faruk asked for."
metadata:
  node_type: memory
  type: reference
---

"Pollution" = wallet projection rows keyed by a legacy **account id** (or NULL) instead of a **3PL id** — see [[lazy-wallet-migration-pollution-test]] for what pollution is + the fix (category-stream narrowing to `$ce-Pistacia_Wallet` + BUC-336 3PL-id-on-emitted-events; `WalletAppliers` heal legacy account-id TenantIds on rehydration).

## The check (prod, read-only) — Faruk's method 2026-07-14
Healed wallet projections must carry ONLY known tenant ids. Filter OUT the known ids with a `NOT IN` clause; anything left is pollution:
```sql
SELECT DISTINCT tenant_id FROM (
  SELECT tenant_id FROM wallet_ledger
  UNION SELECT tenant_id FROM wallet_balance
  UNION SELECT tenant_id FROM wallet_auto_topup_configurations
) t
WHERE tenant_id IS NULL
   OR tenant_id NOT IN (<known ids>)
LIMIT 20;   -- ALWAYS bounded ([[feedback-always-limit-sql]])
```
Any row returned = pollution (account-id leak / un-healed tenant). 0 rows = clean. conf_node `Pistacia.Projections.Host.Postgres` (prod).

## Canonical PROD 3PL ids — source of truth: `Pistacia.Sdk/TplIds.cs` (4 brands)
- `83d1191e-6451-42d7-af5f-209b79a4b0cc` — **PH** prod (sandbox is a *different* id `61d0547f-ec2b-4bd6-9cb5-5b74abaf3020`)
- `335cd10c-7b7e-479a-8fa8-d227569d6601` — **PC**
- `414326c7-4b0e-4892-8edd-666519926d98` — **DP**
- `c473ae46-ff8b-46b2-94fe-df5344c9426d` — **PH_IE**
Legacy ids (also `IsKnownThreePl`=true, but their presence in wallets = un-healed): PH_LEGACY `e1702761-30bb-4d00-88bf-b79f8da6cf3f` (wallet80/Janio pre-heal — definite pollution if it reappears), PC_LEGACY `3fa85f64-5717-4562-b3fc-2c963f66afa6`. `TplIds.IsKnownThreePl(id)` is the authoritative classifier. The resolver's mapping binds config section `Pistacia:Tenants` (= `TenantMappingOptions`, `appsettings…Pistacia:Tenants`). Projection carries `e.TenantId` verbatim (no re-resolve at projection time) — the heal is upstream in `WalletAppliers`.

## MONITOR ALLOWLIST (use these 5) — corrected + resolved 2026-07-14
The 4 canonical 3PL ids **plus** `2c897ecc-fe94-4cf6-8a40-8d682c07b532` = **a known prod test account** (NOT a 3PL id): Aspen user `f.pehlivanli+prod-001@parcelhero.com`, `UserId==TenantId==WalletId==2c897ecc` (self-tenant), £1 balance-neutral `is_migrated=true` wallet, created 2025-12-03; confirmed by Arthur (read-only AspenIdentity). Present pre-rebuild (Bucky `_bak` read) → pre-existing, not rebuild-introduced. So it's benign test data — allowlist it (chosen over deleting an event-sourced source-of-truth for a £1 test row).

**History (why the allowlist was wrong all day 2026-07-14):** the original monitor ran only `83d1191e, 335cd10c, 2c897ecc` and I *mislabelled* `2c897ecc` as "PH_IE" — it isn't (real PH_IE is `c473ae46`; `2c897ecc` is absent from `TplIds.cs`). That masked the `2c897ecc` row (false-negative) AND would have false-positived real DP/PH_IE wallets. Fixed to the 5-id allowlist above. Lesson: verify allowlists against `TplIds.cs`/`Pistacia:Tenants`, not by inference.

## Monitoring
Bodhi session cron ran this every 10 min (quiet-unless-anomaly). **Handed off to Bucky 2026-07-14 at session close** (Bodhi session cron is session-scoped and dies with the session). Post monitoring updates to incident thread `C08EQJF2RM0` / msg `1784004644.116039`; hourly all-green cadence.

⚠️ **RE-CHECK ≈ 2026-07-28** (Faruk) — re-run this pollution check ~2 weeks post-rebuild to confirm no pollution crept back via live traffic.

Related: [[lazy-wallet-migration-pollution-test]] · [[pistacia-projection-rebuild-runbook]] · [[esdb-projection-category-stream-pattern]] · [[bodhi-orchestration-playbook]]
