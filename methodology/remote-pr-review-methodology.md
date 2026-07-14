---
name: remote-pr-review-methodology
description: Canonical Remote PR Review methodology (Bucky's authoritative brain
  version, captured verbatim-faithful for Bodhi) — on-demand advisory 2nd-pass
  review from the GitHub Slack channel; read-only, separate reviewer + verifier,
  two report files, per-BC rubric.
metadata:
  node_type: memory
  type: reference
---
Authoritative source: **Bucky's brain** — `Projects/Parcelhero/methodology/{remote-pr-review, pr-review-rubric}.md` + `pr-review-bc/{Pachira,Pistacia,Aspen,Moss}.md`. Captured faithfully for Bodhi 2026-07-14 at Ankur's request (must reproduce exactly). Adapt paths to this box under the working dir (`C:\Users\arpan\repos\...`, = `~/repos`).

## What
On-demand **second-pass** code review of ParcelVision PRs, driven from the GitHub-notifications Slack channel (`C099UBDF4UF`). The owner @mentions the reviewer with PR links; the reviewer reviews each, verifies the findings, and posts a short humanized summary + the uploaded report file(s) on that PR's **"Pull request opened"** Slack thread. Verdicts are **advisory — a human merges** (never touch the GitHub side — see [[external-mutations-read-only]]).

## Roles (never review inline in the orchestrator)
- **Orchestrator** — reads the linked Linear ticket **FIRST**, briefs the reviewer, dispatches the reviewer sub-agent, runs verification, posts, and logs. Owns ALL Slack posting; never reviews inline.
- **Reviewer sub-agent** — adds a worktree, reviews code-first (+ superpowers `requesting-code-review`), writes the humanized report, returns a summary + verdict + findings (**each with file:line AND the evidence/proof**), then removes its own worktree. Never posts to Slack.
- **Verification sub-agent** (SEPARATE from the reviewer, and never the author — [[feedback-separate-reviewer]]) — runs superpowers `receiving-code-review` over the findings, **evidence-first**: verifies each claim against source rather than rubber-stamping. Orchestrator verifies itself only when a finding's evidence is weak/missing. Nothing is posted until findings are verified.

## Trigger
On-demand only — owner @mentions in `C099UBDF4UF` with the PR numbers. No auto-pickup cron.

## Workflow (7 steps)
1. **Resolve the PR set** — `gh pr view <n> --repo ParcelVision/<Repo> --json number,state,title,isDraft,mergeable,headRefOid,baseRefName,additions,deletions,changedFiles`. Review in the order requested.
2. **Read the linked Linear ticket FIRST** (+ parent/siblings) for acceptance criteria, required tests, and deploy-coupling; fold into the reviewer's brief. A `CONFLICTING` PR is still reviewed on its merits (log the conflict as "resolve before merge"). Advisory.
3. **Worktree (read-only).** One **FULL clone per repo** at `~/repos/parcelhero/pr-reviews/<Repo>/` (full clone — NOT `--filter=blob:none`, so a needed blob is never lazily missing). Per PR:
   - `git -C pr-reviews/<Repo> fetch origin '+refs/pull/<N>/head:refs/remotes/origin/pr/<N>'`
   - `git -C pr-reviews/<Repo> worktree prune` then `worktree add .tmp/pr<N> origin/pr/<N>`
   - Review inside `.tmp/pr<N>/` (`.tmp/` is git-ignored — throwaway scratch).
   - Diff vs base: `git diff origin/<baseRefName>...origin/pr/<N>`. Never commit/push.
   - When done, the reviewer removes its own worktree: `git worktree remove .tmp/pr<N> --force`. Self-cleaning → no cleanup cron, no merge-monitoring.
   - (Harness note: sub-agent file tools are sandboxed to the session cwd; `~/repos/parcelhero/pr-reviews/...` is inside cwd, so fine — never put the worktree under `~/tmp`. See [[subagent-worktree-must-be-in-cwd]]. In THIS Claude-Code harness sub-agents can't run git/Bash — do the worktree/git in the main session, [[subagent-permission-scope-git-verify]].)
4. **Review + verify.** Open the ACTUAL source — don't guess schemas/SQL/streams/keys from naming. Cover: correctness & domain faithfulness; risk (migrations, projection rebuilds, money/wallet, backward compat, silent-skip branches); tests; security/PII (hardcoded secret/webhook = blocking); maintainability (brief); merge-readiness. Every finding cites **file:line + evidence**. Verification sub-agent checks each against source before anything is posted.
5. **Write TWO report files per version** (from the same verified facts):
   - `<Repo>-<pr>-v<n>-for-humans.md` — full narrative, fully **humanized** (Bucky/Bodhi voice).
   - `<Repo>-<pr>-v<n>-for-agents.md` — dense, structured, file:line-rich, action-ordered, **humanizer-exempt** (source of truth for precise file:line facts).
   Reports live at `pr-reviews/reports/<repo>/<pr>/`; raw evidence under `.../<pr>/v<n>/`. If a review recommends a ticket change, also attach `<Repo>-<pr>-v<n>-<TICKET>-update.md`.
6. **Post to the PR's "Pull request opened" thread** — (a) a humanized summary led by a one-line header (verdict + what the PR does), and (b) **upload BOTH report files**. **@mention the PR owner** (map GitHub author → Slack handle). `slack_bridge_file_upload` with the summary as `initial_comment` posts both in one call.
7. **Log it** in `pr-reviews/journal/<date>.md`: PR, head SHA reviewed, report version, summary reply ts, uploaded file ids/permalinks. Track last-reviewed SHA so unchanged, un-prompted re-runs don't double-post.

## The rubric (what to check)
Generic code-quality/security is **delegated** to the environment's `code-review` + `security-review` skills — don't re-list it. On top, every review adds:

**Three cross-cutting dimensions (every review, every BC):**
1. **Conventions / standards fit** — match the nearest existing exemplar; don't invent a new shape (business/money/auth logic at the right altitude; a new *versioned* event, not a mutated persisted one; use the BC's safe accessor).
2. **Regression / side-effect scan** — what breaks OUTSIDE the diff: event/contract ripple to appliers + projections + other-BC consumers; HTTP status-code contract; read-model/projection replay-safety; per-environment config parity; money math (truncation/rounding/VAT/sign).
3. **Read the linked Linear ticket (MANDATORY)** — orchestrator does this first: acceptance criteria, required tests, deploy-coupling the code won't show ("ship behind flag X", "don't deploy in isolation", "depends on BC Y first"). Confirm the PR satisfies the criteria, not just "compiles". No ticket found = state it as a gap, not a pass.

