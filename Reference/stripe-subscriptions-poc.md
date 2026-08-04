---
name: stripe-subscriptions-poc
description: PVR Membership Subscriptions POC — architecture, decisions, and key
  Stripe learnings (Checkout + webhooks + band ladder + Customer Portal). Built
  with Ankur, 2026-08-04.
metadata:
  node_type: memory
  type: reference
---
Proof-of-concept for PVR Membership Subscriptions, built end-to-end with Ankur on 2026-08-04. Spins the spec (`Membership_Subscriptions_v0.2.md`, 4 Aug engineering call) into runnable code. Sibling of the logistics card work — see [[card-management-setup-intent-key-outcomes]] (deliberately kept separate: Stripe Subscriptions has its OWN card storage and does not reuse the logistics SetupIntent/PaymentIntent mechanisms).

## Where it lives
- Repo: `C:\Users\arpan\repos\ph\Spikes\20260804-stripe-subscriptions` (Node/Express, port 9998, mirrors the FE-flow spike's `server.js` style).
- Files: `subscription-server.js` (endpoints + webhook receiver), `config.js` (band ladder by lookup_key), `scripts/stripe_catalogue_seed.js` (provisions catalogue), `public/index.html` (product picker + Manage button), `public/webhooks.html` (webhook log page).
- Stripe **test** account `…Cu7Eq6LYq9` (local :8054 / tenant e1702761 — same account as the FE-flow spike).

## Spec critical path — POC coverage
The 5 PVR build items (spec §1):
1. Restrict signup to logged-in users — ❌ NOT in POC (PVR auth/registration concern; POC assumes a logged-in user by starting from a supplied `pvrAccountId`).
2. Create PVR account at email-verification step — ❌ NOT in POC (registration flow; `pvrAccountId` handed in).
3. **Persist Stripe customer ID against the PVR account** — ✅ the core of the POC ("the one genuinely new backend piece"). In-memory Map + reverse index, standing in for the Acacia/Aspen/payment modelling session (§4) that will decide real storage.
4. On return, fetch tier/status from Stripe and cache — ✅ `GET /subscriptions/:pvrAccountId` → `refreshFromStripe()`. Stripe is source of truth; recomputes nothing.
5. Webhook receiver keyed off customer ID; on `billing.alert.triggered` move sub to next band's Price + refresh tier — ✅ verified pro-50 → pro-100.

Items 3–5 (the genuinely-PVR build) are done + demonstrated; 1–2 are out of scope (wider platform).

## Architecture / endpoints
- `POST /subscriptions/checkout` — **preferred**, hosted. Opens a subscription-mode Stripe Checkout Session (30-day trial). Stripe captures the card + creates Customer/Subscription; `checkout.session.completed` persists the customer id + caches tier. Accepts `priceId` (picker) or `bandIndex`.
- `POST /subscriptions/create` — raw stub that creates Customer+Subscription server-side and persists synchronously (kept for webhook/band testing without a browser).
- `GET /products` — lists active recurring Prices from Stripe; tags configured bands; filters £0 trial helpers.
- `POST /subscriptions/portal` — Customer Portal session (see below).
- `POST /webhook` — mounted with `express.raw` BEFORE `express.json` (signature needs raw body). Handles `checkout.session.completed`, `customer.subscription.*`, `billing.alert.triggered`; ignores the rest. In-memory ring buffer surfaced at `GET /webhook-log` + `public/webhooks.html`.

## Catalogue + band ladder
- `scripts/stripe_catalogue_seed.js` (idempotent) provisions: products (Parcelhero Pro, Pro Scale, Free), 22 prices (Pro/Pro Scale × 5 monthly + 5 annual bands @ 50/100/500/1000/2000 + a £0 monthly trial helper each), 2 meters (`shipment_count`, `spend_amount`), 5 activated `usage_threshold` billing alerts.
- Prices identified by **`lookup_key`** — PVR code references lookup_keys, NEVER hard-coded price ids. Server `resolveBands()` resolves each to a live price id at startup.
- POC band ladder = Pro monthly (`pro_monthly_50 → 100 → 500 → 1000 → 2000`), ceilings matching alert thresholds.

## KEY LEARNINGS (the non-obvious bits)
1. **pvrAccountId visibility.** A Customer that Checkout auto-creates does NOT inherit the session's metadata → the id lands on the session (`client_reference_id`) + subscription metadata but the CUSTOMER shows blank. Fix: PRE-CREATE the Stripe Customer with `metadata.pvrAccountId` and pass `customer:` to Checkout (also the spec's "PVR creates the Stripe Customer" shape), plus a webhook-side `customers.update` safety net.
2. **Trial = "pay later", not free.** Every sub is created with `trial_period_days: 30` → first invoice is £0, status `trialing`, no money moves until trial end (Stripe then auto-charges the saved card; dunning is Stripe's). A no-trial sub charges immediately (proven: a $15 no-trial sub → `$15 paid`). To test a charge now: `subscriptions.update(sub, { trial_end: 'now' })`. Amount comes from the subscription's current Price — which is why band-crossing "move to next band's Price" is what changes what the customer pays. PVR builds NO charging code.
3. **Customer Portal interval constraint.** Stripe's portal plan-switcher requires each product's listed prices to have UNIQUE billing intervals → it CANNOT offer the 5 same-interval Pro band prices for self-serve switching. So portal "upgrade" = product-level switch **Pro ↔ Pro Scale** (one representative price each). Band upgrades are usage-driven/automatic via billing-alert webhooks, not customer self-serve. Portal config is created lazily (test mode has no default); card update / cancel-at-period-end / invoice history always apply. Portal changes return as `customer.subscription.*` webhooks the existing receiver already handles — no new webhook code.
4. **Webhook delivery.** No dashboard-registered endpoint in the POC. Local receiver `http://localhost:9998/webhook`; Stripe reaches it via the **Stripe CLI** `stripe listen --forward-to …`. The `whsec_…` verified against is the CLI listener's secret, NOT a dashboard endpoint secret. Real deploy: register a public HTTPS endpoint in the dashboard + use THAT endpoint's signing secret.
5. **CLI as portable binary.** Stripe CLI 1.45.0 lives at `.tmp/stripe.exe` (no system install); authenticate non-interactively with `--api-key` (no browser login). Correlated events Stripe can't emit on demand (subscription-mode `checkout.session.completed`, `billing.alert.triggered`) were signed locally with the SDK's `generateTestHeaderString(whsec)` so real signature verification still runs.

## OPEN ITEMS (still unresolved)
- **Billing-alert re-arm** — the KEY verification. Seed creates alerts with `recurrence: "one_time"`, but the band model assumes alerts re-arm per customer per billing cycle across multiple crossings. Not yet proven with real metered usage (needs actual `shipment_count` meter events crossing a threshold, not a synthesized alert).
- **Persistence** — customer-id link is in-memory; awaits the Acacia (onboarding) / Aspen (user-auth) / payment-service modelling session to decide the real store.
- **Overage prices** above 2,000/mo band deliberately omitted from the seed (pending product decision; 2,000+ is "Custom").
- **3DS on first subscription card entry** — spec open, non-blocking.

## Verified end-to-end (all test mode)
Full `stripe listen` round-trip (signed events): `customer.subscription.created`, correlated subscription-mode `checkout.session.completed` (persisted customer id), `billing.alert.triggered` (actually moved a real subscription's price band). Band-crossing on the real lookup_key ladder (pro-50 → pro-100). Checkout customer now carries `metadata.pvrAccountId`. Portal session opens (`bpc_…` config + `billing.stripe.com/p/session/test_…`).
