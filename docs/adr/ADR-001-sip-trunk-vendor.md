# ADR-001 — SIP Trunk Vendor Selection

**Status:** Accepted  
**Date:** 2024-01-18  
**Deciders:** Priya Nair, James Whitfield, [TPM Name]  
**Reviewed by:** Sarah Chen (VP Engineering)

---

## Context

Project NORTH STAR requires a SIP trunking solution to bridge the existing carrier network (Lumen/Bandwidth DIDs) with the new AWS Connect infrastructure during migration. The solution must handle up to 500 concurrent calls, support BYOC (Bring Your Own Carrier) integration with AWS Connect, and operate reliably during a multi-month migration window where both legacy and new systems are active.

Three vendors were evaluated:

| Vendor | Model | Concurrent Calls | AWS Connect BYOC | Monthly Cost (est.) |
|---|---|---|---|---|
| Twilio Elastic SIP | Cloud-native, per-minute | Unlimited (elastic) | Native support | ~$4,200 |
| Vonage SIP | Cloud-native, per-minute | Unlimited (elastic) | Supported | ~$4,800 |
| Bandwidth Direct | Direct carrier, flat rate | 500 max | Manual config | ~$3,100 |

---

## Decision

**Selected: Twilio Elastic SIP (BYOC)**

---

## Rationale

**For Twilio:**
- Native AWS Connect BYOC integration — documented and supported by both Twilio and AWS
- Elastic scaling — no pre-provisioned channel limits during peak migration activity
- Programmatic SIP trunk management via API — enables IaC-managed configuration
- Fastest time-to-configure — team had prior Twilio experience, reducing ramp time
- Built-in failover and redundancy across multiple PoPs

**Against Bandwidth Direct:**
- Lower cost but requires manual LOA/FOC coordination that duplicates existing Lumen relationship
- No native AWS Connect BYOC documentation — integration risk
- 500 concurrent call cap requires capacity planning during cutover windows

**Against Vonage:**
- Higher cost than Twilio with no differentiated capability for this use case
- Less mature AWS Connect BYOC documentation at time of evaluation (Jan 2024)

---

## Consequences

**Positive:**
- Decoupled porting schedule from AWS go-live — calls route through Twilio while DID porting proceeds in parallel
- Twilio provides real-time porting status webhooks used by Lambda (port-status-webhook function)
- Reduced integration risk for Wave 1 due to existing team familiarity

**Negative / Trade-offs:**
- Additional monthly cost (~$4,200/mo) during migration window — estimated 8-month exposure = ~$33,600
- Twilio becomes a dependency during Wave 2 — outage would affect all inbound calls
- Mitigation: Route 53 health checks monitor Twilio endpoint; escalation runbook documented

**Cost is accepted** — the $33,600 Twilio cost during migration is offset by the $85,500 saved by preventing a 3-month delay (R-001 scenario shows the value of carrier independence).

---

## Review Date

To be revisited at Wave 3 go-live. Twilio BYOC layer will be removed once all DIDs are fully ported to Bandwidth.com direct trunking.
