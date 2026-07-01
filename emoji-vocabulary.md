---
name: emoji-vocabulary
description: "Shared emoji vocabulary (3 visually-distinct families) for CLI output — circles=confidence, squares=status, pictograms=callouts"
metadata: 
  node_type: memory
  type: methodology
  originSessionId: 53bb78f7-2e94-4d5b-b843-4cb54da5e724
---

Shared emoji vocabulary from Bucky (2026-07-01) so agents read the same. Three visually-distinct SHAPE families so they never blur even at the same colour (shape carries meaning; same colour across families is fine — 🟢 verified ≠ 🟩 done). **For CLI output** (the "+ waggle" in the original doesn't apply to me — just my CLI). Slack is kept lighter.

**Confidence — CIRCLES** (on a claim, how sure): 🟢 verified firsthand (with source) · 🟡 secondhand/reported · ⚪ assumption/unverified

**Status — SQUARES** (on an item, where it stands): 🟩 done · 🟦 active/in-progress · 🟧 parked · 🟥 waiting on owner · ⬜ not started

**Callouts — PICTOGRAMS** (only when something needs flagging): ⚠️ warning/risk · ❌ error/failed · ℹ️ info · 💡 suggestion · ⛔ blocker (external — distinct from 🟥 waiting-on-owner)

**Rules:** confidence + status can appear routinely; a callout goes ONLY where there's something to flag. They clarify, don't ornament — don't over-decorate.
