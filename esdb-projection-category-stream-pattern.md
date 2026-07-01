---
name: esdb-projection-category-stream-pattern
description: "ESDB projections should subscribe to their $ce-<Category> by-category stream, not raw $all — how/why in Pistacia"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 67e2900b-61a8-4497-b778-32ca1a94b620
---

**Lesson (Ankur asked me to internalize this, 2026-06-29): an ESDB projection should subscribe to its by-category stream `$ce-<Category>`, NOT raw `$all`.** Subscribing to `$all` makes the ESDB server ship *every* event to the client and the projection discards the irrelevant ones in-process (client-side) — wasteful at Pistacia's ~179bn-event `$all` scale.

In `ParcelVision.Utile.EventStore.Projections` each projection has its **own** subscription source (no single shared filter). Set it per-projection with `.UsingCategoryStream("$ce-Pistacia_Wallet")` in `Pistacia/src/Pistacia.Projections.Host/ServiceCollectionExtensions.cs` — chain position: after `.ManageFailures(...)`, before `.Handlers(...)`. With no override the library defaults to the unfiltered `$all` subscription.

- ESDB `$by_category` splits the stream name on the **first `-`**, so `Pistacia_Wallet-<id>` → category `Pistacia_Wallet` → `$ce-Pistacia_Wallet`. `$ce-Pistacia` does NOT exist (the category is `Pistacia_Wallet`, not `Pistacia`).
- Payment projections (`pistacia_payment_details`, `pistacia_payments`) already used `$ce-Pistacia_Payment`; the 3 wallet projections were still on `$all` and got narrowed to `$ce-Pistacia_Wallet` in **PR #215** (ParcelVision/Pistacia, BUC-336 follow-up, by Ankur).
- **⚠️ Checkpoint semantics differ by subscription type — I got this WRONG first, corrected by Bucky + verified live 2026-06-29.** A category-stream subscription checkpoints by **stream REVISION**, not global position. Verified: `pistacia_payment_*` checkpoints = `131310` (a `$ce-Pistacia_Payment` revision), wallet checkpoints = `~179.8bn` (a global `$all` position). The library (`Utile.EventStore.Projections` `GrpcEventsProjector`) resumes a category sub with `FromStream.After(StreamPosition(commit_position))` and only falls back to `FromStream.Start` when the row is **absent (null)**, not when it's `0`.
- **Consequence — switching an EXISTING `$all` projection to a category stream is NOT seamless** (this is the bit I botched). The stored global position (~179.8bn) read as a stream revision lands past the end of the ~tens-of-thousands-event category stream → projects nothing. So **#215 (wallet narrowing) MUST ride a checkpoint reset + replay; it cannot deploy independently.**
- **Reset for the category case = DELETE the 3 wallet checkpoint rows, NOT `UPDATE … = 0`.** `After` is exclusive, so a row at `0` → `After(0)` skips the stream's first event; only a null/absent row → `FromStream.Start` (true full replay). This is the *opposite* of the `$all`/`FromAll` rebuild, where the owner said reset-to-0-not-delete — because `FromAll.After((0,0))` *does* replay from start. **Different subscription type → opposite reset rule.** See [[pistacia-projection-rebuild-runbook]].
- Side benefit (still true): after the reset+replay, future wallet replays are far faster — ESDB ships only `$ce-Pistacia_Wallet`, not the whole `$all` firehose.
- **Source-verified 2026-06-29** (decompiled `Utile.EventStore.Projections` 9.10.2 `GrpcEventsProjector` via `ilspycmd`): `SubscribeToStream` → `checkpoint==null ? FromStream.Start : FromStream.After(new StreamPosition(commit_position))`; `SubscribeToAll` → `checkpoint==null ? FromAll.Start : FromAll.After(new Position(commit,prepare))`. Confirms: category row at 0 skips event 0 (delete instead); `$all` row at (0,0) replays from start (reset-to-0 fine). Package in nuget cache: `~/.nuget/packages/parcelvision.utile.eventstore.projections/9.10.2/lib/net7.0/*.dll`; decompile with `dotnet tool install -g ilspycmd --version 8.2.0.7535` then `ilspycmd <dll> -o <dir>`.
- The post-#215 rebuild runbook is **v7** (DELETE the 3 wallet checkpoint rows); v6 (UPDATE…=0) was for the `$all` era. See [[pistacia-projection-rebuild-runbook]].
- **Lesson:** this is exactly why [[feedback-separate-reviewer]] matters — I asserted "no replay needed" from the authoring seat; an independent reviewer (Bucky) caught it against the SDK source + live checkpoint magnitudes, and I then confirmed it by decompiling the library myself. Verify resume semantics against the actual library/data before claiming "seamless".

Verifying a PR like this without `gh` auth: `git -C <repo> fetch origin pull/<N>/head:pr-<N>` then `git diff main...pr-<N>` (creds were cached for ParcelVision/Pistacia on 2026-06-29).

Related: [[pistacia-projection-rebuild-runbook]], [[buck-mcp-launch-config]].
