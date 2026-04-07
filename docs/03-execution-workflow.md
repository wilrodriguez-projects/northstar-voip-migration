# Section 9 — Execution Workflow

**Document:** `docs/03-execution-workflow.md`
**Program:** Project NORTH STAR
**Version:** 1.1
**Last Updated:** 2024-07-12
**Owner:** [TPM Name]

---

## 9.1 Program Operating Cadence

### Weekly Rhythm

| Day | Activity | Duration | Participants | Output |
|---|---|---|---|---|
| Monday | Dependency review — check Jira filter for overdue deps | 15 min | TPM solo | Updated dependency log |
| Tuesday | Engineering standup | 15 min | Priya, Dana, TPM | Blocker list |
| Wednesday | Carrier sync (James W. + Lumen/Bandwidth) | 30 min | James W., TPM | LOA status update |
| Thursday | Sprint ceremonies (review + retro, bi-weekly) | 2 hrs | Full team | Completed stories, retro actions |
| Friday | Risk review + weekly status report | 45 min | TPM solo → Sarah Chen async | Status report sent by 4pm ET |

### Sprint Structure (2-week sprints)

```
SPRINT START (Monday)
├── Sprint planning (2 hrs) — TPM + full team + Marcus Okafor
│   ├── Review carry-over from last sprint
│   ├── Capacity check (vacations, risk-driven reductions)
│   ├── Backlog grooming — top stories pulled in
│   └── Debt allocation confirmed (15% minimum)
│
SPRINT MID (end of Week 1)
├── Mid-sprint check (15 min) — TPM + leads
│   ├── Any blockers surfaced since planning?
│   ├── Dependency status — anything overdue?
│   └── Velocity on track?
│
SPRINT END (Friday of Week 2)
├── Sprint review (1 hr) — full team + Marcus
│   ├── Demo completed stories
│   └── Accept/reject against definition of done
├── Sprint retrospect (45 min) — engineering team
│   ├── What went well / what didn't / action items
│   └── Risk items surfaced → added to risk register
└── Status report sent → Sarah Chen
```

---

## 9.2 Jira Board Structure

### Board Layout

```
NORTH STAR PROGRAM BOARD
│
├── EPICS (program-level)
│   ├── EPIC-001: Wave 1 — Dev/Staging Migration        [CLOSED]
│   ├── EPIC-002: Wave 1b — ALB + Blue/Green Cutover    [CLOSED]
│   ├── EPIC-003: Wave 2 — Production DID Porting       [ACTIVE]
│   ├── EPIC-004: Wave 3 — Toll-Free + IVR Cutover      [PLANNED]
│   ├── EPIC-005: Technical Debt Remediation            [ACTIVE]
│   └── EPIC-006: Phase 3 — PBX Decommission            [PLANNED]
│
├── COLUMNS
│   ├── Backlog
│   ├── Ready (groomed, estimated, acceptance criteria written)
│   ├── In Progress (WIP limit: 3 per engineer)
│   ├── In Review (PR open, awaiting review)
│   ├── Blocked (tagged with blocker reason)
│   └── Done (accepted by product owner)
│
└── LABELS
    ├── wave-2, wave-3 (wave assignment)
    ├── p1-debt, p2-debt (debt items)
    ├── dependency (has external dependency)
    ├── risk-item (linked to risk register)
    └── decision-required (needs stakeholder sign-off)
```

### Automated Jira Queries (saved filters)

| Filter Name | Query | Used For |
|---|---|---|
| Overdue Dependencies | `label = dependency AND dueDate < now() AND status != Done` | Monday dependency review |
| P1 Debt Open | `label = p1-debt AND status != Done` | Sprint planning — must include |
| Blocked This Sprint | `sprint in openSprints() AND status = Blocked` | Daily standup |
| Decision Required | `label = decision-required AND status != Done` | Weekly stakeholder sync |
| Wave 3 Readiness | `fixVersion = "Wave-3" AND status != Done` | Wave 3 go/no-go tracking |

---

## 9.3 Decision-Making Flow

### How Decisions Are Made

Not all decisions are equal. The framework below determines who owns what and how fast it moves.

```
DECISION RAISED
      │
      ▼
Is it reversible within     YES ──► TPM decides, documents in decision register
1 sprint and <$5K?                  Informs stakeholders async
      │ NO
      ▼
Does it affect schedule     YES ──► TPM + Engineering Lead decide jointly
>1 week or budget                   Present options to Sarah Chen for awareness
>$10K?                              Requires decision register entry
      │ NO
      ▼
Does it affect program      YES ──► Present to Sarah Chen with written options
go-live or budget                   Decision within 24 hours
>$50K?                              Logged in Decision Register with rationale
      │ NO
      ▼
Does it require board       YES ──► Sarah Chen escalates to CFO/CEO
approval (>$100K or                 TPM prepares business case
strategic direction)?               TPM not the decision maker
```

