---
name: feedback-capture-and-reuse-lessons
description: Ankur wants me to capture durable technical lessons from bodhi/infra sessions and apply them next time
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 67e2900b-61a8-4497-b778-32ca1a94b620
---

On 2026-06-29 Ankur (owner) explicitly told bodhi: *"review and keep all this in your memory — we need to learn and implement the things like above."*

**Why:** he treats these infra/runbook sessions as a knowledge-building loop, not one-offs. Corrections he gives (e.g. the rebuild-runbook fixes) and patterns surfaced during a task (e.g. `$all` → `$ce-` category-stream narrowing) should persist and be reused, so the same mistake isn't repeated and the same fix is applied proactively next time.

**How to apply:** after a substantive bodhi/Pistacia infra task, write the durable, non-obvious lessons to memory (a `reference` for the technical pattern, update the relevant runbook memory, a `feedback` for how-to-work guidance). When a new task resembles a past one, lead with the remembered pattern instead of re-deriving it. Verify the remembered fact against current code/PRs before asserting it (memories are point-in-time). Examples captured this session: [[esdb-projection-category-stream-pattern]], [[pistacia-projection-rebuild-runbook]].
