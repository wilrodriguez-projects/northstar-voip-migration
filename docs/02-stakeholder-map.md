# Section 1 — Stakeholder Map & Communications Plan

**Document:** `docs/02-stakeholder-map.md`  
**Program:** Project NORTH STAR  
**Version:** 1.2  
**Last Updated:** 2024-07-12

---

## Stakeholder Engagement Matrix

| Stakeholder | Influence | Interest | Strategy | Preferred Channel | Cadence |
|---|---|---|---|---|---|
| Sarah Chen (VP Eng) | High | High | Manage Closely | Weekly async report + sync on escalations | Weekly |
| Marcus Okafor (Product) | High | Medium | Keep Satisfied | Bi-weekly sync + Slack for blockers | Bi-weekly |
| Priya Nair (Platform) | Medium | High | Keep Informed | Daily standup + Jira | Daily |
| James Whitfield (Telecom) | Medium | High | Keep Informed | Weekly sync + email for carrier issues | Weekly |
| Dana Kim (QA) | Medium | High | Keep Informed | Sprint ceremonies + Slack | Per sprint |
| Tom Reyes (Finance) | Low | Medium | Monitor | Monthly budget report | Monthly |
| Diane Torres (CX) | Medium | Medium | Keep Satisfied | Pre-wave briefings + UAT sign-off | Per wave |
| Raj Patel (Security) | Low | Low | Monitor | Security review gates | Per milestone |

---

## Communications Plan

### Recurring Cadence

| Meeting | Participants | Frequency | Purpose | Owner |
|---|---|---|---|---|
| Program Standup | Priya, James, Dana, TPM | Daily (15 min) | Blocker surfacing, daily progress | TPM |
| Sprint Review | All engineering, Marcus | Bi-weekly | Demo completed stories, accept/reject | TPM |
| Sprint Retrospect | Engineering team | Bi-weekly | Process improvement | TPM |
| Stakeholder Sync | Sarah, Marcus, TPM | Weekly (30 min) | RAG status, decisions needed, risks | TPM |
| Carrier Coordination | James, Lumen/Bandwidth contacts, TPM | Weekly | Porting status, LOA issues | James W. |
| Executive Status Report | Sarah Chen (async) | Weekly | Written RAG report via email | TPM |
| Finance Review | Tom Reyes, TPM | Monthly | Budget burn, cost-to-complete | TPM |
| Wave Go/No-Go | All stakeholders | Per wave | Gate review before cutover | TPM |

### Escalation Path

```
Issue Identified (TPM)
        │
        ▼
 Can it be resolved     YES ──► Resolve + document in decision register
 within the team?
        │ NO
        ▼
 Does it affect         YES ──► Immediate Slack/call to Sarah Chen
 schedule > 2 wks              Follow up with written impact analysis
 or budget > $10K?             within 24 hours
        │ NO
        ▼
 Raise in weekly               Document in risk register
 stakeholder sync              Propose mitigation options
```

---

## Stakeholder Communication Templates

### Wave Go-Live Notification (T-5 days)

```
Subject: [NORTH STAR] Wave [N] Go-Live — [Date] — Action Required

Hi team,

We are on track for the Wave [N] cutover on [DATE] at [TIME] ET.

WHAT'S HAPPENING:
[2-sentence plain language description of what is being cut over]

IMPACT TO YOUR TEAM:
[Specific, honest statement of any disruption window]

ROLLBACK PLAN:
If issues are detected, we will roll back to [prior state] within [X] minutes.
The rollback decision gate is [TIME] ET on [DATE].

CONTACTS DURING CUTOVER:
- Technical issues: Priya Nair (Slack: @priya.nair)
- Business issues: [TPM Name] (Slack: @[handle])
- Emergency escalation: Sarah Chen

Please confirm receipt of this notice by [DATE].
```

### Risk Escalation Template

```
Subject: [NORTH STAR] Risk Escalation — [Risk Title] — [Action Required/FYI]

Sarah,

I need to flag a risk that requires your awareness [and a decision by DATE].

RISK: [One sentence description]
IMPACT: [Schedule / Budget / Quality impact — be specific]
CURRENT STATUS: [What is being done right now]
OPTIONS: 
  A) [Option with tradeoffs]
  B) [Option with tradeoffs]
RECOMMENDATION: [Your recommendation + rationale]
DECISION NEEDED BY: [DATE]

I will follow up with a full impact analysis document by [DATE].
```

---

*Back: [Program Overview](./00-program-overview.md) | Next: [Execution Workflow →](./03-execution-workflow.md)*
