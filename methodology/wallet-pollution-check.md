---
name: wallet-pollution-check
description: How to detect wallet-projection pollution (rows keyed by a legacy
  account id instead of a 3PL id) — the NOT-IN-known-3PL-ids query, current prod
  3PL ids, and the ~2-week re-check Faruk asked for.
metadata:
  node_type: memory
  type: reference
---
"Pollution" = wallet projection rows keyed by a legacy **account id** (or NULL) instead of the **3PL id** — see [[lazy-wallet-migration-pollution-test]] for what pollution is + the fix (category-stream narrowing to `$ce-Pistacia_Wallet` + BUC-336 3PL-id-on-emitted-events; `WalletAppliers` heal legacy account-id TenantIds on rehydration).

## The check (prod, read-only) — Faruk's method 2026-07-14
Healed wallet projections must carry ONLY 3PL tenant ids. Filter OUT the known 3PL ids with a `NOT IN` clause; anything left is pollution:
```sql
SELECT DISTINCT tenant_id FROM (
  SELECT tenant_id FROM wallet_ledger
  UNION SELECT tenant_id FROM wallet_balance
  UNION SELECT tenant_id FROM wallet_auto_topup_configurations
) t
WHERE tenant_id IS NULL
   OR tenant_id NOT IN (<known 3PL ids>);
```
Any row returned = pollution (account-id leak / un-healed tenant). 0 rows = clean. conf_node `Pistacia.Projections.Host.Postgres` (prod).

## Known-good PROD 3PL ids (2026-07-14, post-rebuild — the ONLY tenant ids in all 3 wallet tables):
- `83d1191e-6451-42d7-af5f-209b79a4b0cc` (PH — bulk: 5078 ledger / 386 balance / 49 autotopup)
- `335cd10c-7b7e-479a-8fa8-d227569d6601` (PC — 412 / 31 / 6)
- `2c897ecc-fe94-4cf6-8a40-8d682c07b532` (PH_IE — 1 / 1 / 0)
Validate against the `Pistacia:Tenants` config before calling a stray id pollution — a NEW *legit* 3PL tenant would also surface here (not pollution). Old account id seen during the incident (definite pollution if it reappears): `e1702761-30bb-4d00-88bf-b79f8da6cf3f` (wallet80/Janio's pre-heal tenant).

## Monitoring
Bodhi session cron `93bbd2ff` runs this pollution query + a Honeycomb anomaly check every 10 min, quiet-unless-anomaly (session-scoped — dies with the session; recreate next session if still needed).

⚠️ **RE-CHECK in ~2 weeks (≈ 2026-07-28)** — Faruk (2026-07-14) asked to re-run this pollution check a couple of weeks post-rebuild, to confirm no pollution has crept back in via live traffic since the rebuild.

Related: [[lazy-wallet-migration-pollution-test]] · [[pistacia-projection-rebuild-runbook]] · [[esdb-projection-category-stream-pattern]]
