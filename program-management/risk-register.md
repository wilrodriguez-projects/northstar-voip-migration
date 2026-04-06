# Section 3 — Risk Management Framework

**Document:** `program-management/risk-register.md`  
**Program:** Project NORTH STAR  
**Version:** 3.2  
**Last Updated:** 2024-07-12  
**Owner:** [TPM Name]  
**Review Cadence:** Weekly — every Friday standup

---

## 3.1 Risk Management Methodology

### How Risks Are Identified

Risks on Project NORTH STAR are surfaced through five channels:

| Channel | Method | Frequency | Owner |
|---|---|---|---|
| Weekly risk review | Standing agenda item in Friday standup | Weekly | TPM |
| Pre-mortem sessions | Structured "what could go wrong" before each wave | Per wave | TPM + leads |
| Dependency tracking | Automated Jira query for overdue dependency tasks | Daily | TPM |
| Retrospect findings | Risk items surfaced during sprint retrospects | Bi-weekly | Scrum team |
| Stakeholder escalations | Ad-hoc risks raised by stakeholders outside cadence | As needed | Any stakeholder |

### Risk Scoring Model

Each risk is scored on two dimensions using a 1–3 scale:

| Score | Likelihood | Impact |
|---|---|---|
| 3 — High | Likely to occur within current sprint/wave | Threatens schedule >2 weeks, budget >$10K, or SLA |
| 2 — Medium | Possible within current program timeline | Delays a deliverable, requires replanning |
| 1 — Low | Unlikely but plausible | Minor disruption, manageable within team |

**Severity = Likelihood × Impact**

| Severity Score | Rating | Response |
|---|---|---|
| 9 | CRITICAL | Immediate escalation to executive sponsor. Daily tracking. |
| 6 | HIGH | TPM-owned mitigation plan within 24 hrs. Weekly exec update. |
| 4 | MEDIUM | Mitigation plan within 1 sprint. Bi-weekly review. |
| 1–2 | LOW | Monitor. Review monthly. |

### Risk Lifecycle

```
IDENTIFIED → ASSESSED → MITIGATION PLANNED → ACTIVE MITIGATION → RESOLVED / ACCEPTED
     │              │              │                    │                    │
  Risk log      Score 1-9      Owner assigned       Sprint tracked      Closed in register
  created       assigned       within 24hrs         until resolved      with lessons learned
```

---

## 3.2 Risk Register

*Last reviewed: 2024-07-12 · Sprint 14 · Total open risks: 11 · High: 2 · Medium: 4 · Low: 5*

| ID | Risk Description | Category | Likelihood | Impact | Severity | Owner | Mitigation Strategy | Status | Last Updated |
|---|---|---|---|---|---|---|---|---|---|
| R-001 | **Carrier porting delay (Lumen)** — LOA rejections causing slip on DID block transfer | Vendor | 3 | 3 | **9 — CRITICAL** | James W. | Dual-carrier strategy via Bandwidth.com; BYOC interim routing active | Mitigating | 2024-07-08 |
| R-002 | **IaC drift in AWS Connect** — manual console changes creating Terraform state conflicts | Technical | 3 | 3 | **9 — CRITICAL** | Priya N. | Import drift to state; SCP blocking console edits; IaC gate in CI | In Progress | 2024-07-10 |
| R-003 | **CNAM vendor API not production-ready** — delays real-time caller ID for toll-free numbers | Vendor | 2 | 3 | **6 — HIGH** | James W. | Static CSV CNAM fallback; Wave 3 scope reduced; TD-007 logged | Mitigating | 2024-07-05 |
| R-004 | **IVR regression coverage <80%** — insufficient test coverage for migrated call flows | Quality | 2 | 2 | **4 — MEDIUM** | Dana K. | Dedicated test sprint; Twilio Studio test harness added; at 61% | In Progress | 2024-07-11 |
| R-005 | **AWS Connect usage overrun** — per-minute pricing exceeded Q2 projection by 18% | Financial | 2 | 2 | **4 — MEDIUM** | Tom R. | Reserved pricing negotiated; CloudWatch alerts at 85% cap | Monitored | 2024-06-28 |
| R-006 | **Key SIP engineer attrition risk** — 1 of 2 SIP engineers interviewing externally | Staffing | 2 | 2 | **4 — MEDIUM** | Sarah C. | Runbook documentation sprint; cross-training; comp review requested | In Progress | 2024-07-09 |
| R-007 | **Wave 3 IVR go-live blocked by Wave 2 slip** — cascading delay from R-001 | Schedule | 2 | 2 | **4 — MEDIUM** | TPM | Option B replan compresses schedule; parallel workstream prep started | Mitigating | 2024-07-08 |
| R-008 | **Cisco UCM EOS compliance gap** — if decommission slips past Dec 31 2024 | Compliance | 1 | 3 | **3 — LOW** | Sarah C. | Wave 3 schedule has 6-week buffer; escalation path defined | Monitored | 2024-07-01 |
| R-009 | **DID number loss during porting** — incorrect LOA data causing number to be lost | Technical | 1 | 3 | **3 — LOW** | James W. | LOA validation checklist; dual-verify with carrier before submission | Monitored | 2024-06-15 |
| R-010 | **AWS Connect service limit hit** — concurrent call limit reached during peak | Technical | 1 | 2 | **2 — LOW** | Priya N. | Limit raised via AWS support; CloudWatch alarm at 80% concurrency | Resolved | 2024-06-20 |
| R-011 | **Stakeholder misalignment on IVR greenfield scope** — product pushback on behavior parity gate | Scope | 1 | 2 | **2 — LOW** | TPM | Feature parity gate agreed in writing; A/B test pause plan documented | Resolved | 2024-05-15 |

