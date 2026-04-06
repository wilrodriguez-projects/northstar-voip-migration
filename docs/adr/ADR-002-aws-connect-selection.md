# ADR-002 — Contact Center Platform Selection (AWS Connect)

**Status:** Accepted  
**Date:** 2024-01-12  
**Deciders:** Sarah Chen, Marcus Okafor, Priya Nair, [TPM Name]  
**Reviewed by:** Raj Patel (Security), Tom Reyes (Finance)

---

## Context

The existing Cisco UCM 11.5 PBX reaches End of Software Maintenance on December 31, 2024. A replacement contact center platform is required that can handle 12,000 inbound calls/day, support 34 IVR flows, integrate with Salesforce (future), and operate within a $96,000/year infrastructure budget post-migration.

Four platforms were evaluated:

| Platform | Model | Annual Cost (est.) | Salesforce Integration | ML Routing | IaC Support |
|---|---|---|---|---|---|
| AWS Connect | Cloud, per-minute | ~$96,000 | CTI adapter available | Contact Lens (native) | Terraform provider |
| Genesys Cloud | Cloud, per-seat | ~$210,000 | Native | Yes | Limited |
| Five9 | Cloud, per-seat | ~$185,000 | Native | Yes | None |
| Twilio Flex | Cloud, per-hour | ~$140,000 | Custom build required | None native | API-driven |

---

## Decision

**Selected: AWS Connect**

---

## Rationale

**For AWS Connect:**
- Lowest 3-year TCO by a significant margin ($288K vs $630K for Genesys)
- Native AWS ecosystem integration — no additional data transfer, VPC peering, or auth layers required
- Contact Lens provides ML-based call analytics, sentiment analysis, and agent coaching natively
- Terraform AWS provider has mature Connect support — aligns with IaC-first architecture principle
- Per-minute pricing model eliminates per-seat licensing overhead for variable-staffing support teams
- AWS SOC 2 / PCI DSS compliance inherited — reduces compliance burden vs. third-party platforms

**Against Genesys Cloud:**
- 2.2x higher annual cost with no differentiating capability for our use case
- Per-seat model penalizes seasonal staffing flexibility

**Against Five9:**
- No IaC support — all configuration is GUI-only, creating drift risk and no audit trail
- Per-seat model, higher cost

**Against Twilio Flex:**
- Requires custom development to replicate IVR functionality — significant engineering investment
- No native ML routing — would require custom build

---

## Consequences

**Positive:**
- $244,000/year savings vs. legacy PBX at steady state
- Contact Lens analytics capability unlocked for FY2025 agent coaching initiative
- Full IaC coverage of contact center infrastructure — first time in company history

**Negative / Trade-offs:**
- AWS Connect has limited IVR versioning — contact flows not natively stored in version control (tracked as TD-004)
- Per-minute cost model creates budget variability — spike risk during campaigns (triggered R-005)
- Mitigation: Reserved concurrency pricing negotiated; CloudWatch usage alerts at 85% monthly cap
- Salesforce CTI integration deferred to Phase 2 — not available at Wave 3 go-live

---

## Review Date

Steady-state review at Month 6 post-decommission (May 2025). Evaluate Contact Lens ROI and Salesforce CTI timeline.
