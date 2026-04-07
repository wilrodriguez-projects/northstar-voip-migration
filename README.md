# 🚀 Project NORTH STAR — Carrier-Grade VoIP Migration to AWS Connect

> **Program Type:** Cloud Infrastructure · Telecom Migration  
> **Duration:** January 2024 – November 2024 (11 months)  
> **Budget:** $2.4M allocated · $2.1M burned at Sprint 14  
> **Scope:** 6,200 DIDs · 48 Toll-Free Numbers · 34 IVR Flows · Legacy PBX Decommission  
> **Status at Sprint 14:** 🟡 AT RISK — carrier delay mitigated, on revised schedule  
> **Savings at steady state:** $244,000/year vs. legacy PBX

---

## What This Repository Demonstrates

This is a fully documented Technical Program Management portfolio simulating a real-world carrier-grade VoIP migration. Every artifact, decision, risk, and report reflects how a senior TPM operates in a complex, multi-stakeholder technical environment.

| TPM Competency | Artifact | Location |
|---|---|---|
| Risk identification & mitigation | Risk register with 11 risks, 3 playbooks, heat map | [`/program-management/risk-register.md`](./program-management/risk-register.md) |
| Upstream dependency delays | Lumen 90-day slip — full scenario, 3 options, decision | [`/program-management/dependency-log.md`](./program-management/dependency-log.md) |
| Technical debt management | 14-item register, impact scoring, P1 playbooks | [`/program-management/technical-debt-register.md`](./program-management/technical-debt-register.md) |
| Scope creep control | 8 CRs — CR-007 full scenario, 22 weeks protected | [`/program-management/scope-change-log.md`](./program-management/scope-change-log.md) |
| Eng vs. Product trade-offs | 3 real decisions with context, options, and outcomes | [`/docs/04-technical-tradeoffs.md`](./docs/04-technical-tradeoffs.md) |
| System architecture | Component map, data flows, IaC structure, security | [`/docs/01-architecture.md`](./docs/01-architecture.md) |
| Architecture decisions | 4 ADRs with options evaluated and rationale | [`/docs/adr/`](./docs/adr/) |
| Stakeholder communication | RACI, comms plan, escalation templates | [`/docs/02-stakeholder-map.md`](./docs/02-stakeholder-map.md) |
| AI-assisted operations | 6 production Claude prompts with sample outputs | [`/ai-operations/prompt-library.md`](./ai-operations/prompt-library.md) |
| Program execution | Weekly cadence, Jira structure, decision flow, RAG | [`/docs/03-execution-workflow.md`](./docs/03-execution-workflow.md) |
| Weekly status reporting | 2 real reports + reusable template | [`/status-reports/`](./status-reports/) |
| Decision history | 8 program decisions — immutable, with rationale | [`/program-management/decision-register.md`](./program-management/decision-register.md) |

---

## Program Summary

**The Problem:** Acme Corp operated a Cisco Unified Communications Manager (UCM) 11.5 cluster reaching End of Software Maintenance on December 31, 2024. The system managed 6,200 DIDs, 48 toll-free numbers, and a 34-flow IVR handling 12,000 inbound calls/day. Annual cost: **$340,000/year**. Any failure had direct customer impact with no automated failover.

**The Mandate:** Migrate all voice infrastructure to AWS Connect before Cisco EOS, achieving operational parity while enabling ML-based routing and Contact Lens analytics.

**The Outcome (Sprint 14 of 22):** 3,842 of 6,200 DIDs ported. Wave 1 complete. One critical dependency slip (Lumen carrier delay, 90 days) identified, mitigated via dual-carrier strategy, and communicated proactively to executive stakeholders. Program on revised schedule.

---

## System Architecture

```
  ┌──────────────────────────────────────────┐
  │        PSTN / CARRIER LAYER              │
  │   Lumen (DID) · Bandwidth.com (TF+DID)  │
  └──────────────────┬───────────────────────┘
                     │ SIP Trunks
  ┌──────────────────▼───────────────────────┐
  │    Twilio Elastic SIP — BYOC Interim     │
  │    Active during Wave 2 porting          │
  └──────────────────┬───────────────────────┘
                     │
  ┌──────────────────▼───────────────────────┐
  │         AWS us-east-1 (Multi-AZ)         │
  │                                          │
  │  Route 53 → AWS Connect (34 IVR flows)  │
  │  ALB (blue/green) → Lambda Functions    │
  │  DynamoDB · Secrets Manager · Kinesis   │
  └──────────────────────────────────────────┘
                     │
  ┌──────────────────▼───────────────────────┐
  │  Legacy Cisco UCM 11.5 — Decommission   │
  │  Nov 2024 · Co-lo Ashburn VA            │
  └──────────────────────────────────────────┘
```

