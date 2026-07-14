---
name: subagent-worktree-must-be-in-cwd
description: Sub-agent file tools are sandboxed to the session cwd — PR-review
  worktrees must live inside the working dir, not ~/tmp
metadata:
  node_type: memory
  type: feedback
---

When running the Bodhi remote-PR-review flow, dispatched sub-agents' file tools (Read / Grep / Bash / PowerShell) are **sandboxed to the session's working directory** (`C:\Users\arpan\repos`). A git worktree created outside it — e.g. under `C:\Users\arpan\tmp\pr-review\...` — is **invisible** to sub-agents: they get a hard permission-denied on Read/Grep/Bash and only `Glob` (path listing, no content) works. The orchestrator (main loop) can still Read the tmp path fine, which masks the problem until an agent reports back "blocked, cannot access the code."

**Why:** the harness scopes sub-agent tool access to cwd as a sandbox boundary; it is not a broken toolset and should not be worked around by copying files out.

**How to apply:** always create the review worktree **inside** the working dir, e.g. `C:\Users\arpan\repos\.pr-review\<repo>-<pr>`, then `git -C <mainrepo> worktree add --detach <path-in-cwd> <headSha>`. Give sub-agents that in-cwd path. Self-clean with `git worktree remove --force` + `worktree prune` after posting. Complements [[bodhi-orchestration-playbook]] (self-cleaning worktrees) and [[buck-mcp-launch-config]].
