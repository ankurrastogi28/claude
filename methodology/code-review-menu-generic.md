---
created: '2026-08-03'
tags:
  - methodology
  - code-review
  - pr-review
aliases:
  - Code Review Menu
  - Review sizes
  - S M L XL XXL XXXL review
type: methodology
description: "order-only heavier code-review instruments: sizes S–XXXL, ingredients, combination laws, locality ceiling"
---
> **Provenance & relationship:** generic, sanitised extract of an internal engineering practice (fleet-origin), saved to canonical home 2026-08-03 on Ankur's instruction. This is **NOT the default review** — the default is the single self-contained [[pr-review-rubric-generic]] run top to bottom. This menu is for when someone deliberately wants a *heavier* review and explicitly chooses how much heavier. Bodhi's run-mechanics: [[remote-pr-review-methodology]]; the everyday baseline: [[generic-pr-review-checklist]].

**Why this exists:** without a hard rule, reviewers quietly add "just one more cheap check" and a fixed default drifts into something bigger/slower than asked. The house rule that fixes it: **the scope of a review is set by whoever requests it, not widened unilaterally by whoever reviews.** If you think a heavier instrument is needed, say so and wait — never run it first and explain afterward. All verdicts remain advisory; a human merges.

## §0 — How to order
You order a **size**: S · M · L · XL · XXL · XXXL. Example phrasings: "Do an **L** on this PR." · "Do a **remote XL**." · "Do a **receipt** on that push." · "Do **consumers** only." (à la carte) · "Do a **pre-push**." Sizes work because everyone already understands small/medium/large intuitively.

Rules of the house:
1. **Orders are exact.** Ambiguous → ask; never widen or narrow yourself.
2. **Never substitute a cheaper thing for what was ordered.** An L ordered ≠ a lightweight "receipt", nor a summary of a previous L against different code.
3. **Never add an un-ordered ingredient.** Think one's warranted? Say so and wait.
4. **Locality is a hard constraint, not a preference.** Say which variant (local vs remote) you served.
5. **Every report opens by naming the variant served and what it excluded.** A remote review whose report doesn't say "these deeper instruments were not run — remote seat" is a failed review. Scope degradation is stated, never silent.

## §1 — The sizes
| Size | Adds | Buys you | ~Cost | Remote? |
|---|---|---|---|---|
| **S** | provenance + ticket check | Right code, does what the ticket asked. No deep review. | ~5 min | yes |
| **M** | + full checklist | One independent pass, full checklist. Everyday review. | ~30 min | yes (build/run items named as skipped) |
| **L** | + adversarial pass | ⭐ **The standard for anything being merged.** Two independent passes asking different questions, each blind to the other. | ~1 hr | yes (same caveat) |
| **XL** | + spec conformance + consumer-impact sweep | Contract-level scrutiny: claim-by-claim conformance + cross-repo blast radius. | ~1.5 hr | yes — **practical ceiling for a remote-only reviewer** |
| **XXL** | + measured coverage + flake measurement | What the tests actually execute + is the suite stable enough to trust. | ~2 hr | **no** — needs local build/suite |
| **XXXL** | + mutation testing | Full pre-merge arsenal: proof the tests would notice if the code changed. | ~2.5 hr | no |

**Aliases:** "full independent review" = L · "gate" = XXXL. Costs assume a small suite — flake & mutation scale with suite runtime (flake runs the whole suite many times); state your own estimate when ordering XXL/XXXL.

### Off the ladder
- **receipt** = provenance + delta check only. NOT a review — confirms a push landed byte-identical to an already-certified tree. Valid **only** on an unchanged, already-certified tree. Kept off the ladder deliberately so "order by size" can only order a real review; a lighter substitute must be asked for by its own honest name. If the tree carries genuinely new work, `receipt` is the wrong tool — say so.
- **pre-push** = the author's own self-check (§4) on their OWN unpushed work — a different moment and person, so it can't sit on a ladder measuring how thoroughly you reviewed *someone else's* code.

### Locality
**A remote-only reviewer's ceiling is XL.** Above it every added ingredient requires building/running locally (coverage, flake, mutation). Even XL isn't 100% API-only (spec-conformance reads the working tree), but it's the ceiling because nothing *above* it can be done remotely at all. A remote review is **not** "a local review with a caveat" — it's genuinely missing the instruments that make higher sizes worth ordering; never present them as equivalent. Anything a remote reviewer can do, a local reviewer can too → the local variant of any size is the complete one; remote is a defined, explicitly-named subset.

