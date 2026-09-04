---
type: reference
tags:
  - skills
  - fleet
  - router
  - which-skills
maintainer: Bodhi
updated: 2026-09-04
plugin_versions: mattpocock-skills@1.2.3
---
# which-skills — situation → skill router (fleet)

**Recall trigger:** "Recall which-skills note" → load this and route the owner to the right skill for the active topic.

**What this is:** a cross-plugin decision aid over the two installed skill sets — **`mattpocock-skills`** (Matt Pocock's agentic-dev plugin, v1.2.3) and **`superpowers`**. Maintained by Bodhi; update when either plugin changes.

## Two native routers already exist — know them first
- **`/ask-matt`** — *mattpocock's own router* ("ask which skill or flow fits your situation"). **User-invoked** (`disable-model-invocation: true`). It maps mattpocock's full idea→ship flow. When the owner asks "which mattpocock skill/flow do I use for X?", the honest answer is often just **run `/ask-matt`** — it's the source of truth for that plugin's routing.
- **`superpowers:using-superpowers`** — superpowers' entry discipline for finding/using its skills.

**This note's unique value** is the layer neither router covers: **mattpocock ↔ superpowers overlaps + fleet defaults**, so we never run two conflicting skills for one job.

## mattpocock flow (from the `/ask-matt` map)
Only a subset auto-triggers; most are user-invoked (`/name`). The full shape:
- **Main flow (idea → ship):** `/grill-with-docs` (stateful interview, writes CONTEXT.md/ADRs) → *(optional `/prototype` detour, bridged by `/handoff`)* → `/to-spec` → `/to-tickets` → `/implement` (drives `/tdd` internally, then `/code-review`). Single-session build skips spec/tickets and goes straight to `/implement`.
- **On-ramps:** `/triage` (incoming bugs/requests you didn't create) · `/diagnosing-bugs` (something broken — hard/intermittent/regression) · `/wayfinder` (a huge, foggy, multi-session effort → charts a map of decision tickets; produces *decisions, not deliverables*; hands off to `/to-spec`).
- **Codebase health:** `/improve-codebase-architecture` (surfaces deepening opportunities) → feeds `/grill-with-docs`.
- **Vocabulary layers:** `/domain-modeling` (domain terms, CONTEXT.md, ADRs) · `/codebase-design` (deep-module design vocabulary).
- **Standalone:** `/grill-me` (stateless grilling, no repo) · `/grilling` (the interview primitive) · `/prototype` · `/research` (background agent, cited markdown) · `/to-questionnaire` (interview someone else) · `/wizard` (human-only steps) · `/wait-what` (re-pitch a message that didn't land) · `/teach` · `/writing-for-agents` · `/resolving-merge-conflicts`.
- **Precondition:** `/setup-matt-pocock-skills` (run once before first flow).

## superpowers set
`using-superpowers` · `brainstorming` · `writing-plans` · `executing-plans` · `subagent-driven-development` · `dispatching-parallel-agents` · `using-git-worktrees` · `test-driven-development` · `systematic-debugging` · `verification-before-completion` · `requesting-code-review` · `receiving-code-review` · `finishing-a-development-branch` · `writing-skills`.

## Overlaps — pick ONE (fleet defaults)
- **TDD:** `superpowers:test-driven-development` ⟷ `mattpocock:tdd` (the latter runs *inside* `/implement`). **Default: superpowers** standalone; inside a mattpocock build flow, `/implement` will drive mattpocock's tdd — that's fine, don't stack them.
- **Debugging:** `superpowers:systematic-debugging` ⟷ `mattpocock:diagnosing-bugs`. **Default: superpowers**, unless you're already in the mattpocock flow.
- **Idea shaping:** `superpowers:brainstorming` (generative, open-ended) vs `mattpocock:grilling`/`grill-with-docs` (adversarial stress-test of a formed idea). Use brainstorming to *generate*, grilling to *harden*.
- **Planning:** `superpowers:writing-plans` ⟷ `mattpocock:to-spec`+`to-tickets`. Prefer mattpocock's spec→tickets when you'll run a multi-session build via `/implement`; superpowers writing-plans for a lighter plan.
- **Code review:** `mattpocock:code-review` *does* the review (Standards + Spec, parallel); `superpowers:requesting-code-review`/`receiving-code-review` are the request/respond workflow. **Complementary.**
- **Skill/agent-doc authoring:** `mattpocock:writing-for-agents` (+ AGENTS.md/CLAUDE.md) ⟷ `superpowers:writing-skills`. **Complementary.**

## Quick situation → skill
| Active topic | Use |
|---|---|
| "Which skill for this?" (mattpocock) | `/ask-matt` |
| Explore/generate an idea | `superpowers:brainstorming` |
| Harden a formed plan/idea | `mattpocock:grill-with-docs` (repo) / `grill-me` (no repo) |
| Multi-session build | `mattpocock:/to-spec → /to-tickets → /implement` |
| Light plan then build here | `superpowers:writing-plans` |
| Huge foggy effort (many sessions) | `mattpocock:/wayfinder` |
| Something broken | `superpowers:systematic-debugging` (default) |
| Test-first a concrete behaviour | `superpowers:test-driven-development` (default) |
| Throwaway prototype | `mattpocock:/prototype` |
| Research vs primary sources | `mattpocock:/research` (repo note) / `deep-research` (cited report) |
| Review a branch/PR | `mattpocock:/code-review` |
| Verify before "done" | `superpowers:verification-before-completion` |
| Merge/rebase conflict | `mattpocock:/resolving-merge-conflicts` |
| Parallel independent tasks | `superpowers:dispatching-parallel-agents` |
| Isolated workspace | `superpowers:using-git-worktrees` |
| Human-only setup (creds/dashboards) | `mattpocock:/wizard` |
| Incoming bugs/requests to sort | `mattpocock:/triage` |
| A message didn't land | `mattpocock:/wait-what` |
| Write/edit a skill or AGENTS.md/CLAUDE.md | `mattpocock:/writing-for-agents` (+ `superpowers:writing-skills`) |

## Maintenance
Bodhi owns this. `/ask-matt` is the authoritative router for the mattpocock set — defer to it for that plugin and keep this note focused on the cross-plugin overlaps + fleet defaults. Re-check on plugin updates (current: mattpocock-skills@1.2.3). Verify a named skill exists before recommending it. Note: only a subset of mattpocock skills auto-trigger; the rest are user-invoked (`/name`).
