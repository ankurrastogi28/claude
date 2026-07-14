---
tags: [buck-mcp, sql, postgres, operating-rule, convention]
---

# buck SQL query rules (always LIMIT; no un-approved queries)

Durable operating rule for any Postgres work through **buck-mcp** (`pg_query`), across ALL projects — not just wallet monitoring.

## Rules
1. **Always include a `LIMIT`** on every SQL query, no exceptions — even trivial ones. Unbounded queries get gated/blocked.
2. **Never fire novel / ad-hoc queries** (e.g. `SELECT 1`) through buck. Any query not on the pre-approved list trips buck's **owner-approval gate** and will **time out** unless Ankur (the owner) approves it out-of-band. Leaving these pending needlessly pesters the owner.
3. For **connectivity probes**, use OS-level checks instead of SQL — e.g. `Test-NetConnection <pg-host> -Port 5432` (TCP), `Resolve-DnsName`, adapter/VPN status. These don't touch the approval gate.

## Source
Faruk (peer agent-runner, U08Q952LB1C) — enforced 2026-07-14 (and earlier, ref x58d). He explicitly asked that this be a **long-term** rule for all DB work, not scoped to the current monitoring task.

## Related
- Wallet pollution query (the pre-approved, `LIMIT 20`-bounded read) lives in the Pistacia projection-rebuild runbook / wallet-pollution-check notes.
- Approval model: buck gates commands (and un-recognized reads) for owner approval; standard bounded reads are auto-approved.
