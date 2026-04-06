# Section 4 — Dependency Management

**Document:** `program-management/dependency-log.md`
**Program:** Project NORTH STAR
**Version:** 2.3
**Last Updated:** 2024-07-12
**Owner:** [TPM Name]
**Review Cadence:** Weekly — every Monday before standup

---

## 4.1 Dependency Management Methodology

### What Counts as a Dependency

A dependency is any external input, deliverable, or decision that must be completed before a program workstream can proceed. On Project NORTH STAR, dependencies fall into four categories:

| Category | Examples | Risk Level |
|---|---|---|
| External Vendor | Carrier LOA processing, CNAM API readiness, AWS service limits | HIGH — outside our control |
| Internal Team | QA coverage gates, IaC drift remediation, security reviews | MEDIUM — can be escalated |
| Platform / Infra | AWS Connect service limit increases, Terraform state migration | LOW — predictable timelines |
| Decision / Approval | Executive sign-off on scope changes, budget approvals | MEDIUM — stakeholder-dependent |

### Dependency Tracking Process

1. **Identify** — All dependencies logged in this register at project kickoff and updated as new ones emerge
2. **Assign** — Every dependency has a single owner and a due date
3. **Track** — Overdue dependencies surface automatically via Jira filter (query: `type = Dependency AND dueDate < now() AND status != Done`)
4. **Escalate** — Dependencies overdue by >5 days trigger automatic Slack alert to TPM and owner
5. **Replan** — Any dependency slip >2 weeks triggers a formal impact assessment (see Section 4.3)

---

## 4.2 Dependency Register

| ID | Dependency | Type | Owner | Due Date | Status | Blocks | Notes |
|---|---|---|---|---|---|---|---|
| DEP-001 | Lumen LOA processing — 2,358 DID block | External Vendor | James W. | 2024-07-05 | 🔴 DELAYED +90d | Wave 2 DID porting | See full scenario: Section 4.3 |
| DEP-002 | Bandwidth.com contract execution | External Vendor | James W. | 2024-07-10 | ✅ Complete | DEP-001 mitigation | Signed 2024-07-08 |
| DEP-003 | CNAM vendor API — production readiness | External Vendor | James W. | 2024-08-01 | 🟡 At Risk | Wave 3 real-time CNAM | Static fallback active — TD-007 |
| DEP-004 | IVR regression coverage ≥80% | Internal / QA | Dana K. | 2024-09-15 | 🟡 In Progress | Wave 3 go/no-go gate | At 61% — 2 sprints dedicated |
| DEP-005 | Terraform state migration to S3 | Internal / Infra | Priya N. | 2024-07-19 | 🟡 In Progress | IaC team access | TD-003 remediation |
| DEP-006 | SCP console-change blocking policy | Internal / Infra | Priya N. | 2024-07-22 | 🟡 In Progress | R-002 mitigation | Blocks console drift |
| DEP-007 | AWS Connect service limit increase | Platform | Priya N. | 2024-06-20 | ✅ Complete | Wave 2 call volume | Raised via support ticket |
| DEP-008 | Executive approval — Option B replan | Internal / Decision | Sarah C. | 2024-07-05 | ✅ Complete | Wave 2 replan | Approved 2024-07-05 |
| DEP-009 | Runbook documentation — SIP config | Internal / Staffing | Priya N. | 2024-08-01 | 🟡 In Progress | R-006 mitigation | Cross-training dependency |
| DEP-010 | Wave 3 go/no-go gate approval | Internal / Decision | Sarah C. | 2024-10-01 | ⬜ Planned | Wave 3 cutover | Requires DEP-003, DEP-004 |

---

## 4.3 Critical Dependency Scenario — DEP-001: Lumen Carrier Delay

### Background

On **June 28, 2024**, James Whitfield received an informal notice from the Lumen Technologies account manager that our Letter of Authorization (LOA) submission for 2,358 DID numbers had been flagged for manual review due to an internal staffing backlog in Lumen's porting operations team.

On **July 1, 2024**, formal written confirmation was received: LOA processing would be delayed by **90 days**, moving the expected First Order Confirmation (FOC) date from July 5 to October 3, 2024.

### Impact Analysis

**Immediate Impact (Wave 2):**

| Item | Original Date | Revised Date | Slip |
|---|---|---|---|
| Wave 2 DID porting complete | 2024-07-05 | 2024-09-30 | +87 days |
| DIDs operational on AWS Connect | 2024-07-05 | 2024-10-01 | +88 days |

**Cascading Impact (Wave 3 & beyond):**

| Item | Original Date | Revised Date | Slip |
|---|---|---|---|
| Wave 3 (TF + IVR) go-live | 2024-09-01 | 2024-12-15 | +105 days |
| Legacy PBX decommission | 2024-10-15 | 2025-01-15 | +92 days |
| Annual savings realization | 2025-01-01 | 2025-04-01 | +90 days |

**Financial Impact:**

| Cost Item | Amount | Notes |
|---|---|---|
| Additional legacy PBX operation (3 months) | $85,500 | $28,500/mo × 3 months |
| Option B (Bandwidth.com parallel port) | $4,200 | One-time — contract + porting fees |
| Delayed savings realization | $61,000 | $244K/yr savings × 3/12 months delayed |
| **Total financial exposure** | **$150,700** | If no mitigation action taken |

---

### Replanning Options

Three options were formally evaluated and presented to Sarah Chen on July 5, 2024:

#### Option A — Accept the Slip, Compress Downstream