**Per-BC checklist** — run the matching list: **Pachira** (billing), **Pistacia** (payments), **Aspen** (account/identity/auth), **Moss** (edge gateway). Full detail in `pr-review-bc/<BC>.md` — **Pistacia captured** at [[pr-review-bc-pistacia]] (payments/projection/wallet hotspots); Pachira/Aspen/Moss still to pull from Bucky.

## Severity / verdict model (advisory)
- **Blocking** — correctness/security/data-integrity defect to fix before merge.
- **Non-blocking (nit)** — style/naming/minor convention drift.
- **`deploy-blocker`** — code is correct but shipping it *as-is, alone* is unsafe (flag on, migration first, depends on BC Y). Usually comes from the ticket. **Before tagging a dependency as a deploy-blocker, check the base branch** — if the prerequisite is already on `origin/<base>`, the PR merely depends on already-shipped work (NOT a blocker): `git cat-file -e origin/<base>:<path>`.
- **Verdict:** `approve` / `approve-with-nits` / `request-changes` (needs ≥1 blocking **code** finding). A `deploy-blocker` does **NOT** by itself force `request-changes` — surface it as a prominent separate flag and let the human decide merge/deploy ordering.

## Guardrails
- Read-only on the repos: no commits, no pushes. Read-only on GitHub entirely — never post review/approve/comment/merge; a human makes the merge call ([[external-mutations-read-only]]).
- **Humanize every posted message and the -for-humans report** (my own voice) even when an agent will ingest it; the `-for-agents` file is the only humanizer-exempt artifact.
- Orchestrator owns all Slack posting and never reviews inline; sub-agents write the report + summary, they don't post.
- Factual, no speculation dressed as fact; every summary @mentions the PR owner; a human makes the merge call.

## Living document
Each real review teaches something (new hotspot, missed convention, false-positive pattern) — fold it back into the rubric + per-BC files so the next review is sharper.

Related: [[bodhi-orchestration-playbook]] · [[feedback-separate-reviewer]] · [[subagent-worktree-must-be-in-cwd]] · [[agent-skills-toolkit]] · [[external-mutations-read-only]] · [[subagent-permission-scope-git-verify]]

## Owner & environment (adopted for Ankur / Bodhi)
The process above is Bucky's canonical methodology; the owner/environment-specific bits are adopted for my context:
- **Owner = Ankur Rastogi (`U03NK1M7G6R`)** — he @mentions me in `C099UBDF4UF` with the PR(s) to review; he (or a human) makes the merge call. Faruk (`U08Q952LB1C`) is a peer/co-driver, NOT my owner — see [[slack-users-who-is-who]]. `@mention the PR owner` in summaries maps the GitHub author → the right Slack handle on Ankur's team.
- **Voice = Bodhi (my own).** Humanize every Slack post AND the `-for-humans` report as *myself*, not Bucky. The `-for-agents` report is the only humanizer-exempt artifact.
- **Paths on THIS box (Windows):** full clone per repo at `C:\Users\arpan\repos\parcelhero\pr-reviews\<Repo>\` (= `~/repos/parcelhero/pr-reviews/<Repo>/`), `.tmp\pr<N>` worktree, reports under `pr-reviews\reports\<repo>\<pr>\`, journal under `pr-reviews\journal\<date>.md`. Keep everything under the working dir (`C:\Users\arpan\repos`) so dispatched sub-agents can read it ([[subagent-worktree-must-be-in-cwd]]).
- **Per-BC checklists** (`pr-review-bc/<BC>.md`) live in Bucky's brain — pull each from Bucky and store in my vault as I get them (Pistacia first, since #217 lives there).
