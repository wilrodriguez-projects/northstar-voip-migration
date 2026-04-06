# Section 1 — Program Overview

**Document:** `docs/00-program-overview.md`  
**Program:** Project NORTH STAR  
**Version:** 1.4  
**Last Updated:** 2024-07-12  
**Author:** [TPM Name]  
**Status:** Active — Sprint 14 of 22

---

## 1.1 Problem Statement

### The Situation

Acme Corp operates a **Cisco Unified Communications Manager (UCM) 11.5 cluster** hosted in a co-located data center in Ashburn, VA. The system manages all inbound and outbound telephony for the business, including:

- **6,200 Direct Inward Dial (DID) numbers** assigned across sales, support, and operations teams
- **48 toll-free numbers** used in customer-facing marketing campaigns and support lines
- **34 IVR call flows** handling an average of **12,000 inbound calls per day**
- **SIP trunking** via Lumen Technologies (primary) and AT&T (backup)

### The Business Problem

The Cisco UCM system faces three converging pressures:

**1. End of Support (EOS)**  
Cisco announced End of Software Maintenance for UCM 11.5 on **December 31, 2024**. After this date, no security patches, bug fixes, or technical support will be available. Continuing to operate post-EOS creates unacceptable security and compliance exposure, particularly against our SOC 2 Type II obligations.

**2. Operational Cost**  
Annual total cost of ownership for the legacy PBX stack:

| Cost Component | Annual Cost |
|---|---|
| Cisco UCM maintenance contract | $142,000 |
| Co-location hosting (rack, power, cooling) | $88,000 |
| SIP trunk fees (Lumen + AT&T legacy rates) | $67,000 |
| Internal ops labor (Tier 2/3 support) | $43,000 |
| **Total Annual Cost** | **$340,000** |

AWS Connect at current projected call volume is estimated at **$96,000/year** — a savings of **$244,000 annually** at full deprecation.

**3. Capability Ceiling**  
The legacy IVR has no analytics, no ML-based routing, and no integration with our CRM (Salesforce). Competitive analysis shows that three of our top five competitors have deployed intelligent call routing with Contact Lens for Amazon Connect, enabling CSAT measurement, agent coaching, and real-time sentiment analysis. Our current platform cannot support these capabilities.

---

## 1.2 Business Impact

### Financial Impact

| Scenario | 3-Year Cost |
|---|---|
| Do nothing (maintain legacy) | $1,020,000 |
| Migrate to AWS Connect (this program) | $288,000 (infra) + $2.4M (migration) = $2.69M one-time, then $96K/yr |
| Net 3-year savings post-migration | ~$450,000 |
| Break-even point | Month 14 post go-live |

### Operational Impact

- **Risk reduction:** Elimination of single-datacenter-of-failure for all inbound calls
- **Reliability:** SLA improvement from 99.7% (Cisco UCM measured) to 99.9% (AWS Connect SLA)
- **Scalability:** Elastic capacity for call volume spikes (seasonal peaks, product launches)
- **Compliance:** SOC 2 / PCI DSS alignment; Cisco EOS removes compliance evidence capability

### Strategic Impact

- Unlocks AWS Contact Lens for ML-based agent coaching and CSAT measurement
- Enables Salesforce CTI integration (deferred to Phase 2)
- Reduces dependency on specialized Cisco CCIE talent for platform maintenance
- Creates foundation for AI-assisted routing (planned FY2025 initiative)

---

## 1.3 Stakeholders

### RACI Summary

| Stakeholder | Name | Role | R | A | C | I |
|---|---|---|---|---|---|---|
| Executive Sponsor | Sarah Chen | VP Engineering | | ✅ | | |
| Product Owner | Marcus Okafor | Director of Product | ✅ | | ✅ | |
| Technical Program Manager | [TPM Name] | TPM | ✅ | ✅ | | |
| Lead Platform Engineer | Priya Nair | Staff Engineer, Platform Infra | ✅ | | | |
| Telecom Operations | James Whitfield | Sr. Manager, Telecom Ops | ✅ | | ✅ | |
| QA Lead | Dana Kim | QA Engineering Lead | ✅ | | | |
| Finance / FP&A | Tom Reyes | Sr. Financial Analyst | | | ✅ | ✅ |
| CX / Contact Center | Diane Torres | Director, Customer Experience | | | ✅ | ✅ |
| Security | Raj Patel | Sr. Security Engineer | | | ✅ | |

