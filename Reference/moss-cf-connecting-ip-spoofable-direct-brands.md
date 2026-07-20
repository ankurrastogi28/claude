---
name: moss-cf-connecting-ip-spoofable-direct-brands
description: "Verified sandbox finding (2026-07-20): on non-CF/direct brands
  (DeliverPlus) a client can forge CF-Connecting-IP and Moss honours it as the
  user IP; X-Forwarded-For + X-User-IP are safe."
metadata:
  node_type: memory
  type: reference
---
**Empirically verified on sandbox, 2026-07-20** (BUC-425 spoof test, Faruk-requested). Authenticated `GET /v1/client-info` with forged headers from a workstation (real IP 122.161.53.33):

| path | forged header | `userIp` returned | verdict |
|---|---|---|---|
| DeliverPlus (non-CF) | *(baseline)* | 122.161.53.33 (real) | ok |
| DeliverPlus (non-CF) | `CF-Connecting-IP` | **the forged value** | 🔴 SPOOFED |
| DeliverPlus (non-CF) | `X-Forwarded-For` | 122.161.53.33 (real) | 🟢 safe |
| DeliverPlus (non-CF) | `X-User-IP` | 122.161.53.33 (real) | 🟢 Moss strips it |

**Root cause:** Moss `Startup`/`ServiceProxyExtensions.SetUserIpHeader` sets `X-User-IP` = `CF-Connecting-IP` **first** (when present), else the forwarded-headers-resolved `RemoteIpAddress`; it strips inbound `X-User-IP` but NOT inbound `CF-Connecting-IP`. On **direct (non-Cloudflare) brands** — DeliverPlus `*.deliverplus.uk` → Azure 51.11.x — nothing overwrites/strips a client-supplied `CF-Connecting-IP`, so the client forges the user IP outright. On CF-fronted brands (ParcelCompare / ParcelHero / PH-IE) Cloudflare sets `CF-Connecting-IP` from the true connection and drops client copies, so they're safe.

**Impact:** on direct brands the mandate-acceptance IP (BUC-412 Stripe mandate) is forgeable → weak legal-acceptance evidence. Pre-existing edge behaviour (NOT introduced by Moss #187 / Pistacia #220 — those preserved the prior posture). Not a #220 blocker; warrants a follow-up ticket.

**Fix options:** trust `CF-Connecting-IP` only when the request truly came via Cloudflare — gate on CF-fronted hostnames, or verify the socket peer is in Cloudflare's IP ranges, or strip inbound `CF-Connecting-IP` on direct brands the same way `X-User-IP` is already stripped.

**How it was tested:** owner handed a short-lived sandbox PVR bearer privately (CLI, never in Slack); ran via `rapi`-style `curl` with forged headers (`buck papi` couldn't inject headers, `rapi` had no Pistacia auth — the token bridged that gap). Keep tokens out of Slack; rotate after such transit.

Related: [[remote-pr-review-methodology]] (the Moss #187 trust-all forwarded-headers note that first flagged the XFF-spoof concern).
