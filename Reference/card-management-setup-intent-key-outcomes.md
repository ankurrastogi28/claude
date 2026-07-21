---
name: card-management-setup-intent-key-outcomes
description: "Design decisions (Key Outcomes) for the Stripe setup-intent card-management workflow — review context for the upcoming card-management PR (Ankur, 2026-07-21)"
metadata:
  node_type: memory
  type: reference
---

Design-session outcomes Ankur posted in Slack (channel C08EQJF2RM0, thread 1784569546.742719, 2026-07-21) and explicitly asked me to memorize as review context for the upcoming card-management PR. Apply this as the SPEC when reviewing that PR — deviations from these decisions are findings.

## Summary
Full card-management workflow moves to **Stripe setup intents** — replacing the previous approach of directly creating payment methods and payment intents from the UI. Soft-deletion via metadata, duplicate-card detection via fingerprint matching, and no unnecessary 3DS re-confirmation for already-confirmed cards.

## Decisions made
1. **Setup-intent flow adopted:** UI calls the setup-intent API on page load, receives `client_secret` + `setup_intent_id`, then passes `setup_intent_id` to the backend save-card endpoint on form submit — NOT a payment-method ID directly.
2. **Soft delete only:** cards are never hard-deleted from Stripe; `is_deleted` flag lives in metadata. Backend returns ALL cards (including deleted); UI decides filtering.
3. **Fingerprint-based duplicate detection:** re-adding an existing card (even previously deleted) is identified via card fingerprint → reset `is_deleted=false`, update nickname — no new 3DS confirmation.
4. **Expired card cannot be set as default:** backend must validate and reject.
5. **Nickname optional:** no automatic fallback name required (last-four digits shown separately).
6. **Single DTO for add/edit:** one request object for both; delete is via the `is_deleted` flag, not a separate payload.

## Pending confirmation
- Does Stripe return the SAME payment-method ID when the same card is re-added via setup intent? (Ankur observed different IDs in POC; Faruk asserts fingerprint matching overrides this.)
- Do mandates carry an expiration date / can the customer cancel them via bank or Stripe account? Needs Stripe docs verification.
- Edge-case combinations to be fully mapped: card saved without setup intent · card saved without mandate · setup intent exists but no mandate.

## Open questions
- Mandate-validity check at card add/update time: does the Stripe API expose mandate status reliably?
- Can a mandate saved on a PAYMENT intent (not just a setup intent) be reused to skip 3DS on later transactions?

## Action items (Ankur's, for context)
- List all card-save edge-case combinations (no mandate, no setup intent, expired card, deleted-card re-add) and query the AI agent, incl. Stripe-docs prompts on mandates/setup intents/payment methods.
- Review the merged expired-card bug fix (ticket ~#406) to understand the existing fingerprint-matching logic first.
- Add `is_deleted` to the list-cards response DTO.
- Feed UI mockup screenshots into the agent to generate accurate DTOs.

## In-thread guidance (Faruk)
- Don't confuse the agent — ask it to DISCOVER possible combinations of cases rather than prescribe.
- "Just show the DTO model."
- Force Stripe-side combinations of different situations (Ankur 👍'd).

Related: [[pr-review-bc-pistacia]] · [[remote-pr-review-methodology]] · [[generic-pr-review-checklist]]
