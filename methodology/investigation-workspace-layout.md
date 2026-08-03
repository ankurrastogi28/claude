---
created: '2026-05-13'
adopted: '2026-08-03'
tags:
  - methodology
  - sub-agents
  - workspace
aliases:
  - Tmp investigation layout
  - Sub-agent file naming
type: methodology
description: "naming .tmp sub-agent dirs: BRIEF/FINDINGS per sub-topic under a topic folder"
---
> **Provenance:** fleet methodology (origin: Faruk's vault), adopted into Ankur/Bodhi's brain 2026-08-03 on owner instruction. The file-based-deliverable convention that [[sub-agent-investigation]] §1 requires.

Convention for organising investigation working files under your temp dir (my context: `./.tmp` inside a project, or `~/tmp` under home).

## Structure
```
<temp-dir>/<topic>/                # parent investigation dir, kebab-case topic name
  sub-<sub-topic>/                 # one dir per sub-agent task (or discrete sub-investigation)
    BRIEF.md                       # spawn contract / input — what the sub-agent was told
    FINDINGS.md                    # deliverable / output — what the sub-agent returned
    *                              # working files: queries.sql, raw.json, scratch.py, etc.
```

## Naming rules
1. **`<topic>`** — kebab-case umbrella name (e.g. `ghost-wallets`, `gocardless-422-stuck`, `buc450-scan`).
2. **`sub-<sub-topic>`** — prefix `sub-` is mandatory; makes the parent-vs-sub distinction visible at `ls` time. Sub-topic itself kebab-case (`sub-root-cause`, `sub-verify-creation-paths`, `sub-remediation-plan`).
3. **`BRIEF.md` and `FINDINGS.md`** — fixed names, paired 1:1. Brief = the input the main thread wrote; findings = the file the sub-agent produced.

## Numeric prefix — only when reading order is meaningful
Add a zero-padded sequence prefix (`sub-01-root-cause/`, `sub-02-verify-creation-paths/`) **only when reading order matters** — i.e. the next sub-topic depends on a prior one. Independent sub-topics get plain names; numbering them implies a sequence that doesn't exist.

## Why this matters
- Multiple sub-agents per investigation is the norm (root-cause → verifier → remediation plan).
- A single flat brief file gets overwritten or confused. A predictable dir-per-sub-task keeps every spawn auditable.
- `BRIEF.md` + `FINDINGS.md` lets the main thread re-read input contract and output side-by-side when assessing proof tightness.
- Working files (raw query output, scratch scripts) stay scoped to the sub-task that produced them — no orphans cluttering the parent dir.

## Related
- [[sub-agent-investigation]] — the spawn-contract discipline that produces these files.
- [[sub-agent-efficiency]] — whether to spawn in the first place.
