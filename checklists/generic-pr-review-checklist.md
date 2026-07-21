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
- **Lesson:** on #219 I read the `docs/superpowers/specs/` files as legitimate spec evidence across v3/v4 instead of flagging the whole tree as committed scratch — Faruk caught it. Fix = `git rm -r` + gitignore `docs/superpowers/` (ideally global gitignore).
