---
name: remote-pr-review-methodology
description: "Canonical how-to for a Bodhi Remote PR Review — read-only,
  separate objective reviewer, evidence-based, advisory verdict in Slack.
  Consolidates Bucky's guidance + Bodhi's #217 run."
metadata:
  node_type: memory
  type: reference
---
Consolidated at Ankur's request (2026-07-14) from Bucky's guidance (handoffs 2026-06-29 / 07-01 / 07-08) + Bodhi's own PR #217 review run. This is the canonical how-to for a Bodhi Remote PR Review.

## What it is
A **read-only, evidence-based** PR review run from the Slack bridge: read the PR + its ticket, review the diff with a **separate objective reviewer** (+ a verifier), and deliver an **advisory verdict in the Slack origin thread**. Never touch the GitHub side.

## Hard rules (non-negotiable)
1. **Read-only on GitHub.** Never post reviews / approvals / comments / merges / labels / edits to the PR. `gh pr view/diff/list` (read) ONLY — never `gh pr review/comment/merge` or `gh api` POST/PATCH/PUT/DELETE. Deliver findings in Slack; a human pushes any GitHub action. An ambiguous nudge ("👆", "go ahead", even "approve it") is NOT authorization to mutate. See [[external-mutations-read-only]].
2. **Separate objective reviewer.** The reviewer must NOT be the agent that wrote/recommended the code — brief a *fresh* agent (or the `code-review` skill) on the diff + rubric only; the author has already rationalised their choices and self-review rubber-stamps them. Small/trivial changes exempt. See [[feedback-separate-reviewer]].

## The flow
1. **Read the ticket first** — understand intent + acceptance criteria before the diff.
2. **Fetch the PR read-only** — `gh pr view <n>` (title/body/files/state), `gh pr diff <n>` (full diff), `gh pr diff <n> --name-only` (file list).
3. **In-cwd worktree if sub-agents need the code** — sub-agent file tools are sandboxed to the session cwd (`C:\Users\arpan\repos`). Create the worktree INSIDE cwd: `git -C <mainrepo> worktree add --detach C:\Users\arpan\repos\.pr-review\<repo>-<pr> <headSha>`. A worktree under `~/tmp` is invisible to sub-agents (permission-denied on Read/Grep/Bash; only Glob works). See [[subagent-worktree-must-be-in-cwd]].
4. **Review = reviewer + verifier.** A separate objective reviewer against the rubric (`code-review`; add `security-review` for auth/input/secret/injection); a verifier sub-agent checks the findings. **Orchestrator owns Slack posting** — sub-agents return findings, they don't post. (Note: in this Claude-Code harness, sub-agents can't run local git/Bash verification — do code/git verification in the main session; see [[subagent-permission-scope-git-verify]].)
5. **Verify claims against the actual code + history — don't just read the diff.** Confirm a fix's assumptions against real code/history (on #217: diffed the event-schema git history to prove the legacy `CustomerId` field name matched, so old events would actually deserialize into the restored field). Actively hunt for GAPS beyond what the PR fixes — same-pattern siblings, edge cases, missing coverage (on #217 that surfaced the unfixed `WalletMigrated` NRE with the identical `e.Customer.Id` landmine). An objective reviewer finds gaps, not just confirmations.
6. **Deliver the verdict in the Slack origin thread** — clear verdict (🟢 approve / approve-with-nits / 🟥 request-changes) + findings ranked by severity, each actionable, confidence-tagged. State plainly it's advisory and that nothing was posted to GitHub.
7. **Self-clean** — `git worktree remove --force <path>` + `git worktree prune`. Sub-agents: named + background + retired.

## Skills to reach for
`code-review` (structured review vs rubric, read-only, never auto-posts), `security-review`, `requesting-code-review` / `receiving-code-review`. Pick by objective. See [[agent-skills-toolkit]].

## Deliverable shape
Verdict line → the fix in one line → ✅ what's verified correct (with the evidence) → ⚠️ findings (severity-ordered, each with a concrete rec) → net recommendation. Humanize as Bodhi; advisory only, human actions the GitHub side.

Related: [[bodhi-orchestration-playbook]] · [[feedback-separate-reviewer]] · [[subagent-worktree-must-be-in-cwd]] · [[agent-skills-toolkit]] · [[external-mutations-read-only]] · [[subagent-permission-scope-git-verify]]