*R = Responsible, A = Accountable, C = Consulted, I = Informed*

### Stakeholder Profiles

**Sarah Chen — VP Engineering (Executive Sponsor)**  
Primary concern: On-time decommission of the Cisco PBX before EOS date. Sarah approved the $2.4M budget in Q4 2023 and reports program status to the CFO quarterly. She needs clean, concise communication — RAG status with one-sentence context. She will escalate to CEO if budget overrun exceeds 10%.

**Marcus Okafor — Director of Product (Product Owner)**  
Represents the business outcome. Cares deeply about zero disruption to inbound call SLA during migration. Has 4 active A/B tests running on IVR flows and pushed back on the greenfield IVR rebuild (see Trade-off #3 in `docs/04-technical-tradeoffs.md`). Requires weekly touchpoint during active wave periods.

**Priya Nair — Staff Engineer, Platform Infra (Lead Engineer)**  
Owns all AWS infrastructure. Primary concern is IaC coverage and configuration drift. Flagged the Terraform local state issue (TD-003) and the Lambda hardcoded credentials issue (TD-001) during Wave 1 retrospect. Priya is the single point of failure for SIP trunk configuration — a staffing risk being actively mitigated (see R-006).

**James Whitfield — Sr. Manager, Telecom Ops (Carrier Liaison)**  
Owns all carrier relationships and LOA (Letter of Authorization) processing. Source of the Wave 2 delay (Lumen backlog — R-001). Has the working relationship with Bandwidth.com that enabled the dual-carrier mitigation strategy. Must be involved in all porting decisions.

**Dana Kim — QA Engineering Lead**  
Owns all IVR regression testing. Raised the test coverage gap (R-004) — coverage was at 47% entering Wave 2, now at 61% after a dedicated sprint. Target of 80% is gated for Wave 3 go-live approval.

**Tom Reyes — Sr. Financial Analyst (Finance)**  
Tracks budget burn against milestones. Flagged the AWS Connect usage overrun in Q2 (R-005). Does not need to be in the weeds of technical decisions but requires accurate monthly cost-to-complete projections.

---

## 1.4 Program Phases

| Phase | Name | Duration | Status |
|---|---|---|---|
| Phase 0 | Discovery & Architecture Lock | Jan 2024 (4 wks) | ✅ Complete |
| Wave 1 | Dev/Staging Migration + ALB Cutover | Feb–Apr 2024 (10 wks) | ✅ Complete |
| Wave 2 | Production DID Block Porting (3,842 of 6,200) | May–Sep 2024 (18 wks) | 🟡 In Progress |
| Wave 3 | Toll-Free + IVR Cutover | Oct–Nov 2024 (8 wks) | 🔴 At Risk |
| Phase 3 | Legacy PBX Decommission | Nov 2024 (2 wks) | ⏳ Planned |

---

## 1.5 Success Criteria

The program is considered successful when all of the following are true:

- [ ] 100% of 6,200 DIDs are ported and operational on AWS Connect
- [ ] 100% of 48 toll-free numbers are ported and operational
- [ ] All 34 IVR flows pass regression testing (≥95% test coverage)
- [ ] Uptime SLA ≥99.9% maintained for 30 days post-cutover
- [ ] Cisco UCM cluster decommissioned (hardware returned to co-lo)
- [ ] IaC coverage ≥90% for all AWS Connect resources
- [ ] All P1 technical debt items resolved
- [ ] Final program cost within 10% of $2.4M budget

---

*Next: [System Architecture →](./01-architecture.md)*
