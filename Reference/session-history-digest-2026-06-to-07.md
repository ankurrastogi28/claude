---
name: session-history-digest-2026-06-to-07
description: "Consolidated knowledge digest mined from all prior Bodhi session transcripts (22 Jun → 20 Jul 2026) — identity/infra, the wallet-rebuild incidents, PR/ticket map, standing rules, memory corrections, open threads. Read to recover context."
metadata:
  node_type: memory
  type: reference
---

Consolidated digest of every prior Bodhi session transcript (22 June → 20 July 2026), produced 2026-07-21 by fan-out extraction agents (one per era). **Raw per-era extracts (full detail) preserved at** auto-memory `session-extracts/extract-{1-june,2-jul01-03,3-jul08-09,4-jul15-big,5-jul17-21}.md`. This note surfaces the load-bearing facts, the corrections to existing memory, and the still-open threads.

## Identity, Slack & infra (source-of-truth)
- Owner **Ankur (Rastogi)** = Slack `U03NK1M7G6R`. Peers: **Faruk** `U08Q952LB1C` (peer agent-runner, NOT owner — approvals are Ankur's), **Bucky** bot `U0AT6JCGXKK` (app `A0ASAPK7VPH`). **Bodhi** = `U0BBTT302FR` (Slack app `A0BCL14V69E`). GitHub PR-bot = `U01UM4WP1M2`. ⚠️ earliest session (a89f82d3) mislabeled Faruk as `U857LV9CM` — current mapping (Faruk=U08Q952LB1C) is correct; `U857LV9CM`/`U068HFJQWNM` are other teammates.
- Slack: team/workspace `T7QB2RM7X`; owner DM channel `D0BBTUXCATZ`; GitHub-notifications/PR channel `C099UBDF4UF`; general test/rebuild channel `C08EQJF2RM0`; chatbot-site-issue-report `C0A1SSX6XFY`; finance/report channel `C0B19SAPY7K` (Bodhi is `not_in_channel` there); report-bot `B0A2828K3G9`.
- Bodhi Slack config canonical at `~/.bucky/settings.json` (`slackBridge` block: enabled, ownerSlackId, ownerDmChannel, botToken `xoxb-…`, appToken `xapp-…`, inboxRetentionDays 30). **Secrets live only there — never copy tokens into the vault.**
- Platform = **Azure** (AKS `.azmk8s.io` + Postgres `.postgres.database.azure.com`). Sandbox Pistacia PG = PostgreSQL 11.22, db `pistacia`, user `pvadmin`. `CONNECT_TIMEOUT` to Azure PG / ESDB "fail to discover" while RabbitMQ stays up = **VPN gate, not outage** (use `env_health`). Prod PG needs VPN/bastion. buck response files: `C:\Users\arpan\tmp\buck-mcp\responses\<dd-MM-yyyy>\<uuid>.response`.
- **3 brand-level Stripe accounts** (PC / PH / PH_IE), one API key each via `GetApiKey`; customer lookups are brand-scoped. Moss non-CF/direct brand = **DeliverPlus** (`*.deliverplus.uk` → Azure-direct); PC/PH/PH_IE are Cloudflare-fronted.

## Major incident — the £426.41 prod wallet-projection data-loss (session 4, 2026-07-14) — CRITICAL
- Prod wallet rebuild (PR #198 runbook) looked "clean" on aggregate counts (within 0.3%), but an **independent `_bak`-vs-rebuilt diff on a business key** caught ONE wallet (`80b1d4d3…` "wallet80") that lost its `wallet_balance` row (£426.41 → absent) with a corrupted ledger.
- **Root cause:** `Projections/Extensions.cs` maps `WalletCredited` + `WalletMigrated` via nested `e.Customer.Id`. The nested `Customer` field was added 2025-05-13 (commit `1baa1f8`, PR #28). Events **before** that date deserialize `Customer=null` → `e.Customer.Id` throws NRE → the projection's `ManageFailures` **swallows** it → row silently drops from ledger AND balance. `WalletDebited` uses flat `e.CustomerId` (immune).
- **Fix = PR #217 / BUC-413** (re-adds `string? CustomerId` legacy field + mappers use `e.Customer?.Id ?? e.CustomerId`) → prod `pistacia-3.38.9` → re-rebuilt → wallet80 restored (57 ledger rows, £426.41).
- ⚠️ **STILL-OPEN GAP:** `WalletMigrated` was NOT fixed in #217 — `Extensions.cs` L27/L67 still use `e.Customer.Id`, and `WalletMigrated` also went flat→nested in #28. **Any pre-2025-05-13 `WalletMigrated` event will NRE+drop identically. BEFORE any future re-rebuild: apply the same `?? e.CustomerId` guard OR ESDB-confirm zero pre-2025-05-13 migrated events.**
- Lessons: (a) verify **per-wallet balances, not just aggregate counts** — keep `_bak` until the delta is explained; (b) **"Honeycomb clean" ≠ "replay clean"** (projections host has thin telemetry, `serilog.consoleenabled=false`); (c) replaying OLD events through NEW code — the decider is **when each event's field was born**, not tag-to-tag mapper diffs; (d) truncating the LIVE read-model 404'd ~40 wallets for the replay gap — prefer shadow-table+swap or a maintenance window.

## Wallet rebuild — v6→v7 mechanics (see [[pistacia-projection-rebuild-runbook]])
- v7 DELETE-checkpoints is **provable** from decompiled `Utile.EventStore.Projections` 9.10.2: a *present* category-stream checkpoint (even `0`) resumes `FromStream.After(revision)` and **skips event 1**; only a **null/absent** row gives a full replay. `$all` uses `FromAll.After((0,0))` (below event 1) so reset-to-0 replays. → post-#215 (category streams) you **DELETE** wallet checkpoints; the old `$all`-era v6 reset-to-0 was right only for `$all`.
- Checkpoint-shape = the #215 on/off signal: small revision values (74, 7408) = category-stream (working); ~179.8bn global positions = old `$all` (broken).
- `TRUNCATE … RESTART IDENTITY` (plain TRUNCATE won't reset the id sequence); `WalletRepository.Add` is a bare INSERT (double-counts on replay). Replay curve: ~15 min flat at 0 then 0→~72% over ~40 min ([[lesson-be-patient-with-replays]]). `Pistacia:Tenants` must be non-empty on the projections host (silent no-op heal if empty).
- **Prod healed 3PL ids:** PH `83d1191e-6451-42d7-af5f-209b79a4b0cc`, PC `335cd10c-7b7e-479a-8fa8-d227569d6601`, PH_IE `2c897ecc-fe94-4cf6-8a40-8d682c07b532`. **Sandbox 3PL ids:** PH `61d0547f-ec2b-4bd6-9cb5-5b74abaf3020`, PC `335cd10c…`, PH_IE `c473ae46-ff8b-46b2-94fe-df5344c9426d`. (PC `335cd10c…` is in BOTH.) Prod pollution NOT-IN query in [[wallet-pollution-check]]; 10-min anomaly+pollution cron standing; re-check ≈2026-07-28.

## PR / ticket map
- **BUC-336** = the 3PL-heal umbrella. #198→**BUC-389** (wallet leg / rebuild runbook), #215→**BUC-399** (category-stream `$ce-Pistacia_Wallet` narrowing), #200→**BUC-398** (payments leg; head `5765661`, 8 TenantId heal-wraps + 3 ingress guards, state-only heal, alerter per-process throttle → N× across replicas), #217→**BUC-413** (wallet hotfix). Morning prod deploy **3.38.2→3.38.8** carried #198/#215/#200; hotfix **3.38.9** carried #217.
- **#219 = BUC-404** (Stripe SetupIntent card management) — reviewed v1 (2026-07-17, request-changes) → v2 (merge-ready) → v3/v4/v5 (2026-07-21, this session). **#220 = BUC-429** (Utile XFF swap; ClientInfoController) merged into BUC-404.
- **Moss #187 = BUC-427** (real client IP; net8 upgrade deferred to **BUC-432**). **CF-Connecting-IP spoof = BUC-433** (see corrections). Pachira: #477 (BUC-403 invoice format), #484 (BUC-392 cancel adjustment invoice), #485 (BUC-421 v2 prepay revenue report).

## Standing rules / operating discipline (origins)
- **External write-actions READ-ONLY** (origin: session 6ba93c40, 2026-07-08 — Bodhi wrongly posted a GitHub comment + formal Approve on Pachira #477 off "👆" nudges; Faruk: "it can go sideways"). Ambiguous nudge ≠ authorization.
- **Always LIMIT SQL**; keep **PII/SEON tokens out of Slack**; **Ankur is owner, not Faruk**; **Slack mentions = `<@ID>`** (Slack only — NOT on waggle, use plain names, see [[waggle who-is-who]]).
- **Review checklist HARDENED (Faruk 2026-07-20):** (a) generic 10-point baseline in `checklists/`; (b) categorize brain notes by TYPE not default-`methodology/`; (c) **gating Pipeline/CI check — a red pipeline blocks `approve`, and every verdict must declare static-code-read vs build/CI-verified**; (d) 2026-07-21 additions: flag committed agent-scratch (`.tmp/`, `docs/superpowers/`); check new exception/type names vs sibling convention; **local verification = build + FULL test suite (unit AND integration), not unit-only**.
- **Watcher exit codes:** 0=message (read+relaunch), 1=crash (do NOT relaunch), **2=lock held by a live watcher (you're covered, don't relaunch)**, 3=idle `--timeout` self-exit (use block-forever, no `--timeout`), 130/143=intentional stop. Watcher self-trips on its own dismiss/reply/archive fs-events (empty inbox after a self-trip is normal). Two-session pattern: bridge read-only (separate session) + main for mutations. Spawned sub-agents have Bash/PowerShell + out-of-repo Read + `gh`/shell DENIED → do git/build verification in the MAIN session ([[subagent-permission-scope-git-verify]]).
- **buck reconnect** drops bridge/watcher (must re-arm); buck tools take a moment to re-register after reconnect (retry ToolSearch); `bucky`→`buck` rename + `.claude.json` edits need a **full restart, not `/mcp` reconnect** (reconnect keeps stale config/namespace). `memo-sync pull` via the **PowerShell tool**, not Bash.

## Corrections to existing memory (apply)
1. **Moss CF-Connecting-IP spoof was filed as BUC-433**, not BUC-412. BUC-412 = the impacted mandate-acceptance-IP work; BUC-433 = the security spoof ticket itself. (Grid-proven on sandbox: forged `CF-Connecting-IP` honoured on DeliverPlus; XFF + X-User-IP safe; CF brands unaffected.) Fixed in [[moss-cf-connecting-ip-spoofable-direct-brands]].
2. **WalletMigrated pre-2025-05-13 NRE gap** (above) — added to [[pistacia-projection-rebuild-runbook]] as a pre-rebuild guard.

## Still-open threads (as of 2026-07-21)
- **WalletMigrated hardening** (latent NRE, above) — not ticketed/fixed; guard before any re-rebuild.
- **Payments (#200/BUC-398) projection leg still HELD** — not rebuilt. `_bak` tables retained until owner drops them.
- **BUC-433 CF-Connecting-IP fix** filed (owners suggested Savvas/Evan), not implemented; Bodhi's offer to draft a walkthrough is outstanding.
- **#219 ES follow-ups** (this session): map 3DS-decline 500→4xx; churn the sun-set ES card write-path (`Pistacia_Cards` stream / `CustomerCardsState` / appliers / repo / consumer — consumer now non-fatal); ratify the Key Outcome 1 body-contract with FE.
- **EOM auto-report** — Pistacia-driven (no prod CronJobs); Sycamore `MonthEnded` V3-vs-V4 mismatch is the leading "never-fired" candidate, unproven; June report produced manually.
- **PRD-250** (wallet-status 404s innocuous; residual 403s + FE re-poll loop the real issue) root cause OPEN; **PRD-255** (empty-shipperId 400) handed to humans.
- **Sentry MCP** staged, pending restart+auth. **project-claude-md-fleet-draft** pending owner sign-off. **Stripe.js SetupIntent test frontend** brainstormed, never built.

Related: [[pistacia-projection-rebuild-runbook]] · [[wallet-pollution-check]] · [[moss-cf-connecting-ip-spoofable-direct-brands]] · [[remote-pr-review-methodology]] · [[generic-pr-review-checklist]] · [[card-management-setup-intent-key-outcomes]]
