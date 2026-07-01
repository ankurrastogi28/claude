---
name: pachira-seon-decline-test-email
description: "Sandbox test email that triggers a SEON DECLINE in Pachira's fraud check"
metadata: 
  node_type: memory
  type: project
  originSessionId: b358309a-4307-42c4-a471-adac67c7c0f4
---

In **sandbox**, the email **`ashish.parcelhero@gmail.com`** reliably produces a **SEON DECLINE** in Pachira's basket/wallet fraud check — use it as the test customer's billing-contact email to exercise the declined "FraudCheck" problem-details response. (Owner confirmed in Slack 2026-06-26.)

Context for why this works (verified in the Pachira repo `C:\Users\arpan\repos\ph\Pachira`):
- Sandbox does **NOT** mock SEON — `MockExternalDependencies: true` in Pachira.API config is dead config (defined in `ApiOptions`, only printed in the startup dump, never read in code). The `seon` HttpClient always calls live `https://api.seon.io` with the sandbox `seon-apikey`. So the decline comes from SEON's real scoring of that email's footprint.
- A "fail" = SEON returns `state == "DECLINE"` OR `fraud_score >= 90` (`SeonFraudResult.SeonFraudScore = 90`) → `ShippersBasketApplicationService.PerformFraudCheck` throws `ApplicationValidationException("FraudCheck", "Transaction declined due to security concerns. Please contact support.")`.
- Two gates before SEON is even called: Unleash flag `pachira_seon_fraud_check` (basket) / `pachira_wallet_seon_fraud_check` (wallet) — both default OFF, must be ON in sandbox — AND the customer must not have Pistacia setting `SkipFraudCheck = true`.
- SEON is scored from billing-contact email (fallback shipper email) + shipper phone + name + address + card last4/expiry + amount; no IP is sent.

Related: [[buck-mcp-launch-config]] (the bodhi/bucky setup used to answer this in Slack).