---

## Key Metrics — Sprint 14 Snapshot

| Metric | Value | Target | Status |
|---|---|---|---|
| DIDs Ported | 3,842 / 6,200 | 4,200 by Sprint 14 | 🟡 Behind (carrier delay) |
| Uptime SLA | 99.91% | 99.9% | 🟢 Met |
| Budget Burn | $2.1M / $2.4M | ≤$2.2M at S14 | 🟢 On track |
| Open High/Critical Risks | 4 | 0 | 🔴 Mitigating |
| Sprint Velocity | 34 pts | 42 pts | 🟡 Degraded |
| IVR Test Coverage | 61% | 80% by Wave 3 | 🟡 Improving |
| P1 Tech Debt Open | 3 | 0 | 🔴 Sprint 15/16 target |
| Scope CRs Deferred | 6 of 8 | Protect schedule | 🟢 22+ weeks protected |

---

## Repository Structure

```
northstar-voip-migration/
│
├── README.md
│
├── docs/
│   ├── 00-program-overview.md
│   ├── 01-architecture.md
│   ├── 02-stakeholder-map.md
│   ├── 03-execution-workflow.md
│   ├── 04-technical-tradeoffs.md
│   └── adr/
│       ├── ADR-001-sip-trunk-vendor.md
│       ├── ADR-002-aws-connect-selection.md
│       └── ADR-003-004-iac-and-region.md
│
├── program-management/
│   ├── risk-register.md
│   ├── dependency-log.md
│   ├── technical-debt-register.md
│   ├── scope-change-log.md
│   └── decision-register.md
│
├── status-reports/
│   ├── template-weekly-status.md
│   └── 2024-W27-W28-status.md
│
└── ai-operations/
    └── prompt-library.md
```

---

## How to Navigate — By Audience

**Hiring Manager / Recruiter:** Start with [`/docs/00-program-overview.md`](./docs/00-program-overview.md) for context. Then [`/program-management/dependency-log.md`](./program-management/dependency-log.md) to see how a program-breaking crisis was handled. Then [`/program-management/scope-change-log.md`](./program-management/scope-change-log.md) to see how "no" was said professionally.

**Engineering Leader:** Start with [`/docs/01-architecture.md`](./docs/01-architecture.md) and the [`/docs/adr/`](./docs/adr/) folder to see how technical decisions were made with real options documented.

**Product Leader:** [`/docs/04-technical-tradeoffs.md`](./docs/04-technical-tradeoffs.md) shows three moments where engineering and product had different views — and how each was resolved with data.

**TPM / PM:** [`/docs/03-execution-workflow.md`](./docs/03-execution-workflow.md) for the operating model. [`/program-management/risk-register.md`](./program-management/risk-register.md) for risk methodology. [`/ai-operations/prompt-library.md`](./ai-operations/prompt-library.md) for AI-assisted operations.

---

## Program Highlights

**Handled a program-breaking crisis:** Lumen Technologies delayed DID porting 90 days. $150,700 exposure. Response: Option B dual-carrier strategy approved in 7 days, $4,200 cost, zero customer impact. → [`dependency-log.md`](./program-management/dependency-log.md)

**Protected 22+ weeks of schedule:** 6 of 8 change requests deferred with documented rationale and Phase 2 charters — not just "no." → [`scope-change-log.md`](./program-management/scope-change-log.md)

**Made hard trade-offs stick:** Real-time CNAM deferred, multi-region deferred, IVR rebuilt from scratch. Each decision has a named owner and written consequences. → [`04-technical-tradeoffs.md`](./docs/04-technical-tradeoffs.md)

**Uses AI to operate faster:** 6 production Claude prompts. Status reports drafted in 8 seconds, edited in 10 minutes. → [`prompt-library.md`](./ai-operations/prompt-library.md)

---

*All data, names, and scenarios are realistic simulations based on real-world telecom migration patterns. Built as a TPM portfolio project.*
