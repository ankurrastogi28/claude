---
name: lazy-wallet-migration-pollution-test
description: "How to test lazy wallet import/migration on sandbox to verify the wallet-pollution fix — mechanism, verification surface, cohort A test accounts, and two buck-mcp access gaps"
metadata: 
  node_type: memory
  type: project
  originSessionId: 53bb78f7-2e94-4d5b-b843-4cb54da5e724
---

Task (raised by Ankur + architect U08Q952LB1C, 2026-07-01): test **lazy wallet import/migration** on sandbox to verify the **wallet-pollution fix**.

**Mechanism (Pistacia):**
- CSV `POST /lazy-import` → rows written to Postgres `lazy_wallet_delta` (cols: `three_pl_id`, `customer_email`, `csv_balance`, `delta_amount`, `processed`, `processed_at`). CSV columns: `three_pl_id, customer_email, legacy_balance` (+ optional `payment_token, minimum_balance, minimum_topup` for auto-topup). Delta is **relative**: `delta = new_legacy_balance − last_csv_balance_for_(3pl,email)`; re-import at same balance → delta 0 (ZeroDeltaProcessed).
- Lazy trigger: `GET /wallet/{walletId}/balance|ledger|status` → `LazyDeltaProcessor.ProcessLazyDeltasIfAny(walletId)` → resolves customer via Acacia `GET /v1/accounts/{walletId}` → returns `(ThreePlId, Email)` → drains unprocessed deltas for that `(3pl,email)` under a distributed lock → emits MigrateWallet (new wallet) / WalletCredited (delta>0) / WalletDebited (delta<0) / ZeroDeltaProcessed. **Gated by Unleash flag `EnableLazyWalletMigration`.**
- CSV `(3pl,email)` MUST match what Acacia returns for that account's walletId, or the GET won't find the delta.

**Pollution fix being verified:** (1) wallet projections narrowed from `$all` → `$ce-Pistacia_Wallet` category stream (`Pistacia.Projections.Host/ServiceCollectionExtensions.cs`, `.UsingCategoryStream`); (2) BUC-336 — every emitted wallet event carries the **3PL id, not the account id**; `WalletAppliers` heals legacy account-id TenantIds → 3PL on rehydration. "Pollution" = projection rows keyed by wrong tenant (account id) or foreign events leaking in. See [[esdb-projection-category-stream-pattern]], [[pistacia-projection-rebuild-runbook]].

**Verification surface** (conf_node `Pistacia.API.Host.Postgres`, sandbox): `wallet_balance` (tenant_id=3PL uuid, `is_migrated` bool, customer_id==wallet_id for these), `wallet_ledger`, `wallet_auto_topup_configurations`, `lazy_wallet_delta`, `checkpoints`. Recent rebuild left `*_bak_20260626` / `*_bak_20260629` backup tables. ESDB events on `$ce-Pistacia_Wallet`.

**Cohort A (already imported & processed) — real sandbox accounts:** 3PL `61d0547f-ec2b-4bd6-9cb5-5b74abaf3020`: `ankit+1..+9@parcelhero.com` (+1/+2/+3/+5 fully processed; +4/+6/+7/+8/+9 have `processed=false` deltas pending = free lazy-drain coverage). 3PL `335cd10c-7b7e-479a-8fa8-d227569d6601`: `ankit@parcelhero.com`, `ankit@parcelvision.com`.

**Sourcing Cohort B (onboarded, not-yet-migrated) — the working recipe:**
Acacia is event-sourced in its OWN ESDB cluster (esdb+discover cdcheodo0aeg8kgkllb0.mesdb.eventstore.cloud:2113) and was initially NOT in buck-mcp conf. Fix: `proconf_populate {service:Acacia, environment:sandbox, repo_path: C:\Users\arpan\repos\ph\Acacia}` → creates node **`Acacia.Host.EventStore`** (nested under `host`; also `Acacia.Host.Postgres`, db `acacia`). Then:
- `esdb_list_streams` is DISABLED (expensive scan). Read the category stream instead: `esdb_read_stream {conf_node:Acacia.Host.EventStore, stream_name:"$ce", stream_id:"Acacia_Account", direction:backward}` → link events; distinct account ids are in each event's `metadata.$o` = `Acacia_Account-{accountId}`.
- Read each `Acacia_Account-{id}` stream (stream_name `Acacia_Account`, stream_id `{guid}`): events `AccountCreated` → `AccountContactUpdated` (has **Email** + **ThreePLId**) → `ShippingTermsAndConditionsAccepted` → **`AccountIsNowActive`** (= onboarded) → `AccountTypeUpdated`. accountId == Pistacia walletId.
- Cohort B = onboarded account whose (ThreePLId,Email) is NOT in `lazy_wallet_delta` AND whose accountId is NOT in `wallet_balance.wallet_id` (no wallet yet → import+GET fires fresh MigrateWallet).
- Found (2026-07-01, 3PL 61d0547f): walletId `c9f610aa-e779-4c55-90df-777e9fddb196`=e2e-guest+1782892602010@parcelhero.com; `894587ef-cb3a-43de-893f-9d8980505fcf`=e2e-guest+1782892496238@parcelhero.com; `bd4f7580-…198860`=ankit+11@parcelhero.com (dupe email with `4036b331-…722525` — same key, avoid). Test CSV at `C:\Users\arpan\tmp\buck-mcp\lazy-wallet-migration-test.csv`.

**Unleash flag-read (wired + verified 2026-07-01):** the Pistacia/wallet flags live in Unleash project **`aftersales`**. buck-mcp's `unleash_*` tools hit the **Admin API**, so the token MUST be the **admin** token: set `sandbox.buck-mcp.unleash.projects.aftersales.apiToken` = `refkey:unleash-aftersales-apiadmintoken` (NOT `apitoken` — the client/SDK token 403s "expected a different token type for this endpoint"; both secrets exist in `pvsandbox-keyvault`). Then call with `project: "aftersales"`, `environment: sandbox`. **The C# `Feature.EnableLazyWalletMigration.Name` maps to Unleash key `pistacia_enable_lazy_wallet_migration`** — verified `enabled: true` in dev + prod (ON). Flag ON also confirmed verbally by the team for sandbox + PROD.
