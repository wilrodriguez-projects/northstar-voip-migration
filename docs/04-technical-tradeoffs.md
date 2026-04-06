# Section 5 — Technical Trade-offs

**Document:** `docs/04-technical-tradeoffs.md`
**Program:** Project NORTH STAR
**Version:** 1.4
**Last Updated:** 2024-07-12
**Owner:** [TPM Name]
**Purpose:** Document engineering vs. product trade-off decisions with full context, options, and outcomes

---

## 5.1 Trade-off Framework

Every trade-off on Project NORTH STAR was evaluated using four questions:

1. **What is the delivery risk if we do this now?** — Schedule, quality, or stability impact
2. **What is the business risk if we defer this?** — Revenue, compliance, or customer impact
3. **Can it be phased?** — Is there a version of this that ships now with a clear path to full delivery later?
4. **Who accepts the risk in writing?** — Every trade-off decision is owned by a named person

No trade-off was made verbally. Every decision is in the Decision Register.

---

## 5.2 Trade-off #1 — Real-Time CNAM vs. On-Time Wave 3 Delivery

**Decision date:** 2024-06-15
**Decision maker:** Marcus Okafor (Product), accepted engineering recommendation
**Logged as:** DR-005

### Context

Caller ID display (CNAM — Calling Name) for toll-free numbers requires integration with a third-party CNAM database vendor. The vendor's API was selected in Wave 1 planning and was expected to be production-ready by May 2024. By June 2024, it was clear the vendor would not be ready before Wave 3 go-live.

### The Tension

| | Engineering Position | Product Position |
|---|---|---|
| **Stance** | Ship Wave 3 without real-time CNAM | Real-time CNAM must ship with Wave 3 |
| **Reasoning** | Vendor API is not load-tested, has no SLA, and has no fallback — shipping it introduces P1 call failure risk for all toll-free callers | 12% of enterprise prospects specifically ask about CNAM in sales discovery. Delay puts us behind Competitor X who launched in June |
| **Risk if wrong** | Shipping unstable API could cause caller ID to display blank or incorrect — immediate customer complaints | Deferring CNAM means 90 days where sales team cannot demo feature to prospects |

### Options Evaluated

| Option | Schedule Impact | Quality Risk | Business Risk |
|---|---|---|---|
| A — Ship real-time CNAM with Wave 3 | None | HIGH — API not production-ready | Low |
| B — Defer CNAM entirely to Q1 2025 | None | None | HIGH — 6-month gap |
| C — Ship static CNAM now, real-time in Q1 2025 | None | LOW — static CSV is reliable | LOW — feature exists, just not real-time |

### Decision: Option C — Static CNAM Now, Real-Time Q1 2025

**What ships at Wave 3:** Static CNAM via CSV upload — caller ID displays correctly for all toll-free numbers. Updates are manual (batch process, weekly refresh).

**What is deferred:** Real-time CNAM API integration. Logged as TD-007 (P2 tech debt). Dedicated 3-sprint track in Q1 2025.

**Why product accepted the deferral:** Engineering provided a written risk assessment confirming that static CNAM has zero SLA impact for the first 90 days (weekly refresh cadence is acceptable for current call volume). Product accepted after TPM framed it as "the feature ships — it just doesn't update in real time yet."

**Consequence accepted:** Sales team must caveat CNAM capability as "weekly refresh" during Q4 sales conversations. Product owner briefed the sales team directly.

---

## 5.3 Trade-off #2 — Multi-Region Failover vs. Cost Ceiling

**Decision date:** 2024-02-02
**Decision maker:** Sarah Chen (VP Engineering)
**Logged as:** DR-004 (also ADR-004)

### Context

Architecture review in January 2024 identified that single-region deployment (us-east-1) creates a blast radius risk: if us-east-1 suffers a full AZ outage during business hours, all 12,000 daily inbound calls would fail. The engineering team proposed active-active multi-region as the gold standard.

### The Tension

| | Engineering Position | Finance / Product Position |
|---|---|---|
| **Stance** | Active-active multi-region (us-east-1 + us-west-2) | Single-region with enhanced monitoring |
| **Reasoning** | AZ failure = total call failure. 4-hour blast radius is unacceptable for a customer-facing system | $18,000/month ($216K/year) for multi-region requires board approval. Not in program budget. Legacy PBX ran at 99.7% — even single-region is a major improvement |
| **Risk if wrong** | Single AZ failure = 4-hour outage for all customers | Multi-region not funded = architectural debt that gets harder to add later |

