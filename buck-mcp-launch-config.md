---
name: buck-mcp-launch-config
description: How to register/launch the buck-mcp (Bucky/bodhi) MCP server on this Windows machine
metadata: 
  node_type: memory
  type: project
  originSessionId: be648f27-f21f-4e86-872f-6280cc5c6900
---

The Bucky MCP server (project at `C:\Users\arpan\repos\ph\buck-mcp`) was renamed from `bucky` to `buck-mcp` (v0.16.0+). Three Windows-specific gotchas when registering it in `.mcp.json`:

1. **Entry file is `src/buck-mcp.ts`** (NOT the old `src/bucky.ts`, which no longer exists — stale CLAUDE.md/README still reference `bucky.ts`). Wrong entry → server spawns then exits "module not found" → client reports `-32000`.
2. **Command must be the absolute `bun.exe` path** `C:/Users/arpan/.bun/bin/bun.exe` — Claude Code spawns MCP servers without a shell, so bare `"bun"` won't resolve on Windows.
3. **`BUCK_MCP_HOME` must be set to `C:/Users/arpan/.bucky`** — the renamed server defaults its home/settings dir to `~/.buck-mcp`, but the bodhi Slack-bridge config (`slackBridge` block) lives in the old `~/.bucky/settings.json`. Without the override it reports "Slack bridge is not configured."

**The active registration is the TOP-LEVEL (user-scoped) `mcpServers.bucky` block in `C:/Users/arpan/.claude.json`** — NOT a project `.mcp.json` (there is none) and NOT the per-project `projects[...].mcpServers` copies (those are stale/unused). As of 2026-06-23 that block was missing both the absolute bun path and the env; the working form is:
```json
"bucky": { "command": "C:/Users/arpan/.bun/bin/bun.exe",
  "args": ["C:/Users/arpan/repos/ph/buck-mcp/src/buck-mcp.ts"],
  "env": { "BUCK_MCP_HOME": "C:/Users/arpan/.bucky" } }
```
After editing `.claude.json` you MUST reconnect bucky (`/mcp` reconnect, or restart session) — config changes only take effect on relaunch; a fresh `start_session` creation time confirms the new process.

**"bodhi" = connect/run the owner's personal Slack bot** (buck-mcp slack-bridge component). Runbook when the user says "bodhi" / "start bodhi":
1. `start_session { session_id }` (re-run after any reconnect — server restart drops the session).
2. `slack_bridge_status` to confirm configured (the `.bucky/settings.json` `slackBridge` block must have `enabled:true` + `ownerSlackId`/`botToken`/`appToken`/`ownerDmChannel`). "Slack bridge is not configured" = server launched without `BUCK_MCP_HOME` → fix registration + reconnect.
3. `slack_bridge_start` **with NO session_id** (omitting it makes the read-only session-latch a no-op, keeping the main session free for mutating Commands).
4. Watch loop (Option A): run **`scripts/watch-slack.ts`** (the old `watch-inbox.ts` no longer exists) as a background task: `"C:/Users/arpan/.bun/bin/bun.exe" "C:/Users/arpan/repos/ph/buck-mcp/scripts/watch-slack.ts" --timeout 3600` with `run_in_background:true`. It exits 0 the moment a message arrives — it does NOT keep running.
5. On each trip: read `slack_bridge_inbox {exclude_in_flight:true}` → `slack_bridge_claim` → `slack_bridge_reply` (addressed to bot) or `slack_bridge_dismiss` (not) → **relaunch the watcher** (always after clearing/claiming, else it trips again immediately). Empty/greeting DM mentions to the bot warrant a short friendly reply, not dismiss.
