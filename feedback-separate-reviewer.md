---
name: feedback-separate-reviewer
description: "Don't review code with the same agent that wrote/recommended it — use a separate objective code-review agent"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 67e2900b-61a8-4497-b778-32ca1a94b620
---

Principle from Faruk (U08Q952LB1C) + Bucky, endorsed by Ankur 2026-06-29: **the reviewer must not be the agent that wrote (or recommended) the code, except for small changes.** Use a *separate, objective* agent briefed only on the diff, running the `code-review` skill with its ruleset — "a fresh agent briefed only on the diff catches what the author already talked themselves past."

**Why:** the author has already rationalised their choices; self-review rubber-stamps them. An independent reviewer with no stake catches what the author can't see.

**How to apply:**
- After producing/recommending a code change, route the review through a *fresh* sub-agent (or the `code-review`/`security-review` skill) with no knowledge of my reasoning — give it only the diff + rubric. Don't have the authoring context "confirm" itself.
- Pick the skill that matches the objective (review vs investigate vs plan vs implement) — see [[agent-skills-toolkit]].
- Small/trivial changes are exempt.

**My own slip to learn from:** earlier this session I "verified" PR #215 by diffing it against *my own* recommended fix — that's the author checking their own work, exactly the anti-pattern. It was a 3-line change (borderline-exempt), but the right move was an independent `code-review` agent. Bucky correctly kicked off an independent Remote PR Review of #215. Going forward: separate objective reviewer by default. Related: [[feedback-capture-and-reuse-lessons]], [[esdb-projection-category-stream-pattern]].
