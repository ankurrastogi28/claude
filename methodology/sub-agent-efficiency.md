---
created: '2026-06-14'
adopted: '2026-08-03'
tags:
  - methodology
  - sub-agents
  - efficiency
  - cost
  - workflow
aliases:
  - Sub-agent efficiency
  - When to use sub-agents
  - Sub-agent cost model
  - Delegation decision rules
type: methodology
description: whether/when to delegate + which model; spawn startup-tax cost; Sonnet default
---
> **Provenance:** fleet methodology (origin: Faruk's vault, grounded in a measured 10-run study), adopted into Ankur/Bodhi's brain 2026-08-03 on owner instruction. Companion to [[sub-agent-investigation]] (*how* to dispatch); this note covers *whether* to delegate, *which model*, *which structure*.

## ⭐ The operating rule (owner-set, 2026-08-02 — read this, the rest is the evidence)
Your MAIN loop is the expensive thing (frontier-tier weekly limits). So:
1. **Do as little as possible yourself.** Main loop = spec, delegate, verify, report. Everything else is a sub-agent.
2. **Name the model on EVERY spawn** — an unnamed spawn inherits your main-loop model and silently burns the expensive tier.
3. **Match model to task, fairly:** Haiku = bounded single-shot trivia · **Sonnet = the default workhorse** (owner: "Sonnet 5 is really successful nowadays, most of the time you might not need Opus at all") · Opus = only when Sonnet demonstrably struggles.
4. **Long runs → NAMED agents / agent teams** (addressable, context persists across sends) — re-briefing a fresh spawn re-pays the startup tax AND your own tokens.
5. **Don't delegate** trivia, small-context, or tightly-coupled work — the ~13K startup tax loses (table below).
6. **Verify a sub-agent's work by RUNNING it, never by reading its report** — and the verifier is a different mind than the builder.

**Why:** the headline finding — **a sub-agent's cost is CONTEXT, not output, and every spawn pays a fixed startup tax (~9–21K tokens) before it does any work.** Delegate only when the benefit (context preserved, independent perspective, parallel wall-clock) clears that tax.

*(Full study + data + run files live on Faruk's box: `Waggle/fleet/lab/sub-agent-usage-efficiency-study/` — SYNTHESIS.md, METHODS-CORRECTION.md, GUIDE-using-sub-agents.md. Fleet-origin, not on my workstation; kept for provenance.)*

## Decision rules (data-backed)
| Task signature | Delegate? | How | Model |
|---|---|---|---|
| Large input → small output (search big corpus, summarize long doc, scan logs) | **YES** | single sub-agent | Sonnet |
| Independent review / verification | **YES** | do-then-verify, **both cheap** | Sonnet do + Sonnet verify |
| Genuinely parallel, independent, **heavy** shards | maybe | parallel fan-out | mid |
| Small context (few-K) | **NO** | inline | — |
| Trivial / one-liner | **NO** | inline | — |
| Already-solved / easy task | **NO extra steps** | inline | — |
| Tightly-coupled, shared-state, sequential | **NO** (theory) | inline | — |

**Three "don't delegate" red flags:** triviality, small context, tight coupling.

## Cost model — measure the right tokens
- **Sub-agent cost ≈ startup tax + per-turn context re-reads. Output is <1% of cost** — never judge cost by output tokens (`budget.spent()` is output-only, hides ~99% of the bill). Parse the full sub-agent JSONL `usage` (4 keys); price `cache_creation ×1.25`, `cache_read ×0.10`.
- **Startup tax** ≈ 9–21K tokens/spawn (floor ~8.9K = system prompt + tool defs). The fixed cost of delegation, and why trivial/small-context delegation loses.
- **Break-even:** delegate context-heavy work only when the input absorbed **≫ the ~13K startup tax**. `CtxLeverage` (input kept out of the orchestrator) measured 0.95–0.99 for 8–32K docs, ~0 for tiny inputs.
- **Cost driver is metric-dependent:** on RAW tokens → "fewer spawns, fewer turns"; on PRICED tokens `cache_read` collapses ×0.1 so cache-warm delegations are cheap. Report both.

## Model fit (with a caveat)
- On tasks within every tier's competence, tiers separate on **cost, not quality** → pick the cheapest that runs *efficiently*.
- **Sonnet = sweet spot** for competent delegated work (cheapest, stable). **Opus** only for genuinely hard reasoning + orchestration/synthesis. **Haiku is a trap for open-ended work** — low per-token price erased by runaway turn-counts (measured 4× Sonnet's cost); fine only for truly bounded single-shot tasks.
- **Caveat:** the study's tasks were all solvable by all tiers → this is a **cost-tiebreak among competent tiers, not a hard-task model-fit answer** (the cheap-model quality cliff is untested).

## Strategy fit
- **Single sub-agent** = default winner (lowest spawn count).
- **Parallel fan-out** lost 1.9–3.6× on tokens with no quality gain on small inputs — only wins when per-shard work ≫ startup tax *and* wall-clock matters.
- **Do-then-verify** with cheap models beat a single strong model on **both** quality and cost for error-prone review; **the verifier's model doesn't matter — the independent structure does** (Opus verifier no better, sometimes worse, 36% costlier). Pure overhead on easy/already-correct tasks (+44% cost, zero gain).

## Applied pattern — continuous low-density triage (e.g. a watcher/Observer)
Continuous scanning of high-volume conversational text (cheap to read, rare to act on) is the **don't-delegate** case on three counts at once (small input, trivial-per-message, continuous = re-pays the tax every cycle). **Scan resident & inline; delegate only the occasional deep-dive** (a real concern → pull full history + corroborate, input ≫ tax) — one Sonnet, do-then-verify if it feeds a downstream decision. Per-room parallel watchers are a trap. If volume outgrows one context, shard by **time**, not per-room.

## Process lessons (meta)
- **Always add an independent critic pass** — in this very study an independent reviewer caught a load-bearing token-instrument error, and a builder caught an arithmetic bug in the spec. See [[sub-agent-investigation]] §4.
- **Output is low-variance, token cost is high-variance** across runs — repeat runs before trusting a cost number.
- **Critic loops bloat** (a 10× plan-improve loop grew a doc 11KB→50KB; critics add, rarely cut) — cap iterations or add a deletion-biased "trim" critic.

## Implications for waggle minions / teams
1. **Gate spawn on input-size / task-type** — don't spawn for trivial/small-context; the startup tax dominates.
2. **Prefer few long-lived minions over many short ones** — amortize the tax; an N-way fan-out pays it N times.
3. **Cap cheap-model turn/token budgets** — a Haiku minion can silently burn 4× a Sonnet one; default Sonnet.
4. **Make cheap do-then-verify first-class** — a do-minion + a separate cheap verify-minion beats one expensive minion on quality and cost.
5. **Decomposition width scales with per-shard work, not with available parallelism.**

## Related
- [[sub-agent-investigation]] (dispatch discipline), [[investigation-workspace-layout]] (file-based deliverables), [[feedback-separate-reviewer]], [[feedback-capture-and-reuse-lessons]].
