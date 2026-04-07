# 🚀 Project NORTH STAR Carrier Grade VoIP Migration to AWS Connect

> **Program Type:** Cloud Infrastructure / Telecom Migration  
> **Duration:** January 2024 – November 2024 (11 months)  
> **Budget:** $2.4M allocated · $2.1M burned at Sprint 14  
> **Scope:** 6,200 DIDs + Toll-Free Numbers + IVR Cutover + Legacy PBX Decommission  
> **Status at Sprint 14:** 🟡 AT RISK — carrier dependency delay mitigated, on revised schedule

---

## What This Repository Demonstrates

This is a fully documented Technical Program Management portfolio project simulating a real-world carrier-grade VoIP migration. Every artifact, decision, and report reflects how a senior TPM operates in a complex technical environment.

| Competency | Where to Find It |
|---|---|
| Risk identification & mitigation | [`/program-management/risk-register.md`](./program-management/risk-register.md) |
| Upstream dependency delays | [`/program-management/dependency-log.md`](./program-management/dependency-log.md) |
| Technical debt management | [`/program-management/technical-debt-register.md`](./program-management/technical-debt-register.md) |
| Scope creep control | [`/program-management/scope-change-log.md`](./program-management/scope-change-log.md) |
| Eng vs. Product trade-offs | [`/docs/04-technical-tradeoffs.md`](./docs/04-technical-tradeoffs.md) |
| System architecture | [`/docs/01-architecture.md`](./docs/01-architecture.md) |
| Stakeholder communication | [`/program-management/dependency-log.md#stakeholder-comms`](./program-management/dependency-log.md) |
| AI-assisted program operations | [`/ai-operations/prompt-library.md`](./ai-operations/prompt-library.md) |
| Weekly status reporting | [`/status-reports/`](./status-reports/) |
| Architecture decisions | [`/docs/adr/`](./docs/adr/) |

---

## Program Summary

**The Problem:** Our organization ran a Cisco Unified Communications Manager (UCM) PBX cluster that was end-of-support in Q4 2024. The system managed 6,200 Direct Inward Dial (DID) numbers, 48 toll-free numbers, and a 34-flow IVR serving ~12,000 inbound calls per day. Annual maintenance cost: **$340,000/year**. Any failure had direct customer impact with no automated failover.

**The Mandate:** Migrate all voice infrastructure to AWS Connect before the Cisco support contract expired, achieving parity with existing call handling while enabling future ML-based routing capabilities.

**The Outcome (Sprint 14 of 22):** 3,842 of 6,200 DIDs successfully ported. Staging and dev environments fully migrated. Wave 2 production porting underway. One critical dependency slip (carrier delay, 90 days) identified early, mitigated via dual-carrier strategy, and communicated proactively to executive stakeholders.

---

## System Architecture (Overview)

```
  ┌─────────────────────────────────────────────────┐
  │              PSTN / CARRIER LAYER               │
  │        Lumen (DID Blocks) · Bandwidth (TF)      │
  └────────────────────┬────────────────────────────┘
                       │ SIP Trunks (LOA-authorized)
  ┌────────────────────▼────────────────────────────┐
  │         Twilio Elastic SIP — BYOC Interim       │
  │         (Active during Wave 2 porting)          │
  └────────────────────┬────────────────────────────┘
                       │
  ┌────────────────────▼────────────────────────────┐
  │              AWS us-east-1 (Multi-AZ)           │
  │                                                 │
  │   Route 53 Health Checks                        │
  │        │                                        │
  │        ▼                                        │
  │   AWS Connect Cluster ──── Contact Flows (IVR) │
  │        │                                        │
  │   ALB (blue/green) ◄── Lambda Functions        │
  │                              │                  │
  │              ┌───────────────┤                  │
  │              ▼               ▼                  │
  │          DynamoDB       Secrets Manager         │
  │          (DID state)    (SIP credentials)       │
  └─────────────────────────────────────────────────┘
```

> Full architecture diagram with data flows, team ownership, and constraint annotations:  
> [`/docs/01-architecture.md`](./docs/01-architecture.md)

---

## Repository Structure

```
northstar-voip-migration/
├── README.md                          ← You are here
├── docs/
│   ├── 00-program-overview.md         ← Problem statement, stakeholders, business impact
│   ├── 01-architecture.md             ← System design, components, data flows
│   ├── 02-stakeholder-map.md          ← RACI, engagement model, comms plan
│   ├── 03-execution-workflow.md       ← Jira structure, sprint cadence, decision flow
│   ├── 04-technical-tradeoffs.md      ← 3 real eng vs. product decisions with outcomes
│   └── adr/
│       ├── ADR-001-sip-trunk-vendor.md
│       ├── ADR-002-aws-connect-selection.md
│       ├── ADR-003-iac-terraform-remote-state.md
│       └── ADR-004-single-region-multiaz.md
├── program-management/
│   ├── risk-register.md               ← Full risk register, 11 risks, methodology
│   ├── dependency-log.md              ← Dependency map + carrier delay response
│   ├── scope-change-log.md            ← Change requests, impact assessments, decisions
│   ├── decision-register.md           ← All program decisions with rationale
│   └── technical-debt-register.md     ← 14 debt items, priority scores, sprint schedule
├── status-reports/
│   ├── template-weekly-status.md      ← Reusable status report template
│   ├── 2024-W27-status.md
│   └── 2024-W28-status.md
├── ai-operations/
│   ├── prompt-library.md              ← 5 production-grade Claude prompts
│   └── sample-outputs/
│       ├── risk-analysis-sample.md    ← Example AI risk output
│       └── status-report-sample.md   ← Example AI status report output
└── runbooks/
    ├── did-porting-rollback.md        ← Step-by-step DID porting failure recovery
    └── wave-cutover-checklist.md      ← Pre/during/post cutover gates
```

---

## Key Metrics (Sprint 14 Snapshot)

| Metric | Value | Target | Status |
|---|---|---|---|
| DIDs Ported | 3,842 / 6,200 | 4,200 by Sprint 14 | 🟡 Behind |
| Uptime SLA | 99.91% | 99.9% | 🟢 Met |
| Budget Burn | $2.1M / $2.4M | ≤$2.2M at S14 | 🟢 On track |
| Open High Risks | 2 | 0 | 🔴 Action needed |
| Sprint Velocity | 34 pts | 42 pts | 🟡 Degraded |
| IVR Test Coverage | 61% | 80% | 🟡 Improving |
| Legacy PBX Cost (ongoing) | $28,500/mo | Decommission Nov 2024 | ⏳ Pending |

---

## How to Navigate This Repo

**If you're a hiring manager:** Start with [`/docs/00-program-overview.md`](./docs/00-program-overview.md) for context, then [`/program-management/risk-register.md`](./program-management/risk-register.md) to see how risks were managed in practice. The [`/program-management/dependency-log.md`](./program-management/dependency-log.md) shows a real carrier delay scenario with the full response playbook.

**If you're an engineer:** Start with [`/docs/01-architecture.md`](./docs/01-architecture.md) and the ADRs in [`/docs/adr/`](./docs/adr/) to see technical decision-making.

**If you're a product leader:** [`/docs/04-technical-tradeoffs.md`](./docs/04-technical-tradeoffs.md) and [`/program-management/scope-change-log.md`](./program-management/scope-change-log.md) show how engineering and product priorities were balanced.

---

*Built as a TPM portfolio project. All data, names, and scenarios are realistic simulations based on real-world telecom migration patterns.*