**Approach:** Accept the 90-day Lumen delay. Overlap Wave 3 prep with Wave 2 execution. Add 1 FTE contractor for IVR configuration work.

| Factor | Detail |
|---|---|
| Schedule recovery | ~6 weeks (partial compression) |
| Cost | +$32,000 (contractor, 8 weeks) |
| Risk | Parallel workstreams increase coordination overhead. Team already at capacity. |
| Dependency | Lumen remains single carrier — 90-day slip could extend further |

#### Option B — Dual-Carrier Parallel Port ✅ SELECTED

**Approach:** Initiate a parallel DID port with Bandwidth.com while Lumen processes the LOA. Both carriers work simultaneously. BYOC interim routing keeps calls running throughout.

| Factor | Detail |
|---|---|
| Schedule recovery | ~4 weeks (Wave 2 completion: Sep 30 vs. original Jul 5) |
| Cost | +$4,200 one-time (Bandwidth.com contract + porting fees) |
| Risk | LOW — Bandwidth.com has faster LOA processing (avg 15 days vs. Lumen 90+) |
| Dependency | Removes single-carrier risk permanently |

#### Option C — Escalate to Lumen Executive Contacts

**Approach:** Escalate through Lumen account management to VP-level to expedite processing. No parallel carrier work.

| Factor | Detail |
|---|---|
| Schedule recovery | Unknown — depends on Lumen responsiveness |
| Cost | $0 |
| Risk | HIGH — no guarantee of acceleration. Puts program at mercy of vendor politics. |
| Dependency | Still single-carrier dependent |

---

### Decision: Option B Selected

**Decision date:** July 5, 2024
**Decision maker:** Sarah Chen (VP Engineering)
**Recommended by:** [TPM Name]
**Rationale:** Option B provides the best risk-adjusted outcome. The $4,200 cost is negligible compared to the $85,500 exposure from 3 months of additional PBX operation. Dual-carrier strategy also permanently reduces single-carrier dependency risk, which was an architectural concern raised in Wave 1 planning but deferred due to cost.

Logged in Decision Register as **DR-007**.

---

### Stakeholder Communication

The following email was sent on July 3, 2024 — 48 hours after formal confirmation of the delay, and 2 days before the executive decision meeting.

---

**From:** [TPM Name]
**To:** Sarah Chen, Marcus Okafor, Tom Reyes
**CC:** James Whitfield, Priya Nair
**Date:** July 3, 2024
**Subject:** [NORTH STAR] Wave 2 Schedule Impact — Lumen Carrier Delay Confirmed + Options

---

Hi team,

I want to get ahead of an issue confirmed this week before it becomes a surprise in our next status report.

**What happened:** Lumen Technologies has formally notified us of a 90-day delay on our DID block LOA processing. Their porting operations team has an internal backlog, and our 2,358-number batch was flagged for manual review. FOC date moves from July 5 → October 3.

**What this means for the program:**
- Wave 2 DID porting completion: July 5 → **September 30** (+87 days)
- Wave 3 (TF/IVR) go-live: September 1 → **December 15** (+105 days)
- Legacy PBX decommission: October 15 → **January 2025**
- Additional PBX costs at $28,500/month for 3 months = **$85,500**

**Good news — inbound calls are not affected.** BYOC interim routing via Twilio is active. All 12,000 daily calls are routing normally. This is a porting schedule issue only.

**I am recommending Option B:** Initiate a parallel DID port with Bandwidth.com. Cost: $4,200 one-time. Expected schedule recovery: ~4 weeks. This also permanently eliminates our single-carrier dependency.

**I need 30 minutes with Sarah and Tom by end of day Friday** to authorize Option B and update the program schedule formally. I will have a revised Gantt, updated risk register, and cost-to-complete projection ready by Thursday EOD.

I will continue weekly LOA status calls with the Lumen account manager. If there is any acceleration possible on their end, we will take it in addition to Option B.

Happy to discuss on Slack or by phone if questions before Friday.

— [TPM Name]

---

### Post-Decision Actions

| Action | Owner | Completed |
|---|---|---|
| Bandwidth.com contract executed | James W. | 2024-07-08 ✅ |
| Revised program schedule published | TPM | 2024-07-08 ✅ |
| Risk register updated (R-001, R-007) | TPM | 2024-07-08 ✅ |
| Decision Register entry DR-007 created | TPM | 2024-07-08 ✅ |
| Weekly Lumen LOA status calls initiated | James W. | 2024-07-08 ✅ |
| Bandwidth.com port scheduled Aug 15 | James W. | 2024-07-10 ✅ |
| All-hands program update sent | TPM | 2024-07-10 ✅ |

---

## 4.4 Dependency Map — Wave 2 Critical Path

```
[Lumen LOA Processing] ──── DEP-001 (DELAYED) ────────────────────────────────┐
                                                                                │
[Bandwidth.com Contract] ── DEP-002 (Complete) ──► [Parallel Port Aug 15] ──► │
                                                                                ▼
[BYOC Interim Routing] ──────────────────────────────────────────────────► [Wave 2 DIDs Complete]
                                                                                │
                                    [IVR Coverage ≥80%] ─── DEP-004 ──────►   │
                                    [CNAM API Ready] ─────── DEP-003 ──────►   │
                                                                                ▼
                                                                          [Wave 3 Go/No-Go]
                                                                                │
                                                                                ▼
                                                                     [Legacy PBX Decommission]
```

---

*Back: [Risk Register](./risk-register.md) | Next: [Technical Trade-offs →](../docs/04-technical-tradeoffs.md)*
