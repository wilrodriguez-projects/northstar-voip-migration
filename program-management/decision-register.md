# Decision Register

**Document:** `program-management/decision-register.md`
**Program:** Project NORTH STAR
**Version:** 1.5
**Last Updated:** 2024-07-12
**Owner:** [TPM Name]
**Purpose:** Immutable log of every significant program decision with full context and rationale

---

## Why This Register Exists

Every TPM has been in a meeting where someone says "I thought we agreed to X" and no one can remember why. This register exists to prevent that. Every decision that affects schedule, budget, architecture, or stakeholder commitments is logged here — permanently.

**Rules:**
- Entries are never deleted — only marked superseded
- Every entry names a human decision maker — never "the team"
- Options considered are always documented — not just the chosen one
- Consequences (positive and negative) are stated explicitly

---

## Decision Register

| ID | Decision | Date | Decision Maker | Category | Status |
|---|---|---|---|---|---|
| DR-001 | Twilio Elastic SIP selected as BYOC interim carrier | 2024-01-18 | Priya Nair | Architecture | Active |
| DR-002 | Greenfield IVR rebuild with feature parity gate | 2024-01-28 | TPM (facilitated) | Architecture | Active |
| DR-003 | Cisco UCM parallel operation maintained through Wave 3 | 2024-02-01 | Sarah Chen | Operations | Active |
| DR-004 | Single-region (us-east-1) Multi-AZ — multi-region deferred to FY2025 | 2024-02-02 | Sarah Chen | Architecture | Active |
| DR-005 | Static CNAM at Wave 3, real-time CNAM deferred to Q1 2025 | 2024-06-15 | Marcus Okafor | Product/Eng | Active |
| DR-006 | CR-007 analytics dashboard deferred to Phase 2 | 2024-06-21 | TPM | Scope | Active |
| DR-007 | Option B — dual-carrier parallel port with Bandwidth.com | 2024-07-05 | Sarah Chen | Vendor | Active |
| DR-008 | Sprint 15 ring-fenced as debt sprint (43% velocity to P1 items) | 2024-07-12 | TPM + Marcus Okafor | Delivery | Active |

---

## DR-007 — Full Detail (Most Recent Critical Decision)

**Decision:** Execute Option B — initiate parallel DID port with Bandwidth.com while Lumen LOA processes.

**Date:** July 5, 2024
**Decision Maker:** Sarah Chen (VP Engineering)
**Recommended by:** [TPM Name]
**Context:** Lumen Technologies confirmed 90-day delay on LOA processing for 2,358 DID block. Without mitigation, Wave 2 slips 87 days, cascading to Wave 3 (+105 days) and PBX decommission (+92 days). Total financial exposure: $150,700.

**Options Considered:**

| Option | Cost | Recovery | Risk | Verdict |
|---|---|---|---|---|
| A — Accept slip, compress downstream | +$32,000 | 6 weeks | Team at capacity | Rejected |
| B — Dual-carrier parallel port | +$4,200 | 4 weeks | Low | **Selected** |
| C — Escalate to Lumen executives | $0 | Unknown | High (vendor-dependent) | Rejected |

**Rationale:** $4,200 cost is negligible vs. $150,700 exposure. Option B permanently eliminates single-carrier dependency — an architectural concern from Wave 1 that was deferred on cost grounds. Option C has no guaranteed outcome and keeps the program at the mercy of Lumen's internal operations.

**Consequences:**
- Positive: Wave 2 recovery of ~4 weeks. Single-carrier risk eliminated.
- Negative: Adds Bandwidth.com as a second vendor relationship to manage.
- Accepted risk: If Bandwidth.com FOC slips, Wave 2 recovery is reduced.

**Review date:** August 16, 2024 (post Bandwidth.com FOC confirmation)

---

*Back: [Scope Change Log](./scope-change-log.md)*
