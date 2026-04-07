# Section 8 — AI Integration & Prompt Library

**Document:** `ai-operations/prompt-library.md`
**Program:** Project NORTH STAR
**Version:** 1.2
**Last Updated:** 2024-07-12
**Owner:** [TPM Name]
**Purpose:** Document how Claude (Anthropic) is used to augment program operations — not replace judgment, but accelerate it

---

## 8.1 AI Integration Philosophy

AI assistance on Project NORTH STAR is used for three things:

1. **Acceleration** — First drafts of status reports, risk analyses, and stakeholder communications that the TPM then edits and owns
2. **Pattern recognition** — Surfacing risks and blockers that may be buried in dense sprint data or dependency logs
3. **Consistency** — Ensuring every status report, risk entry, and decision record follows the same structure and quality bar

**What AI does NOT do on this program:**
- Make decisions — every output is reviewed and approved by the TPM
- Replace stakeholder relationships — AI drafts communications, humans send them
- Substitute for engineering judgment — architecture decisions are owned by engineers

---

## 8.2 Prompt Library

### Prompt 1 — Weekly Status Report Generator

**Use case:** Every Friday, the TPM feeds sprint velocity, completed stories, carry-over items, and open risks into this prompt. Claude produces a first-draft status report in under 60 seconds. The TPM spends 10 minutes editing vs. 45 minutes writing from scratch.

**Prompt:**
```
You are a senior Technical Program Manager writing a weekly status report
for an executive audience (VP Engineering and Director of Product).

Given the data below, produce a concise status report in this exact format:

## Overall Status: [RED / AMBER / GREEN]
One sentence justification. Be specific — name the risk or blocker.

## Accomplishments This Week
- [Max 3 bullets. Each must reference a specific story, milestone, or deliverable.]

## Active Risks Requiring Attention
| Risk ID | Description | Owner | Mitigation ETA |
[Top 2 risks only. If severity is HIGH or CRITICAL, flag it explicitly.]

## Decisions Needed from Leadership
| Decision | Options | Recommendation | Needed By |
[Only include if a decision is genuinely needed. Leave blank if none.]

## Next Week Focus
- [3 bullets. Specific deliverables with owners.]

Rules:
- Be direct. No fluff. No passive voice.
- Assume the reader has 90 seconds.
- Never say "we are working on" — say what will be done and by when.
- If status is RED, the first sentence must say why.

---
SPRINT DATA:
Velocity: {sprint_velocity} of {target_velocity} points
Completed: {completed_stories}
Carry-over: {carryover_stories}
Blockers: {open_blockers}

RISK REGISTER (top risks):
{top_risks_csv}
```

**Sample output trigger:** "Sprint 14: 34/42 pts. Completed: DID webhook handler, Secrets Manager IAM policy. Carry-over: SCP policy (blocked on AWS support ticket). Blockers: Lumen LOA still pending. Top risks: R-001 (carrier delay, mitigating), R-002 (IaC drift, in progress)."

---

### Prompt 2 — Risk Spotter

**Use case:** After each sprint review or stakeholder meeting, the TPM pastes the meeting notes or status update into this prompt. Claude identifies risks that may have been missed or under-weighted.

**Prompt:**
```
You are a risk analyst for a complex cloud infrastructure program.

Review the program update below and identify any risks the TPM may have
missed or under-weighted. For each risk:

1. Assign Likelihood (High / Medium / Low) and Impact (High / Medium / Low)
2. Identify the root cause category:
   [Vendor | Technical | Staffing | Scope | Compliance | Financial]
3. Suggest ONE concrete mitigation action (not a vague recommendation)
4. Flag if this risk has a dependency on another open risk

Focus especially on:
- Upstream dependencies mentioned casually
- Single points of failure (one person, one vendor, one system)
- Unowned action items
- Timeline compression risks
- Anything described as "we'll figure it out later"

Output format: Markdown table with columns:
Risk | Category | Likelihood | Impact | Suggested Mitigation | Dependencies

---
PROGRAM UPDATE:
{paste_status_update_or_meeting_notes_here}
```

---

### Prompt 3 — Dependency Slip Analyzer

**Use case:** When a critical dependency slips, the TPM uses this prompt to rapidly produce an impact analysis and stakeholder communication draft. Used directly for the Lumen carrier delay (DEP-001) scenario.

**Prompt:**
```
A critical upstream dependency has slipped by {N} weeks.

Dependency: {dependency_description}
Blocked workstreams: {blocked_workstreams}
Current program status: {rag_status}
Executive sponsor: {sponsor_name}

Produce the following:

1. PLAIN LANGUAGE IMPACT ANALYSIS (3-4 sentences)
   Written for a non-technical executive. No jargon.
   Include: what slipped, what it blocks, financial exposure, what is NOT affected.

2. THREE REPLANNING OPTIONS
   For each option: approach, schedule recovery, cost delta, risk level, dependency.
   Format as a comparison table.

3. RECOMMENDATION with rationale (2-3 sentences)

4. DRAFT STAKEHOLDER EMAIL
   Subject line + 4-paragraph body:
   Para 1: What happened (factual, no spin)
   Para 2: What this means for the program (specific dates and costs)
   Para 3: Your recommendation and why
   Para 4: Ask — specific meeting request or decision needed by date

Tone: Calm, factual, solution-oriented. Do not bury the lead.
Never say "unfortunately" or "I'm sorry to report."
```

---

### Prompt 4 — Technical Debt Triage

