---
created: '2026-08-03'
tags:
  - checklist
  - code-review
  - pr-review
  - methodology
aliases:
  - PR Review Rubric
  - Generic PR review rubric
type: checklist
description: "self-contained PR-review rubric run top-to-bottom: meta-rules → common core → PR-side → cross-cutting → domain → verdict"
---
> **Provenance & relationship:** generic, sanitised extract of an internal engineering practice (fleet-origin), saved to canonical home 2026-08-03 on Ankur's instruction ("digest ... save it in canonical home"). This is the **fuller rubric — the DEFAULT single review, run top to bottom.** The condensed [[generic-pr-review-checklist]] (Faruk's 10-point baseline) is the distilled everyday version of §1 here; [[remote-pr-review-methodology]] is Bodhi's how-to for *running* a review (read-only on GitHub, separate objective reviewer + verifier, verdict in the Slack thread); the *heavier* order-only instruments (S–XXXL) live in [[code-review-menu-generic]]. Per-BC rubrics: `methodology/pr-review-bc/`. To evolve the practice, edit the right ONE of these — don't fork a fourth.

This rubric is a self-contained checklist for reviewing a pull request: one document, run top to bottom. It layers: **meta-rules → common core → PR-side pass → cross-cutting dimensions → domain-specific checklist → verdict.** All verdicts are advisory — a human merges. The reviewer's job is to surface findings and a recommendation, not to gate the merge.

## §0 — Meta-rules (what makes a review trustworthy)
- **Evidence or it didn't happen.** Every finding carries a file:line reference **plus a verbatim quote or command output.** A reviewer who cannot cite a specific line did not verify the claim. A "verified" citation can still be an over-read — quote the actual text, don't rely on line numbers alone.
- **Checklist ≠ substance.** Ticking every box with a shallow pass is a failed review. Each item is a question to *answer with evidence*, not a box to tick. Multiple passes help only if each uses a **different lens** (correctness / contract / security) — repeating the same skim adds no value.
- **Prefer a reviewer blind to prior findings.** A reviewer shown a prior verdict tends to confirm it rather than re-derive it. Treat the PR's own description as *claims to be verified*, not fact. If you run more than one pass, diff them and report disagreements openly rather than presenting the union as agreement.
  - **Corollary: verify a CI-failure claim against the actual build log for that run, never against a reviewer's own local test run** — a local run may not cover the same tests/projects CI runs.

