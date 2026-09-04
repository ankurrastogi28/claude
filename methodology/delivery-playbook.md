---
type: methodology
tags:
  - delivery
  - playbook
  - fleet
  - quality
  - review
adopted: 2026-09-04
source: fleet-shared brain, taught by peer agent (Bucky) at Faruk's direction
caveat: the §1 size table and §3 ingredient table must stay in sync — change
  both if adapting either
---
> **Adopted 2026-09-04** from the fleet-shared brain (taught by the peer agent Bucky at Faruk's direction). This is the full self-contained reference (replaces my earlier summary). Bodhi follows this.
> **⚠️ Maintenance caveat:** the **§1 size table** and the **§3 ingredient table** must AGREE — if you adapt either for our fleet, change BOTH. That pair drifting apart is the failure this document is most prone to.
> Note: the delivery S/M/L/XL ladder is DISTINCT from the review-size ladder (same letters, different meanings) — don't cross them.

# Delivery playbook

The canonical delivery pipeline: never-cut guarantees, the phase spine, roles, and comms.

This is a self-contained, generic copy — internal names, vault paths and cross-note links have been
resolved into plain text so it stands alone. Living document: new scars fold in; the ingredient list
stays capped.

---

## A. The never-cut list — in EVERY mode, no exceptions

The reason modes are safe to order half-awake. No size removes:

1. **Money-semantics verification** — any change touching money figures gets its arithmetic proven, not read.
2. **Full suite green before hand-off** (timezone pinned where the suite requires it) — a red or unrun suite is never handed to the owner.
3. **Owner owns git** — tree ends UNCOMMITTED; never commit/push; no forge writes; no AI-attribution trailers.
4. **Owner gates** — go-word per artifact (urgency ≠ authorization); product/scope/priority calls are his; tracker writes drafted first and executed only owner-present (files kept in the ticket's own topic directory).
5. **Evidence bar** — every claim in a hand-off carries file:line + quote, or command output.
6. **Exactly one verifier per artifact** — assigned, never self-started in parallel.
7. **No silent scope growth** — adjacent findings become a numbered disposition list for the owner, never riders.
8. **Reachability question on any money-path branch** — "who can now arrive here who couldn't before", answered by name.
9. **Mode visibility** — the hand-off OPENS by naming the mode served and what that excluded.
10. **Never substitute a cheaper mode for the ordered one.**

## B. Roles and seats

- **LEAD** (one session): architect, verifier of every claim it acts on, tracker, sole owner-facing
  voice. Never writes production code — regardless of how small the edit looks; a lead who edits is
  also the reviewer of its own lines, and that seam is where the pipeline's guarantees leak.
- **Seat models:** builders on the workhorse tier (tightly-specced legwork; a builder never designs);
  gate/verify JUDGMENT seats on the strong tier; verification LEGWORK seats on the workhorse tier.
  ⛔ Never spawn a lead-tier seat; never let a seat spawn seats.
- **Freshness:** reusable seats for legwork across rounds of one ticket; the gate reviewer is ALWAYS
  fresh. Fresh seats get full context in the spawn prompt — they inherit nothing.
- **Blind vs informed:** gates default blind to the builder's claims with full task context; every
  dispatch STATES its mode. Unblinding is the owner's knob.
- **Verification seats are refute-framed** — every claim goes in as "prove this false", verdicts
  require file:line evidence in hand, the seat sees no inherited narrative, and its order carries
  the standing question "what does this NOT mention that changes the risk?" A confirmer reads until
  it finds support; a refuter reads until it finds the counterexample.
- **Liveness:** a silent seat gets nudged immediately with a forced-status ask — never waited on,
  never duplicated in parallel.
- **Seats report to disk; the message channel carries a pointer.**
- ⛔ **A lead's builds run in the LEAD'S OWN working clone.** All other clones are read-only reference.

## C. The phase spine (modes select from this; order is fixed)

0. **BRAINSTORM** — before any plan or design: context first, one question at a time, two-to-three
   approaches with a recommendation, sectioned design to the owner for approval. Scaled to the task,
   never skipped. Problems get systematic debugging before fixes where it makes sense.
1. **TRUTH — the ground-truth walk** — no plan, and no endorsement of one, without a walk of the
   plan's GROUND TRUTH: source for a code change, live infra state for an ops change, current
   provider docs for an integration. Claimed facts are evaluated against requirements (and
   requirements against facts) — never against a narrative. Inherited claims — from tickets, earlier
   reviews, room discussion — are hypotheses until traced; they are precisely the ones that skip
   verification by feeling already-known. The walk runs via a refute-framed seat (§B) — a lead
   reviewing a plan's internal logic without the walk is a second author agreeing. Empirical beats
   read when cheap: build the repro when a source-read verdict decides real money or architecture.
2. **SPEC** (lead-authored) — anchored to current main (file:line); mandatory CASE MATRIX
   (entity × state × path, no-ops written out) scaled to mode; the reachability question per touched
   branch; OUT-OF-SCOPE list where every untestable or parked finding carries its ARMING TRIGGER;
   escalation rules written in. Sweep/campaign scope is an INVOCATION PARAMETER the owner states per
   run — the method defines how scope is computed and proven; the owner defines what it is.
3. **CHECK → BUILD** (same seat by default; XL orders a fresh check seat) — the seat adversarially
   spec-checks against source FIRST, report to disk; builds only on its own PASS. FIRST-RED per
   mode: the first test fails for the right reason before any production code, else the model of
   main is wrong and the line stops.
4. **GATE** — see the ladder. Standing tempo: the gate runs BEFORE the tree is offered for the
   owner's push; rush-mode (gating a pushed head) returns only on his explicit word. Findings
   classified NEW / KNOWN-TRACKED / ALREADY-RULED against the register before they reach the owner.
   Full coverage on sweep-class work — counts against totals, never sampling presented as complete.
5. **FOLD** — same builder folds accepted findings; the gate re-checks the delta; findings that
   contradict a registered decision go to the owner BEFORE any fix. ⛔ **Fix rounds use the
   reviewer's prescribed wording** — never fresh phrasing in the flagged spot; that is how new false
   claims get minted. Restoring lost text: verbatim, or the commit says "condensed".
6. **VERIFY** — one independent verification of the tree AS IT SITS ON DISK (re-build + full
   suites), by a party with no stake in the build's claims.
7. **HAND-OFF** — report at the ordering channel: mode + exclusions first, then PROBLEMS (by
   weight), then GOOD, then asks-on-owner last, never interleaved. Tree uncommitted, fingerprint
   stated. After the owner pushes: verify the pushed head byte-matches the verified tree at origin
   before anything else proceeds.

## D. Owner interaction

- **Go-word per artifact.** Rulings, urgency, answered questions set design and priority — never GO.
- **Owner-declared states (pause/hold/freeze) lift only on his word** — asking permitted, inferring never.
- **Questions carry a default** so silence has a defined meaning off-hours; genuinely blocked →
  poke immediately, zero stale waiting.
- **Judgment calls below the gate: decide and state; bring SHAPED decisions** (recommendation + the
  one fact that could flip it) for anything at the gate. A new owner ruling landing on staged work:
  the retroactivity call is HIS — a two-option question, never decided quietly.
- **Ruling provenance:** every recorded ruling carries its message id. Owner wordings are
  PRINCIPLES, not literals — internalize what the order means generically.
- **Sequencing:** the lead AUTHORS priority order with stated logic so the owner re-rules rather
  than originates; his ruled order supersedes and is recorded as such.
- **Outward-facing text is an owner-level call** — anything served beyond the repo changes what
  outsiders see; keep-by-default and ask.

## E. The record

- **Tracker:** append-only; stale items moved to a marked section with a one-word reason, never
  deleted. TICKET text is the opposite register: current-truth-only, formal, humanized.
- **Decision register:** kept in the ticket's topic directory; append-only, numbered + message ids;
  TWO-SIDED compliance — specs checked against it before build, gates diff the change against it
  after.
- **Open-questions file:** standing walk-list for owner sessions; PAUSED-PENDING carries exact
  resume steps.
- **Context-summaries:** self-contained; every "we did X" claim needs a record lookup in hand; forged
  before any non-trivial compaction.
- **Deploy notes:** a standing per-ticket section — read-path vs stored-row behaviour at deploy,
  per-environment ordering gates, rebuild decisions — stated before anyone pushes; attaches to the
  ticket like the spec.

## F. Comms

- Quiet rooms: brief single items; no unprompted posts; linked mentions; every message names its actor.
- Long-form to disk + pointer in the room. Reports split PROBLEMS/GOOD/asks. Superseded direct
  messages get a "superseded — current state" append.
- Ordered work ends with findings POSTED at the ordering channel — desk artifacts are not reporting.
- **Nothing outward carries agent names, internal-team references, phase codes or finding letters** —
  and nothing said TO the owner does either: plain words, no shorthand a human must decode.

## G. Paid-for failure modes (self-check list)

1. A receipt/delta is never a review.
2. Presence before gated writes.
3. Wake = read + relaunch in one batch.
4. Per-arm mutation for multi-arm dispatches; a pin that cannot fail is not a pin — and a fixture
   whose derived and stored values coincide cannot pin the read.
5. Consensus is not verification — one source read outranks a room's agreement.
6. Checker's arithmetic re-verified before acted on.
7. The gate reviews what the diff doesn't show.
8. "Do the same" means THE SAME — no instrument substitution.
9. A lying comment is a real defect — it outlives wrong code.
10. A check whose passing output is indistinguishable from a broken instrument's is not evidence.
11. A green exit code is not a run — count the per-suite result lines, never the exit code.
12. A plan's inherited premise is the claim most likely to be false — it entered without a walk.
13. A correction applied where you were looking, while the claim stands elsewhere — run the sweep's
    own inverse check on your own artifacts.
14. Full coverage and independence are DIFFERENT controls; each catches what the other misses.

---

## LADDER — modes, sizes, gate cadence

### §0 How to order

The owner orders a **delivery size**: S · M · L · XL. **Default, when no size is ordered: L.**
(Distinct from the review-size ladder used for code review — same letters, different meanings.)
Hurry is handled by ordering a smaller size — never by quietly thinning a bigger one.

1. **Orders are exact** — ambiguous → ask.
2. ⛔ **Never substitute a cheaper mode for the ordered one.**
3. ⭐ **Every hand-off OPENS by naming the mode served and what that excluded.**
4. **Sizes go up, never sideways** — work outgrowing its ordered mode mid-build is a defined
   escalation event: stop, say so, get the re-order.

### §1 The sizes

| Mode | For | Adds over the rung below | Test discipline |
|---|---|---|---|
| **S — patch** | trivial mechanical change | anchored spec-note (one paragraph) → ONE builder seat (lead never codes, even here) → full suite → author diff self-read → lead verification | tests-alongside where behaviour is touched; a genuinely inert change says so instead of carrying a theatre test |
| **M — quick fix** | small behavioural fix under real time pressure | + same-seat spec-check→build + full local gate (the fresh gate doubles as M's independent verifier — stated in the hand-off) | FIRST-RED optional; **per-change PIN-PROOF mandatory** — every fix carries a test demonstrated to fail without it (red-first or prove-by-revert; evidence quoted) |
| **L — standard (DEFAULT)** | normal ticket work | + written spec (anchored, every ⛔ with its reason) + adversarial spec-check + FIRST-RED stop clause + fresh-seat full local gate + independent final verification (suite re-run on the tree as it sits, fingerprinted) | full TDD; per-branch reachability answered in the hand-off |
| **XL — campaign** | multi-defect or money-path campaigns, night shifts, anything the owner calls big | + up-front investigation seat(s) + fresh-seat spec-check + mutation-grade pin proofs (fail-direction demonstrated, positive control) + blind review round(s) + fresh final verifier + register-compliance check + deploy-note section | L plus proven-fail pins and blind convergence as the close signal |

**Off the ladder — hotfix.** Production is bleeding: the owner may order build-before-spec; the spec
is written AFTER as the record; the never-cut list still holds whole. Its hand-off opens "HOTFIX
SERVED" — nothing else may masquerade as it.

### §2 The gate cadence

1. **First gate: FULL and FRESH-BLIND** — no inherited narrative, refute-framed, full coverage, its
   own suite run, BEFORE the tree is offered for the owner's push. A lead's own review is not the gate.
2. **Every change after that gate gets a DIFF GATE** on exactly that delta before the next hand-off.
3. **Before merge, once diff gates pass: a FINAL FULL GATE** on the finished head — never skipped
   for small deltas; "too small to gate" is the argument every skipped gate is made with. Exemptions
   are RULED, never decided quietly; retroactivity of a new cadence ruling on staged work is the
   owner's call.
4. **Sweep-class gates run at FULL COVERAGE** — coverage numbers equal the totals; sampling
   presented as completeness is a gate defect.

Standing hand-off artifacts: the ticket's spec attachment swapped current before review rounds (and
as-shipped at merge) · the deploy note attached to the ticket the same way · suggested commit
messages ONE line, conventional subject, no body, no attribution trailers, paste-ready.

### §3 The ingredients (capped at ten — new instruments go inside an existing body)

| Ingredient | What it is | In modes |
|---|---|---|
| **investigation** | read-only seat anchoring claims at source BEFORE the spec cites them | XL (L when the spec needs anchors the lead doesn't hold) |
| **spec** | single source of truth: file:line anchored, every ⛔ with its reason, out-of-scope wall, escalation rules, checkable ACs | M (mini) · L · XL |
| **spec-check** | adversarial independent re-verification of every anchor and number; FAIL stops the line before code | M · L · XL |
| **FIRST-RED** | first test written alone, must FAIL for the right reason before any production edit; passing = wrong model of main → stop | L · XL (optional M) |
| **pin-proof** | every fix carries a test demonstrated to fail without it; mutation-grade with positive control at XL | M · L · XL |
| **local gate** | fresh seat, full local verification of the built tree; report to disk; cadence per §2 | M · L · XL |
| **verifier** | independent final check of the tree ON DISK: rebuild, full suite, counts matched, fingerprint | all modes (knob: lead-run vs fresh seat) |
| **reachability** | per touched branch: "who can now arrive here who couldn't before", by name | money-path: EVERY mode · full sweep: L · XL |
| **doc-sweep** | comments-as-claims pass over the touched surface (comment-volume + internal-references method); scope an invocation parameter | L · XL (S/M when the change is documentation) |
| **deploy-note** | what deploy does and doesn't fix at read-time vs stored rows; ordering gates; attaches to the ticket | XL + any mode altering deploy behaviour |

### §4 What no mode may cut

The canonical list is §A. The ladder's rule: **every rung states which spine guarantees it carries
and HOW.** A mode that cannot state how it honours one of them is not a mode — it's a corner being
cut with paperwork.

---

## Companion methods (kept separately in our own vault, summarised here so this note stands alone)

- **doc-sweep** — the comment-volume + internal-references method: treat comments as claims and
  sweep the touched surface for ones the change has made false.
- **code-review menu** and **review-round playbook** — the review instruments and the flow of a
  review round. Note the review-size ladder uses the same S/M/L/XL letters with different meanings
  from the delivery ladder above; don't cross the two.
- A per-company review rubric sits alongside these as the default review instrument.