---

## 3.3 Detailed Mitigation Playbooks

### R-001 — Carrier Porting Delay (CRITICAL)

**Risk Statement:** Lumen Technologies has a backlog in their porting operations team, causing Letter of Authorization (LOA) processing to be delayed by 90 days. This directly blocks Wave 2 DID porting completion and cascades to Wave 3.

**Root Cause:** Lumen internal staffing changes in their porting operations team in Q1 2024 created a processing backlog. Our LOA submission volume (2,358 DIDs in one batch) was flagged for manual review.

**Timeline of Events:**
- 2024-06-28: James Whitfield receives informal notice of processing delay from Lumen account manager
- 2024-07-01: Formal written confirmation received — 90-day delay on LOA processing
- 2024-07-03: TPM sends stakeholder notification email (see dependency-log.md)
- 2024-07-05: Executive sync with Sarah Chen — Option B approved
- 2024-07-08: Bandwidth.com contract executed; parallel port scheduled Aug 15

**Mitigation Actions:**

| Action | Owner | Due Date | Status |
|---|---|---|---|
| Activate BYOC interim routing via Twilio (calls continue uninterrupted) | Priya N. | 2024-07-01 | ✅ Complete |
| Initiate parallel port with Bandwidth.com | James W. | 2024-07-10 | ✅ Complete |
| Replan Wave 2/3 schedule with revised dates | TPM | 2024-07-08 | ✅ Complete |
| Notify all stakeholders with impact analysis | TPM | 2024-07-03 | ✅ Complete |
| Weekly LOA status calls with Lumen account manager | James W. | Ongoing | 🟡 Active |
| Validate Bandwidth.com FOC dates weekly | James W. | Ongoing | 🟡 Active |

**Current Status:** BYOC routing is active — zero customer impact. Bandwidth.com parallel port on track for Aug 15. Wave 2 revised completion: Sep 30, 2024.

---

### R-002 — IaC Drift in AWS Connect (CRITICAL)

**Risk Statement:** Engineers making manual changes in the AWS Console are creating configuration that is not reflected in Terraform state. This creates a divergence between what Terraform thinks exists and what actually exists in AWS, making future `terraform apply` operations dangerous.

**Root Cause:** During Wave 1, time pressure led to two engineers making "quick fixes" in the AWS Console rather than updating Terraform code. This set a bad precedent and the practice continued.

**Mitigation Actions:**

| Action | Owner | Due Date | Status |
|---|---|---|---|
| Audit current state — run `terraform plan` to identify all drift | Priya N. | 2024-07-15 | 🟡 In Progress |
| Import drifted resources into Terraform state | Priya N. | 2024-07-22 | ⬜ Planned |
| Apply SCP (Service Control Policy) blocking console edits to Connect | Priya N. | 2024-07-19 | 🟡 In Progress |
| Add `terraform plan` as required CI gate on all PRs | Priya N. | 2024-07-12 | ✅ Complete |
| Document IaC-first policy in team runbook | TPM | 2024-07-15 | 🟡 In Progress |
| Retrospect — root cause analysis with team | TPM | 2024-07-19 | ⬜ Planned |

---

### R-006 — Key Engineer Attrition Risk (MEDIUM)

**Risk Statement:** One of the two engineers with deep SIP/AWS Connect expertise is actively interviewing externally. If this engineer departs before Wave 3, the program loses critical institutional knowledge with no documented backup.

**Root Cause:** Compensation not competitive with market rate. Engineer has been with the company 3 years with no meaningful raise. External market is paying 22% above current comp for this skill set.

**Mitigation Actions:**

| Action | Owner | Due Date | Status |
|---|---|---|---|
| Initiate comp review with HR | Sarah C. | 2024-07-12 | 🟡 In Progress |
| Begin runbook documentation sprint (SIP config, porting procedures) | Priya N. | 2024-07-15 | 🟡 In Progress |
| Cross-train second engineer on SIP trunk configuration | Priya N. | 2024-08-01 | ⬜ Planned |
| Identify contractor backup (CCIE-certified SIP engineer) | TPM | 2024-07-19 | ⬜ Planned |

**Note:** This risk is being managed carefully. The engineer has not been told they are flagged as a flight risk. All mitigation actions are framed as "knowledge sharing" and "documentation improvement" — standard program hygiene.

---

## 3.4 Risk Trends

| Sprint | Critical | High | Medium | Low | Total |
|---|---|---|---|---|---|
| Sprint 10 | 1 | 2 | 5 | 3 | 11 |
| Sprint 11 | 1 | 2 | 5 | 3 | 11 |
| Sprint 12 | 2 | 2 | 4 | 3 | 11 |
| Sprint 13 | 2 | 2 | 4 | 3 | 11 |
| **Sprint 14** | **2** | **2** | **4** | **3** | **11** |

**Trend Analysis:** Critical risks increased from 1 to 2 in Sprint 12 when R-002 (IaC drift) was identified during a routine Terraform audit. Two risks resolved since Sprint 10 (R-010, R-011). No net reduction in open risks — Wave 2 complexity is generating new risks at the same rate as mitigations are closing existing ones. Target: reduce to 1 critical and 2 high by Sprint 16.

---

*Back: [Architecture](../docs/01-architecture.md) | Next: [Dependency Log →](./dependency-log.md)*
