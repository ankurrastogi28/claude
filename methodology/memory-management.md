---
tags: [memory, methodology, brain, auto-memory, doctrine]
---

# Memory management (canonical model)

How I manage durable memory across the fleet. **One principle: the brain holds the content; auto-memory holds only pointers.** Single source of truth — when a rule changes, edit ONE place, not four.

## Layers (each owns one thing)
1. **Brain note** (`<category>/<slug>.md` in the vault — categorize by TYPE, see Categorization below; `methodology/` is ONLY for actual processes/methodologies, never the default dump) — the **canonical** content: full Why / How-to-apply / When-NOT / anti-patterns. Long-form. This is the real memory.
2. **Brain index** (`index.md` at the vault root — the fleet-standard name, Obsidian-MCP-managed; NOTE the auto-memory index is a *separate* `MEMORY.md` under `~/.claude/projects/<project>/memory/` — different layer) — one line per note: `[Title](path) — hook`. Unindexed = future-me won't find it.
3. **Auto-memory breadcrumb** (`~/.claude/projects/<project>/memory/<type>-<slug>.md`) — ≤5 lines: one paragraph stating the rule + a `[[brain link]]`. A POINTER, never a copy.
4. **Auto-memory `MEMORY.md`** — index only: `- [Title](file.md) — ≤150-char hook`, one line per entry, no content.

## Categorization (classify by TYPE — standing rule, Faruk 2026-07-20)
Every durable note is filed by **type** into a category directory — do NOT default everything into `methodology/`. Baseline categories: `checklists/` (review/ops checklists & rubrics), `runbooks/` (step-by-step operational procedures), `reference/` (external pointers, facts, config), `methodology/` (actual processes/methodologies ONLY), plus the auto-memory `type:` (`feedback`/`project`/`user`/`reference`). Keep BOTH the vault directory tree and the auto-memory structured by category so future-me finds things by kind. A new kind of note → pick (or create) the right category, not methodology-by-reflex.

## Saving a memory (5 steps)
1. Decide general vs CWD/project-specific.
2. Write the full brain note (`methodology/<slug>.md`).
3. Add a line to the brain index (`index.md`).
4. Write the auto-memory breadcrumb (paragraph + brain link).
5. Add a line to the project `MEMORY.md`.

## Memory types (auto-memory frontmatter `type:`)
- `user` — who the owner is
- `feedback` — how to work (always include the **Why**)
- `project` — ongoing work / goals
- `reference` — pointers to external resources

## Evolve / retire
- **Evolve:** edit the brain note ONLY.
- **Retire:** delete all four (brain note, brain index line, breadcrumb, project `MEMORY.md` line).

## Don'ts
- Don't duplicate brain content into auto-memory — breadcrumb ≠ mirror.
- Don't put content in the index files — they're indexes.
- Don't forget the brain index line — unindexed = invisible.
- Don't save what the repo already records (code structure, git history, CLAUDE.md) or conversation-only transient facts.
- Before saving, check for an existing file that already covers it — update it, don't create a duplicate.
- Write/edit vault files via the **Obsidian MCP** (`write_note` / `patch_note` / `move_note`), NOT raw filesystem `Write`/`Edit` or git.

## Access + sync
- Reach the brain via the **Obsidian MCP** (load the obsidian skill first; never raw git/grep on the vault).
- Sync with **`memo-sync`**: pull at session start, push at session end (not raw git in the vault).

## Source
Bucky's fleet memory-management primer (2026-07-14), delivered at Faruk's request. Bucky's canonical doc reference: `methodology/memory-management.md`.
