---
name: wallet-pollution-check
description: "How to detect wallet-projection pollution (rows keyed by a legacy account id instead of a 3PL id) — the NOT-IN-known-3PL-ids query, the canonical prod 3PL ids, and the ~2-week re-check Faruk asked for."
metadata:
  node_type: memory
  type: reference
---

"Pollution" = wallet projection rows keyed by a legacy **account id** (or NULL) instead of a **3PL id** — see [[lazy-wallet-migration-pollution-test]] for what pollution is + the fix (category-stream narrowing to `$ce-Pistacia_Wallet` + BUC-336 3PL-id-on-emitted-events; `WalletAppliers` heal legacy account-id TenantIds on rehydration).

## The check (prod, read-only) — Faruk's method 2026-07-14
Healed wallet projections must carry ONLY 3PL tenant ids. Filter OUT the known 3PL ids with a `NOT IN` clause; anything left is pollution:
```sql
SELECT DISTINCT tenant_id FROM (
  SELECT tenant_id FROM wallet_ledger
  UNION SELECT tenant_id FROM wallet_balance
  UNION SELECT tenant_id FROM wallet_auto_topup_configurations
) t
WHERE tenant_id IS NULL
   OR tenant_id NOT IN (<known 3PL ids>)
LIMIT 20;   -- ALWAYS bounded ([[feedback-always-limit-sql]])
```
Any row returned = pollution (account-id leak / un-healed tenant). 0 rows = clean. conf_node `Pistacia.Projections.Host.Postgres` (prod).

## Canonical PROD 3PL ids — source of truth: `Pistacia.Sdk/TplIds.cs` (4 brands)
- `83d1191e-6451-42d7-af5f-209b79a4b0cc` — **PH** prod (sandbox is a *different* id `61d0547f-ec2b-4bd6-9cb5-5b74abaf3020`)
- `335cd10c-7b7e-479a-8fa8-d227569d6601` — **PC**
- `414326c7-4b0e-4892-8edd-666519926d98` — **DP**
- `c473ae46-ff8b-46b2-94fe-df5344c9426d` — **PH_IE**
Legacy ids (also `IsKnownThreePl`=true, but their presence in wallets = un-healed): PH_LEGACY `e1702761-30bb-4d00-88bf-b79f8da6cf3f` (wallet80/Janio pre-heal — definite pollution if it reappears), PC_LEGACY `3fa85f64-5717-4562-b3fc-2c963f66afa6`. `TplIds.IsKnownThreePl(id)` is the authoritative classifier. The resolver's mapping binds config section `Pistacia:Tenants` (= `TenantMappingOptions`, `appsettings…Pistacia:Tenants`).

⚠️ **CORRECTION 2026-07-14:** the monitoring allowlist I ran all day used `83d1191e, 335cd10c, 2c897ecc` and I *mislabelled* `2c897ecc-fe94-4cf6-8a40-8d682c07b532` as "PH_IE". **`2c897ecc` is NOT a known 3PL id** — it's absent from `TplIds.cs` (`IsKnownThreePl`=false) and the real PH_IE is `c473ae46`. So the correct allowlist is the **4 canonical ids above**, and `2c897ecc` (seen as 1 wallet_ledger / 1 wallet_balance row at rebuild) is an **unrecognized/un-healed row to investigate**, not a clean tenant — my "0 rows" reads were masking it (false-negative). Fix pending Faruk's ok: switch the cron allowlist to the 4 ids + pull the `2c897ecc` row. (Faruk 2026-07-14: "we have 4 brands, PH has different sandbox/prod ids".)

## Monitoring
Bodhi session cron `93bbd2ff` runs this pollution query + a Honeycomb anomaly check every 10 min, quiet-unless-anomaly (session-scoped — dies with the session; recreate next session if still needed). Post monitoring updates to the incident thread `C08EQJF2RM0` / msg `1784004644.116039`; hourly all-green cadence.

⚠️ **RE-CHECK in ~2 weeks (≈ 2026-07-28)** — Faruk (2026-07-14) asked to re-run this pollution check a couple of weeks post-rebuild, to confirm no pollution has crept back in via live traffic.

Related: [[lazy-wallet-migration-pollution-test]] · [[pistacia-projection-rebuild-runbook]] · [[esdb-projection-category-stream-pattern]]