## §1 — Common core (every review)
1. **Scope** — matches its stated intent/ticket; nothing unrelated smuggled in; nothing quietly dropped. Could it be smaller?
2. **Correctness** — logic sound; edge cases (null / empty / boundary / error paths); no off-by-one; concurrency & ordering safe; **idempotency wherever events, messages, or retries exist.**
3. **Tests** — new behaviour & edge cases covered with *meaningful* assertions, not tautologies; all green; flaky/skipped tests named.
4. **Error handling** — failures surfaced and logged, never swallowed; graceful degradation; messages actionable.
5. **Security** — inputs validated; no injection; authn/authz on new surfaces; **no secrets, tokens, or PII in code, diff, config, OR new log statements** — check what the new logging actually prints. Audit/security-linter suppressions and committed credentials are **MUST-FLAG**, never waved through as "out of scope"/"pre-existing".
6. **Readability & conventions** — clear names; matches the nearest existing exemplar; no dead/commented-out code; no debug leftovers; reasonable complexity; comments only where they earn it.
7. **Performance** — no obvious N+1, unbounded query, or needless work on a hot path.
8. **Schema/data changes ship their migration in the same change** — forward-safe, tolerant/nullable for legacy rows.
9. **Regression / blast-radius** — impact outside the diff; backward-compatible API/contract/schema; migrations forward-safe; config & per-environment parity.
10. **Diff hygiene** — no unintended file-mode changes; no mixed line endings (respect the project's config); no churn-only reformatting hiding the real diff.
11. **Docs** — README / API / config docs updated where warranted.

## §2 — PR-side pass (reviewing a submitted PR)
1. **Ticket/issue first (mandatory)** — read the linked issue + parent/sibling issues **before** the diff; confirm acceptance criteria are actually met, not "it compiles". No linked issue = a named gap, not a pass.
2. **Check merge state first.** A merged PR is a post-merge review: findings become follow-ups, not gates. Don't review an already-merged PR unless asked.
3. **Base/conflict check** — is the branch behind its target, and does it merge cleanly? Report both, every time. Behind-ness: `git rev-list --count <head>..origin/<target>`. Conflicts: `git merge-tree $(git merge-base <head> origin/<target>) <head> origin/<target>`. A green "merge result" CI run only proves this at the moment it ran — the synthetic merge commit is stale once the target moves.
4. **Blast-radius scan** — what breaks *outside* the diff: callers, consumers in other services/repos, event/message contracts, derived read models & replays, per-environment config parity.
5. **Consumer-impact analysis — mandatory whenever a PR changes an endpoint, contract, or integration event.** Consumers live elsewhere; reading the diff's own repo isn't enough. Org-wide code search on patterns consumers actually reach (the route path, and the response/event **type name**). Triage every hit by repo: *does this change alter that consumer's behaviour?*
   - Additive optional/nullable fields are generally backward-compatible — report them **informational**, never silently.
   - State the search's limits: index-backed, literal-match, result-capped, default-branches-only → a symbol from an *unmerged* PR returns zero hits even in its own repo. A zero-hit on a brand-new symbol is an index artifact, not evidence of no consumers. Sweep surfaces that already exist on base (route, response type, event type); reason about the new field from source.
   - Name which forges/hosts your search tool covers; an unstated blind spot is a false all-clear.
6. **Pipeline state** — did CI pass on the PR's actual head commit, and are all required checks present? A red build isn't automatically a blocker but must be named and diagnosed to a specific failing test.
7. **Deploy-vs-code distinction** — see §5.
8. **Verdict: advisory** — findings ranked blocking-first; a human merges.
9. **Domain-specific checklist** — run whatever project/domain checklist applies (§4).

## How a review runs (roles keep it honest)
- **Orchestrator/dispatcher** reads the linked issue first, folds acceptance criteria + deploy constraints into the reviewer's brief. For related PRs, discover dependency direction, review the dependency first, reuse the reviewer for both.
- **Reviewer** reviews under the brief, returns findings each with file:line + evidence.
- **Verifier** (ideally a fresh pass) checks each finding against source before anything is published.

Order per PR: **§0 → §1 → §2 → §3 → §4 → §5.**

## §3 — Cross-cutting dimension 1: Conventions / standards fit
Does the PR follow *this codebase's* patterns? Universal rule: **match the nearest existing exemplar; don't invent a new shape.** Watch for:
- Business/domain logic at the wrong altitude (in a controller/handler instead of the domain model).
- Mutating a persisted event/contract in place instead of adding a new versioned one.
- Bypassing the codebase's safe accessor/abstraction for a raw/ad-hoc one.
- Missing the layer-matching tests the conventions expect (unit at the logic layer, integration at the wiring layer).

## §3 — Cross-cutting dimension 2: Regression / side-effect scan
What could break *outside the diff*?
- **Events / messages / contracts:** does a changed event ripple to every consumer/handler/projection that reads it?
- **API contract:** do status codes or response shapes change for callers? Status-code changes drive retry logic everywhere.
- **Read-model / projection replay:** does the change alter how an existing derived value is computed → replay-safe change or rebuild needed?
- **Per-environment parity:** mirrored across every environment's config? "Works in one, missing in another" is a classic regression.
- **Numeric/monetary math:** rounding, truncation, sign, unit changes ripple downstream. Never let values in different units (e.g. currencies) meet in one calculation without explicit conversion.
- **Enumerate the surfaces that *publish* a derived value, not only those that *compute* it.** A re-deriving surface picks up a corrected formula for free; a *publishing* surface (emitted event, webhook, notification payload, DTO mapper) may carry a hard-coded/cached copy and keep shipping the wrong answer silently, even after every computing surface is fixed and verified to agree. Ask: what *transmits* this value outward, and does it re-derive or restate? *(This is exactly the BUC-450 / BUC-447 refund-reporting failure mode — see [[buc-450-swapped-refund-ids-state]], [[pr225-buc447-review-state]].)*

## §3 — Cross-cutting dimension 3: Read the linked issue (required)
A code-only read misses acceptance criteria & deploy constraints. Mandatory — ideally by the dispatcher, folded into the brief.
1. Get the issue ref from branch name / PR title / PR body.
2. Pull the issue **and its parent + sibling issues** for acceptance criteria, required tests, cross-cutting constraints. Check its other linked PRs — an issue often spans several.
3. Look for deploy-coupling the code won't tell you: "don't deploy in isolation", "needs flag X first", "these tests required", "depends on another service shipping first".
4. Verify the implementation delivers the stated **business requirement**, not merely something adjacent. A ticket-vs-implementation gap is one of the highest-value findings a review produces.

If no issue is found, say so explicitly — a stated gap, not a silent pass.

## §5 — Severity / verdict model
Verdicts are advisory — a human merges.
- **Blocking** — a correctness/security/data-integrity defect to fix before merge. (On a merged PR: "must-fix follow-ups".)
- **Non-blocking (nit)** — style, naming, minor convention drift, optional improvement.

**Deploy-vs-code distinction (call out explicitly):** a finding can be a **deploy-blocker without being a code blocker** — correct code that's unsafe to merge/deploy *as-is, alone* (needs a flag, migration, another service first). Tag `deploy-blocker` separately and surface it prominently — usually from the linked-issue check. Before tagging: check the target branch — if the prerequisite already merged there, it merely *depends on already-shipped work*, not a blocker.

**Verdict (recommendation only):** `approve` (no blocking findings) · `approve-with-nits` (only nits) · `request-changes` (≥1 blocking **code** finding). A `deploy-blocker` does **not** by itself force `request-changes`; surface it as a separate prominent flag and let the human decide ordering.

Every posted review states: verdict, blocking findings (file:line), deploy-blockers, nits — and notes explicitly that a human makes the final merge call.

## §4 — Domain-specific checklists
Generic checklists can't capture everything. Maintain a short, living per-service checklist built from real review lessons (see `methodology/pr-review-bc/`), typically:
- The 2–4 recurring "hotspot" areas where the component has actually broken (a lock-ordering rule, an idempotency guard, a status-code contract).
- Any place where two things must change together (a state-transition handler + its read-model/projection) and reviewers keep missing the coupling.
- Domain-specific correctness rules (currency/unit handling, versioning of persisted events, multi-tenancy isolation, authorization-policy naming).

Where no checklist exists for a component, say so explicitly and fall back to §1 + targeted domain checks. Write the missing checklist once a real review justifies it.

## Living document
Each real review teaches something — a new hotspot, a convention missed, a false-positive pattern. Fold it back (here and in the per-BC checklists). Verdicts stay advisory; the rubric makes the human merge decision well-informed, not gated.
