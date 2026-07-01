---
name: feedback-slack-mention-format
description: "In Slack posts, render user mentions with <@USERID> angle-bracket syntax, not @USERID"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 53bb78f7-2e94-4d5b-b843-4cb54da5e724
---

When posting to Slack (via `slack_bridge_reply`/`_post`/`file_upload` initial_comment), write user mentions as **`<@U03NK1M7G6R>`** (angle brackets) — that renders as a real, notifying @-mention. Writing bare **`@U03NK1M7G6R`** shows up as literal text, which is wrong.

**Why:** the architect (U08Q952LB1C) flagged 2026-07-01 that my mentions were rendering as raw `@U03NK1M7G6R @U08Q952LB1C`.
**How to apply:** always wrap Slack user ids as `<@ID>`; same for channels `<#CID>`. Applies to every Slack post Bodhi makes. (User ids for this thread: Ankur = U03NK1M7G6R, architect/Faruk = U08Q952LB1C, Bucky bot = U0AT6JCGXKK, me/Bodhi = U0BBTT302FR.)
