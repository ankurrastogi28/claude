---
name: generic-pr-review-checklist
description: Project-agnostic generic PR review checklist (Bucky's version,
  2026-07-20) — the BASELINE applied to every PR review before any per-BC
  rubric. Faruk-directed durable baseline for Bodhi.
metadata:
  node_type: memory
  type: reference
---
**Baseline for every PR review.** Faruk (`U08Q952LB1C`) directed 2026-07-20: always run this project-agnostic checklist as the baseline on any PR, in any repo/language, BEFORE layering the Parcelhero per-BC rubric on top. Applies whether or not a BC-specific list exists. Source: Bucky's generic checklist, captured verbatim-faithful.

## Generic PR Review Checklist
1. **Scope** — matches its ticket/intent; nothing unrelated, no gold-plating, nothing quietly dropped. Could it be smaller?
2. **Correctness** — logic sound; edge cases handled (null/empty/boundary/error paths); no off-by-one; concurrency/ordering safe.
3. **Tests** — new behaviour + edge cases covered; tests meaningful (not trivial/tautological); all green; any flaky/skipped called out.
4. **Error handling** — failures handled + logged, not swallowed; graceful degradation; clear messages.
5. **Security** — inputs validated; no injection; authn/authz on new endpoints; no secrets/PII in code or logs.
6. **Readability & conventions** — clear names; follows the repo's existing patterns; no dead/commented-out code; complexity reasonable; comments only where they earn it.
7. **Performance** — no obvious N+1, unbounded query/loop, or needless work on hot paths.
8. **Regression / blast-radius** — impact outside the diff; backward-compatible API/contract/schema changes; migrations forward-safe; config/env parity.
9. **Docs** — README/API/config updated where the change warrants.
10. **Pipeline / CI (gating — added 2026-07-20)** — check the PR's build/CI status; it's part of our GitHub PR merge checks, so a **red pipeline blocks `approve`**. State whether the verdict is a *static code-read* or *build/CI-verified* — never let `approve` imply green CI. When the repo is cloned locally, prefer actually running the build/tests before approving.
11. **Verdict (advisory — a human merges)** — `approve` / `approve-with-nits` / `request-changes`; every finding gets file:line + evidence + severity (blocking vs nit). And confirm the linked ticket's acceptance criteria are actually met, not just "it compiles".

## How it fits the wider methodology
- This is the **baseline layer**. On top of it, every ParcelVision review still adds the 3 cross-cutting dimensions and the matching per-BC checklist — see [[remote-pr-review-methodology]] and the per-BC notes (e.g. [[pr-review-bc-pistacia]]).
- Severity/verdict model is shared with the methodology note: **blocking** (correctness/security/data-integrity) · **nit** (style/convention) · **`deploy-blocker`** (code fine but unsafe to ship as-is/alone — check the base branch before calling a missing dependency a blocker).
- Advisory only — a human merges; never mutate GitHub ([[external-mutations-read-only]]).

Related: [[remote-pr-review-methodology]] · [[bodhi-orchestration-playbook]] · [[feedback-separate-reviewer]]

## Addendum — committed agent-scratch artifacts (lesson 2026-07-21, Pistacia #219)
When reviewing PRs produced with AI-agent workflows, **scan for committed working/scratch material** as part of item 1 (Scope) / item 6 (Readability):
- `.tmp/` scratch (build/test-run captures, logs) — often multi-MB; should be gitignored.
- **`docs/superpowers/` — the superpowers skill namespace** (`plans/*.md` = agent implementation plans, `specs/*.md` = design/edge-case docs). Agent working artifacts, NOT production docs — flag like `.tmp`. The `plans/` doc is always scratch; `specs/` may have design value but belong in a real docs path, not the skill namespace.
- Check: `git ls-tree -r origin/pr/<N> | grep -E '^\.tmp/|docs/superpowers/'`; confirm not on `origin/main` (PR-introduced) and gitignored.

## Addendum — local verification must be build + FULL test suite (standing rule, Faruk 2026-07-21, Pistacia #219)
Before calling any change "green / safe to push", **build the full solution AND run the FULL test suite locally — unit AND integration — not just the changed/unit tests.** A rename or contract change can break a *pre-existing* test elsewhere that a unit-only run never touches.
- Concrete miss on #219: the amex error message was generalised (`"American Express cards are not accepted."` → `"Card brand amex is not supported."`); a pre-existing **integration** test `CreatePaymentTests.Create_Returns400_WhenPaymentIsCreatedWithAmex:290` still asserted the old string. My local run was build + unit only; Neha ran only the *new* integration tests. Neither ran the full integration suite → it slipped to CI red.
- **If the integration suite can't be run locally** (needs infra/app-host/Stripe-test-env), say so explicitly and mark the verdict **"unit-verified, integration UNVERIFIED — pending CI"** — never imply green off a unit-only pass. Don't over-lean to "expect green."
- Ties into checklist item 10 (CI gating): a green *local unit* run is NOT a green CI.
- **Lesson:** on #219 I read the `docs/superpowers/specs/` files as legitimate spec evidence across v3/v4 instead of flagging the whole tree as committed scratch — Faruk caught it. Fix = `git rm -r` + gitignore `docs/superpowers/` (ideally global gitignore).