### Options Evaluated

| Option | Monthly Cost Delta | Recovery Time | Complexity | Decision |
|---|---|---|---|---|
| A — Single-region, Multi-AZ | $0 | ~4 hrs (AZ failure) | Low | ✅ Selected |
| B — Active-active multi-region | +$18,000/mo | ~15 min | High | Deferred to FY2025 |
| C — Warm standby (us-west-2) | +$6,500/mo | ~45 min | Medium | Not selected |

### Decision: Option A — Single-Region with Enhanced Monitoring

**Mitigations added:**
- Route 53 health checks with 30-second failover detection
- CloudWatch alarms at 99% availability with PagerDuty P1 escalation
- Runbook documented for regional outage response
- Multi-region architecture documented as ADR-004 for FY2025 budget submission

**Risk formally accepted** by Sarah Chen in writing. Logged in Decision Register as DR-004.

**FY2025 action:** Tom Reyes to include multi-region infrastructure ($216K/yr) in FY2025 budget proposal with 5-nines SLA business case.

---

## 5.4 Trade-off #3 — Greenfield IVR Rebuild vs. Lift-and-Shift

**Decision date:** 2024-01-28
**Decision maker:** [TPM Name] — facilitated; Marcus Okafor (Product) accepted
**Logged as:** DR-002

### Context

The legacy Cisco UCM IVR has 34 call flows accumulated over 11 years. Some flows are well-documented. Many are not. The team faced a choice: rebuild in AWS Connect from scratch (greenfield) or replicate the existing behavior exactly (lift-and-shift).

### The Tension

| | Engineering Position | Product / CX Position |
|---|---|---|
| **Stance** | Greenfield rebuild | Lift-and-shift with 1:1 behavior parity |
| **Reasoning** | Legacy flows contain unknown debt. Lift-and-shift carries over 11 years of undocumented behavior. Greenfield lets us use AWS Connect ML routing and Contact Lens from day one | CX team has 4 live A/B tests running on IVR. Greenfield breaks tested sequences and loses institutional knowledge. Requests zero behavior change until data supports it |
| **Risk if wrong** | Lift-and-shift locks in legacy behaviors we can't improve | Greenfield delays go-live and breaks active CX experiments |

### Options Evaluated

| Option | Timeline Impact | Quality | Business Impact |
|---|---|---|---|
| A — Full lift-and-shift | Fastest | Carries all legacy debt | CX A/B tests preserved |
| B — Greenfield with feature parity gate | +3 weeks for parity verification | Clean architecture | A/B tests paused 2 weeks |
| C — Hybrid: core flows lift-and-shift, new flows greenfield | +1 week | Partial debt | A/B tests partially preserved |

### Decision: Option B — Greenfield with Feature Parity Gate

**What this means in practice:**
- All 34 call flows rebuilt in AWS Connect using best practices
- A **feature parity gate** added to the Wave 3 go-live checklist: all 34 documented flows must pass regression testing before cutover is approved
- A/B tests paused for 2 weeks during cutover window — product accepted after seeing staging performance data

**Why product accepted:** In staging, the greenfield IVR showed a **15% improvement in Average Speed to Answer (ASA)** vs. the legacy system. TPM presented this data in the trade-off review meeting. Product shifted from "we need identical behavior" to "we need equal or better outcomes."

**Undocumented flows:** 8 of 34 flows had incomplete documentation. A **CX discovery sprint** was added to Wave 2 scope — CX team assigned an owner to document all undocumented flows before Wave 3 regression testing begins.

**Consequence accepted:** 3-week additional timeline for parity verification. Offset by ASA improvement business value.

---

## 5.5 Trade-off Decision Summary

| # | Trade-off | Engineering Wanted | Product Wanted | Decision | Owner |
|---|---|---|---|---|---|
| 1 | CNAM delivery | Defer — API unstable | Ship now — sales need it | Static now, real-time Q1 2025 | Marcus O. |
| 2 | Multi-region failover | Active-active (us-east-1 + us-west-2) | Single-region — budget constraint | Single-region + enhanced monitoring | Sarah C. |
| 3 | IVR approach | Greenfield rebuild | Lift-and-shift parity | Greenfield + parity gate + discovery sprint | TPM |

---

*Back: [Dependency Log](../program-management/dependency-log.md) | Next: [Technical Debt →](../program-management/technical-debt-register.md)*