### Three rules that make sizes safe to order without much thought
1. **Anything actually being merged gets L as the minimum.** Asked for less on a real merge candidate → say so before doing it.
2. **Conditional add-ons, applied automatically at the right size and always reported as such:** a consumer-impact sweep whenever the diff touches an endpoint/contract/event (included from XL up) · a flake measurement before trusting any mutation result on an unmeasured suite.
3. **Sizes only go up, never sideways.** Each fully contains the one below → re-ordering L after M on the same code repeats the checklist for nothing; order just the missing ingredient (the adversarial pass) and say why.

## §2 — The ingredients (~10; order one directly only when no size fits)
| Ingredient | What it is | Depth | Where |
|---|---|---|---|
| **provenance/custody** | Confirm you're reviewing what you think: fetch actual head, compute the diff fingerprint yourself, check merge state, verify CI against the actual run, confirm clean tree. | cheap ~2 min | remote |
| **ticket** | Read linked issue + parents/siblings; check it delivers the stated acceptance criteria, not something adjacent. | cheap ~5 min | remote |
| **checklist ("rubric")** | One independent pass of the full common checklist. Asks "is this correct and complete?" | heavy ~20 min | either |
| **adversarial** | One independent pass to make it fail: comment-truth hunting, tests that cannot fail, attacking the write path. Asks "how do I break this?" | heavy ~40 min | either |
| **spec conformance** | Claim-by-claim against a spec's citable assertions. Distinct from ticket (that's AC-vs-outcome; this is anchor-by-anchor vs a written spec). | med-heavy | mixed |
| **consumers** | Org-wide consumer-impact sweep for changed endpoints/contracts/events, limits stated. | medium | remote |
| **coverage** | Touched-line coverage of the diff, per file, measured not asserted. | medium | local only |
| **flake** | Suite stability: N runs on new code + N on baseline, report rates. | heavy | local only |
| **mutation** | Deliberately mutate load-bearing code with a positive control; record what the suite catches. | heavy, build-bound | local only |
| **delta** | Scope check vs a previously reviewed version: what moved, did anything move outside agreed scope. | cheap | either |

**Locality rule underneath:** anything that must **build or run** the code is local-only; anything reading a hosted git/issue/CI API is remote-capable. checklist & adversarial are "either" because most work is reading — but the moment an item says "run the suite," that item is local and a remote reviewer must name it as not-run. Keep the list ~ten; new instruments go *inside* an existing ingredient's body, not as new rows.

## §3 — Combination laws (each reflects a real, corrected mistake)
1. **A lightweight provenance/delta check is never a substitute for a real review on genuinely new code.** It tells you *what* changed, not whether it's *right*. Refusing an inappropriate substitution is your responsibility even if another party handed you the narrowed scope.
2. **Mutation testing belongs to the adversarial pass alone.** Running it under both checklist and adversarial doubles the most expensive instrument for nothing.
3. **Never trust mutation results on an unmeasured suite.** A flaky suite manufactures false "mutant killed" results. If flake shows instability, every mutation survival/kill needs a re-run before reporting. (Why coverage/flake sit below mutation: measure the suite before trusting what it catches.)
4. **A consumer sweep can't see what the current PR introduces, nor any repo outside the search index.** Default-branch-only indexing → a symbol from an unmerged PR returns zero hits even in its own repo (a false "no consumers" identical to a genuine clean sweep). Sweep base-branch surfaces (route, response type, event type); close out new symbols by reasoning from source. Name which platforms your tool covers; if part of the codebase lives where it can't reach, say so — an unstated blind spot is a false all-clear.
5. **Freeze the code under review while any independent-pass or mutation instrument runs.** Mutation writes-and-reverts; a concurrent edit can clobber a revert or corrupt a result. Announce and release the freeze explicitly.
6. **An independent checklist pass without an adversarial pass trades depth for completeness, and vice versa.** They answer different questions. Ordering the lighter is legitimate; presenting it as if it covered the heavier is not.

### The rule underneath all the laws
> **A check whose passing output is indistinguishable from a broken instrument's output is not evidence.**

This is why mutation needs a positive control, flake measures instead of asserts, a consumer sweep states its limits, and a green suite proves nothing until shown capable of turning red. Real defects have hidden behind: a match clause that never matched, a query scoped to the wrong context, two greps sharing one pattern list, a query defeated by an undocumented data envelope, and a large suite that survived multiple intentional mutations catching neither.

## §4 — Ingredient bodies (the checklists)
**provenance/custody** — fetch actual head via the platform's CLI/API (not a raw unauth clone); compute the diff fingerprint yourself (`git diff <base>..<head> | sha256sum`); check merge state first (don't review a merged PR unless asked); base/conflict check (`git rev-list --count <head>..origin/<target>`; `git merge-tree $(git merge-base …) <head> origin/<target>`); verify CI against the actual run for the actual head commit; confirm clean tree before & after.