### Decision Register Format

Every program decision is logged with:

| Field | Content |
|---|---|
| ID | DR-### (sequential) |
| Decision | One sentence — what was decided |
| Context | Why this decision was needed (2-3 sentences) |
| Options considered | All options evaluated — not just the chosen one |
| Decision maker | Named individual — not "the team" |
| Rationale | Why this option vs. the others |
| Date | When the decision was made |
| Consequences | What we gain and what we trade off |
| Review date | When to revisit if assumptions change |

---

## 9.4 Status Reporting Flow

### Report Flow

```
FRIDAY 2pm ET
    │
    ▼
TPM pulls sprint data from Jira
(velocity, completed stories, blockers, carry-over)
    │
    ▼
Claude Prompt 1 → first draft status report (8 seconds)
    │
    ▼
TPM edits: verify facts, add color, sharpen language (10 min)
    │
    ▼
FRIDAY 4pm ET
Status report emailed to Sarah Chen (async)
Copied to: Marcus Okafor, Tom Reyes, Priya Nair, James Whitfield
    │
    ▼
MONDAY — any responses/decisions addressed in weekly sync
```

### RAG Status Criteria

| Status | Criteria |
|---|---|
| 🟢 GREEN | On schedule ±1 week. No critical or high risks without active mitigation. Budget within 5%. |
| 🟡 AMBER | Schedule slip 1–3 weeks OR 1 critical risk without full mitigation OR budget 5–10% over. |
| 🔴 RED | Schedule slip >3 weeks OR 2+ critical risks OR budget >10% over OR go-live threatened. |

**Rule:** Status never moves from RED to GREEN in one week. It must pass through AMBER first. This prevents false optimism and maintains trust with stakeholders.

---

## 9.5 Wave Go/No-Go Gate Process

Before each wave cutover, a formal Go/No-Go gate is held. No wave proceeds without explicit sign-off from all required stakeholders.

### Gate Criteria — Wave 3 (example)

| Gate | Criteria | Owner | Status |
|---|---|---|---|
| Technical | IaC coverage ≥90% for all Connect resources | Priya N. | ⬜ Pending |
| Technical | All 34 IVR flows pass regression testing | Dana K. | ⬜ Pending |
| Technical | Rollback tested in staging within 48 hours of cutover | Priya N. | ⬜ Pending |
| Quality | IVR test coverage ≥80% | Dana K. | 🟡 61% now |
| Vendor | All toll-free numbers ported and confirmed | James W. | ⬜ Pending |
| Vendor | CNAM static CSV loaded and validated | James W. | ⬜ Pending |
| Monitoring | CloudWatch dashboards and PagerDuty alerts active | Priya N. | ⬜ Pending |
| Business | CX team briefed on cutover window and impact | Diane T. | ⬜ Pending |
| Business | Support team trained on new call flows | Diane T. | ⬜ Pending |
| Sign-off | Sarah Chen (VP Engineering) written approval | Sarah C. | ⬜ Pending |
| Sign-off | Marcus Okafor (Product) acceptance | Marcus O. | ⬜ Pending |

**Go/No-Go meeting:** Scheduled T-5 days from cutover. All gate items must be GREEN before the meeting proceeds to a Go decision.

---

## 9.6 Escalation Protocol

### When to Escalate (and How Fast)

| Situation | Escalate To | Timeline | Method |
|---|---|---|---|
| Dependency slip >2 weeks | Sarah Chen | Within 24 hours | Written impact analysis + options |
| Budget overrun >10% | Sarah Chen + Tom Reyes | Within 24 hours | Written with cost-to-complete update |
| Critical risk (score 9) with no mitigation | Sarah Chen | Same day | Slack + follow-up email |
| Key engineer departure | Sarah Chen + HR | Immediately | Phone + email |
| Production outage during cutover | Sarah Chen + Priya N. | Immediately | Phone (not Slack) |
| Vendor contractual failure | Sarah Chen + Legal | Same day | Written with contract reference |

**Golden rule:** Sarah Chen should never hear about a problem first from someone other than the TPM. If a stakeholder raises an issue the TPM hasn't escalated yet, that is a program management failure.

---

*Back: [AI Operations](../ai-operations/prompt-library.md) | Next: [Final Polish → README update]*
