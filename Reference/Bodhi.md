---
tags:
  - bodhi
  - slack
  - buck-mcp
  - parcelhero
  - reference
aliases:
  - Bodhi
  - Bodhi bot
---
# Bodhi

**Bodhi** is the user's personal Slack bot, wired into the **Buck-MCP (bucky)** Slack bridge. It is the user's own equivalent of the "bucky" Slack bot — a separate Slack app (separate tokens) that the user created so they can DM/@mention it in Slack and have a Claude/Buck-MCP agent answer questions about Parcelhero's platform infrastructure (Postgres, Kubernetes, EventStoreDB, RabbitMQ, Redis, Azure DevOps, configs, etc.).

## Key facts
- **Type:** Slack app / bot backed by the Buck-MCP `slackBridge` feature.
- **Config location:** `slackBridge` block in `$BUCK_MCP_HOME/settings.json` (`appToken`, `botToken`, `ownerSlackId`, `ownerDmChannel`). Buck-MCP repo at `C:\Users\arpan\repos\ph\buck-mcp`.
- **Owner Slack ID:** `U03NK1M7G6R`. Owner DM channel: `D0BBTUXCATZ`. Bot user ID: `U0BBTT302FR`.
- **Mode:** runs **read-only** by default — answers questions; infra-mutating commands stay gated behind owner approval.

## How to "activate Bodhi"
1. Start a Buck-MCP session: `mcp__bucky__start_session`.
2. Start the bridge: `mcp__bucky__slack_bridge_start`.
3. Run the inbox watcher in the background: `bun scripts/watch-slack.ts --timeout 3600` (from the buck-mcp repo). Relaunch after every wake — it is one-shot.
4. On each new inbox message, dispatch an investigator that calls `slack_bridge_claim` → answers with Buck-MCP tools → `slack_bridge_reply`.

## Meaning of the name
"Bodhi" (Sanskrit) = awakening/enlightenment — fitting for an "is it awake?" assistant bot.