## Addendum — new type/exception names must match the sibling convention (lesson 2026-07-21, Pistacia #219)
Under item 6 (Readability & conventions) / the "match the nearest exemplar" cross-cutting dim: **when a PR adds new exception/DTO/type classes, diff their names against the existing siblings in the same folder.** A PR that adds two classes in the same namespace with *different* shapes is introducing the inconsistency itself.
- Concrete miss on #219: PR added `CardNotFoundException` + `CardDeletedException` (Card-prefix) alongside `ExpiredCardException` + `AmexCardException` (descriptor-then-`CardException`). Faruk caught the split; I'd passed it in v3/v4. Standardise on one shape (here the `Card…` prefix → `CardExpiredException`, `CardAmexException`).
- Check: `ls src/**/Exceptions/` (or the relevant type folder) and eyeball the naming pattern; flag any new class that doesn't match the dominant sibling shape.

## Addendum — Consumer Impact Analysis is MANDATORY on contract-surface changes (standing rule, Faruk 2026-07-30, Pistacia #225)
**A ParcelVision(Parcelhero)-wide Consumer Impact Analysis is a REQUIRED checklist item whenever a PR changes an endpoint, a contract (request/response DTO or published event), or an integration event.** *Additive optional/nullable field changes ARE backward-compatible and OK — but still REPORT them as **informational** (note them in the sweep; don't raise as blocking/deploy-blocker).* A local/BC-only check — or a single-consumer check like "just Cocobolo" — is NOT sufficient; cross-BC consumers won't surface. This makes checklist item 8 (Regression/blast-radius) + the per-BC §7 downstream-scan explicit and non-skippable for contract-surface changes.
- **Method:** `gh search code --owner ParcelVision "<pattern>"` (read-only) with appropriate pattern(s) — the endpoint path (e.g. `"/v1/payments/"`), AND for the exhaustive pass the **response/event TYPE name** (URL-path search alone misses consumers that build the URL from base+segments or use a generated client).
- **⚠️ Sweep the EXISTING surface, NEVER a newly-added field/symbol name (refinement, 2026-07-30, #225).** `gh search code` indexes **default branches**, so any symbol introduced by the *unmerged* PR returns **0 hits regardless** — a zero-hit sweep on a *new* field looks identical to "swept and safe" but is uninformative/false-comfort. Search the route, the response TYPE, and the event TYPE (all pre-existing on `main`); then confirm the *new* field's handling from **source/logic** (a field absent on `main` cannot be read by any existing consumer — verify at the consumer's DTO, not by searching its name).
- **Triage each hit by repo/consumer:** does it actually READ the changed field, and does the SPECIFIC change alter its behaviour? Worked example (#225 self-heal): org search → Pachira `PrepayInvoiceGenerator` branches `payment.Status == "Completed"`; the heal only moves status `PartiallyRefunded↔Refunded`, never `"Completed"` → inert. That converts "consumer risk assumed" into "consumer risk answered".
- **State the limit:** `gh search code` is index-backed, literal, and capped — report as "every consumer discoverable by these patterns", not proof of totality.

## Addendum — verify a repository READ's null/empty contract at SOURCE, not from tests (lesson 2026-08-05, Pistacia subscriptions)
Under item 2 (Correctness — null/empty paths) + item 3 (Tests): **when a handler calls a repo read (`GetById`/`Find`/…) and dereferences the result, verify what the repo ACTUALLY returns for a missing row/stream — at the source (impl/decompile) or via an existing consumer's handling — never infer it from the handler's own tests.**
- Concrete miss (subscriptions batch): event-sourced `GetById` returns **`null`** (not an empty aggregate) for a never-seen stream; four membership handlers dereferenced it → NRE on the common no-membership path (the bus consumer's version poisons the queue). It hid because the unit tests **mocked `GetById → new State()` (non-null) — a contract production never honours** → guard-tests passed green over a live NRE.
- **Tell:** an existing sibling that DOES null-check the same read (here the logistics `CheckoutSessionCompletedHandler` null-checks `GetById`) proves the contract is nullable; a mock returning a non-null empty object is a red flag to verify at source.
- **Fixes, strongest first:** make the read return `Task<T?>` so the COMPILER forces null-handling at every current+future site (beats guarding N sites); mock the real `null` in tests and watch them go red-before-green.
- Same family as the S1 lesson (verify the wiring/registration at source, not the code shape): **assumed-contract vs verified-contract bit twice in one day — verify reads AND registrations at their source.**