**Use case:** Quarterly Debt Day. The TPM and engineering lead paste the current debt backlog into this prompt to get a ranked priority list and sprint assignment recommendations.

**Prompt:**
```
Review the technical debt backlog below for a VoIP cloud migration program.

For each item, score it on:
- Security Risk (1-5): exposure level if unresolved
  5 = active security finding or SOC2 blocker
  3 = potential exposure, no active exploit
  1 = minimal security relevance
- Delivery Risk (1-5): likelihood of blocking a release milestone
  5 = will block next wave go-live if unresolved
  3 = creates operational risk but not a hard blocker
  1 = cosmetic or low-priority improvement
- Remediation Effort (1-5, where 5 = highest effort)

Compute Priority Score = (Security Risk × 2) + (Delivery Risk × 2) - Effort

Output:
1. Ranked table: ID, Item, Sec, Del, Eff, Score, Priority (P1/P2/P3)
2. Recommended sprint assignments for the top 5 items
3. Any items that should be ESCALATED to the executive sponsor (security findings)
4. Any items that can be CLOSED without fixing (no longer relevant)

---
DEBT BACKLOG:
{paste_debt_items_here}
```

---

### Prompt 5 — Scope Change Impact Assessment

**Use case:** When a stakeholder submits a change request, this prompt produces the 48-hour impact assessment the TPM is required to deliver. Used directly for CR-007.

**Prompt:**
```
A product stakeholder has requested the following addition to an in-flight program:

REQUEST: "{scope_change_request}"
SUBMITTED BY: {stakeholder_name} ({role})
PROGRAM STATUS: Sprint {N} of {total}, {rag_status}
CURRENT WAVE: {wave_description}

Evaluate this request using the following framework:

1. IS IT IN SCOPE? (Yes / No / Partially)
   Reference the program charter if known.

2. ENGINEERING EFFORT ESTIMATE
   Provide a range. Call out if the stakeholder's estimate seems off.

3. SCHEDULE IMPACT
   If accepted now vs. deferred 90 days.

4. CAN IT BE PHASED?
   Is there a v1 that ships now with full delivery later?
   What would CX/users get in the interim?

5. BUSINESS RISK OF DEFERRAL
   What does the business lose by waiting?
   Is the risk immediate or theoretical?

6. RECOMMENDATION
   Accept / Defer / Reject with one-paragraph rationale.

Output: A one-page decision brief suitable for a 15-minute stakeholder review.
Format it so a non-engineer can understand the trade-off immediately.
```

---

### Prompt 6 — Pre-Wave Go/No-Go Checklist Generator

**Use case:** 2 weeks before each wave cutover, this prompt produces a customized go/no-go checklist based on the current program state.

**Prompt:**
```
Generate a Wave {N} go/no-go checklist for a VoIP migration program.

Wave description: {wave_description}
Key dependencies: {dependencies}
Known risks: {open_risks}
Rollback window: {rollback_window}
Stakeholders required for sign-off: {signoff_list}

Produce a checklist with three sections:

PRE-CUTOVER (T-5 days to T-1 hour)
- Technical gates (IaC validated, monitoring active, rollback tested)
- Business gates (stakeholder notifications sent, support team briefed)
- Vendor gates (carrier confirmation, SIP trunk test calls completed)

DURING CUTOVER (T-0 to T+2 hours)
- Go/no-go decision point criteria
- Escalation contacts and phone numbers
- Rollback trigger conditions (specific, measurable — not vague)

POST-CUTOVER (T+2 hours to T+24 hours)
- Validation checks
- Monitoring thresholds
- Stakeholder notification of successful completion

Each item must have: checkbox, owner, time gate, and pass/fail criteria.
No vague items like "confirm everything is working."
```

---

## 8.3 Sample AI Output — Status Report (Sprint 14)

The following is an actual Claude-generated status report draft for Sprint 14, produced using Prompt 1 above, before TPM editing.

---

**Overall Status: 🟡 AMBER**
Wave 2 DID porting remains on revised schedule (Sep 30) following Lumen carrier delay mitigation, but R-002 (IaC drift) is now overdue for remediation and Sprint 14 velocity (34 pts) is below target (42 pts) for the second consecutive sprint.

**Accomplishments This Week**
- DID webhook handler deployed to production — port status events now flowing to DynamoDB in real time
- Secrets Manager IAM policy scoped and validated — TD-001 remediation 60% complete
- Bandwidth.com FOC confirmation received for Aug 15 port batch — Wave 2 recovery on track

**Active Risks Requiring Attention**

| Risk ID | Description | Owner | Mitigation ETA |
|---|---|---|---|
| R-002 | IaC drift — Terraform plan audit in progress, SCP policy blocked on AWS support ticket | Priya N. | Sprint 15 |
| R-006 | SIP engineer attrition — comp review pending HR response | Sarah C. | Jul 19 |

**Decisions Needed from Leadership**

| Decision | Options | Recommendation | Needed By |
|---|---|---|---|
| Sprint 15 capacity | Feature work vs. debt sprint (TD-001, TD-003) | Debt sprint — 43% velocity to P1 items | Jul 15 (sprint planning) |

**Next Week Focus**
- Priya N.: Complete Terraform state S3 migration (TD-003) — Sprint 15 target
- James W.: Confirm Bandwidth.com Aug 15 port batch details with carrier ops team
- TPM: Sprint 15 planning with Marcus Okafor — align on reduced feature capacity

---

*Back: [Scope Change Log](../program-management/scope-change-log.md) | Next: [Execution Workflow →](../docs/03-execution-workflow.md)*