**ticket** — read linked issue + parents/siblings + its other linked PRs; verify the stated business requirement, not something adjacent; find deploy-coupling ("needs a flag first", "don't deploy in isolation"); no issue = a named gap. Belongs to the dispatcher, folded into the brief.

**checklist ("rubric")** — one independent pass, briefed with acceptance criteria and NO prior findings. Runs the common core (scope, correctness+idempotency, tests, error handling, security incl. new log statements, readability, performance, migration-in-same-change, regression/blast-radius, diff hygiene, docs) + PR-side additions (blast-radius scan, deploy-vs-code, domain checklist) + cross-cutting lenses (conventions fit; regression scan; **enumerate publishing surfaces not only computing ones**). Full detail: [[pr-review-rubric-generic]].

**adversarial** — one independent pass briefed specifically to break it (not the checklist twice): mutation protocol when the size includes it (local); **comments as claims** — find one false as written (a wrong justification outlives a wrong conclusion because it gets quoted, not re-tested); **tests that cannot fail** (assertions true by construction, shapes production can't produce, substring where exact was available, over-claiming test names); **attack what the code writes** (persisted records, event payloads, rollback direction, silent rewrite-on-save of legacy data); **replay & rehydration** (old events through new code, old data through new readers); **measure, don't assert** flakiness (N runs each on new + baseline, local); **verdict discipline** — a confident wrong finding is worse than none; "attacked and could not break it" is a required section carrying as much weight as findings.

**spec conformance** — resolve every in-code citation to a spec (a citation naming no file, or a file that exists multiple times, is itself a finding); check anchor-by-anchor; if the spec is an external attachment, say so rather than let a reader assume it was verified; if no line-anchored spec exists, this degenerates to the ticket check — say so, don't run both.

**consumers** — mandatory whenever a PR changes an endpoint/contract/event, at any size; search org-wide on route + response type + event type name; triage every hit by repo; additive optional/nullable = informational not silent; obey Law 4 and state search limits; exclude test-harness repos holding copies of others' contracts (inflate blast radius); name which hosting platforms the search covers — "sweep covers [X] only; consumers on [Y] require manual enumeration and were not swept."

**coverage** (local) — instrumented run scoped to the production files the diff touches, per-file line numbers; measured never asserted; explain a legitimate 0% (inert declarations measure zero and should — don't build artificial paths just to raise a number); remember coverage says a line executed, mutation says a test would notice it changed.

**flake** (local) — N runs on new + N on baseline in a disposable env, actual rates; name every flaking scenario individually; classify pre-existing vs newly introduced; say plainly if CI may go red independent of the change.

**mutation** (local) — pick the properties/branches the change's correctness rests on; include ≥1 positive control you expect caught (if it survives, the instrument is broken → report that, not a false "no issues"); distinguish equivalent mutants from real coverage holes; watch for stale-build artifacts after restoring a mutated file (identical failure counts across different mutations is the tell); revert every mutation, verify clean tree; obey Laws 3 & 5.

**delta** — what moved between the reviewed version and this one; anything beyond agreed delta is a finding; then re-read Law 1 (delta is an add-on, never a substitute).

**pre-push** (author's own, local, off-ladder) — full suite green locally (not "CI will catch it"); clean build, no unexplained new warning; clean working tree (a wrongly-generated file gets the ignore-list fixed in the same change); **read your own full diff top to bottom before pushing** (debug leftovers, TODOs, secrets — cheapest review, most skipped); commit hygiene; branch sanity.

## §5 — Severity, verdict, publishing
**Severity:** Blocking (correctness/security/data-integrity to fix before merge; on a merged PR = "must-fix follow-ups") · Non-blocking nit. **Deploy-vs-code:** a finding can be a deploy-blocker without being a code blocker (correct code unsafe to ship as-is/alone — needs a flag/migration/dependency first); tag separately, surface prominently; check the target branch first (if the prerequisite already merged there it merely depends on shipped work). **Verdict:** `approve` · `approve-with-nits` · `request-changes` (≥1 blocking code finding); a deploy-blocker alone doesn't force request-changes.

**Evidence bar (non-negotiable):** every finding carries file:line + a verbatim quote/command output; every finding carries a concrete failure scenario (specific inputs/state → specific wrong outcome); no scenario = a guess, label or drop it; checklist ≠ substance.

**Dispatcher/orchestrator duties:** run the ticket check first, fold AC into every brief; brief every reviewer blind (no prior findings/verdicts); verify every finding at source before publishing, drop what doesn't hold; if >1 reviewer ran, declare disagreement openly (never a silent union); assign severity only after checking; open the report with the variant served + exclusions (house rule 5).

## Living document
Each real review teaches something — a new instrument goes inside an existing ingredient's body, a new lesson becomes a combination law. Keep the ingredient list ~ten and the sizes S through XXXL.
