# Section 7 — Scope Creep Control

**Document:** `program-management/scope-change-log.md`
**Program:** Project NORTH STAR
**Version:** 1.3
**Last Updated:** 2024-07-12
**Owner:** [TPM Name]
**Review Cadence:** Every change request reviewed within 48 hours of submission

---

## 7.1 Scope Governance Model

### The Problem with Scope Creep

Scope creep doesn't announce itself. It arrives as reasonable requests from well-intentioned stakeholders. The TPM's job is not to say no to everything — it's to make the cost of every addition visible before it is accepted.

### Change Request Process

Every scope change request on Project NORTH STAR follows this process:

```
1. REQUEST SUBMITTED
   Stakeholder submits change request (CR) via Jira or email
   TPM acknowledges within 24 hours
        │
        ▼
2. IMPACT ASSESSMENT (48 hours)
   TPM produces written impact analysis:
   - Schedule delta
   - Budget delta
   - Resource impact
   - Dependencies affected
        │
        ▼
3. DECISION GATE
   Is it in the approved program charter? ──► YES ──► Accept, add to backlog
        │
        │ NO
        ▼
   Can it be phased? ──► YES ──► Defer to Phase 2 with its own charter
        │
        │ NO
        ▼
   Does it block current go-live? ──► YES ──► Escalate to sponsor for approval
        │
        │ NO
        ▼
   DEFER — log in parking lot, create Phase 2 ticket
        │
        ▼
4. DECISION COMMUNICATED
   Stakeholder notified within 48 hours of assessment
   Decision logged in Change Register
```

### Scope Change Decision Criteria

| Criterion | Threshold | Action |
|---|---|---|
| In approved charter? | Yes | Accept |
| Schedule impact | >2 weeks | Requires executive sponsor approval |
| Budget impact | >$10,000 | Requires executive sponsor approval |
| Blocks current go-live? | Yes | Escalate immediately |
| Can be phased post-MVP? | Yes | Defer to Phase 2 |
| Valid but out of scope | Any | Parking lot — Phase 2 charter |

---

## 7.2 Change Request Register

| CR ID | Submitted By | Request | Date | Schedule Impact | Budget Impact | Decision | Outcome |
|---|---|---|---|---|---|---|---|
| CR-001 | Marcus Okafor | Add Salesforce CTI screen pop to agent desktop at Wave 3 | 2024-02-15 | +8 weeks | +$45,000 | Deferred | Phase 2 charter created |
| CR-002 | Diane Torres (CX) | Real-time agent dashboard with live call metrics | 2024-03-08 | +4 weeks | +$18,000 | Deferred | Phase 2 — Q4 2024 |
| CR-003 | Priya Nair | Add multi-region failover at Wave 1 | 2024-03-22 | +6 weeks | +$18K/mo ongoing | Deferred | ADR-004 — FY2025 budget |
| CR-004 | Sarah Chen | Accelerate Wave 3 to recover Lumen delay | 2024-07-05 | -4 weeks (target) | +$4,200 | Accepted | Option B executed |
| CR-005 | Marcus Okafor | Add IVR sentiment analysis (Contact Lens) at Wave 3 | 2024-05-20 | +3 weeks | +$8,000 | Deferred | Phase 2 — Q1 2025 |
| CR-006 | Dana Kim | Expand regression suite to include performance testing | 2024-06-10 | +2 weeks | +$5,000 | Partially accepted | Load testing added, perf suite deferred |
| **CR-007** | **Marcus Okafor** | **Real-time call analytics dashboard** | **2024-06-18** | **+6-8 weeks** | **+$38,000** | **Deferred** | **Phase 2 — full scenario below** |
| CR-008 | James Whitfield | Add STIR/SHAKEN call authentication at Wave 3 | 2024-07-02 | +5 weeks | +$22,000 | Deferred | Phase 2 — Q1 2025 |

---

## 7.3 Full Scenario — CR-007: Real-Time Analytics Dashboard

This is the most detailed example of scope creep management on Project NORTH STAR. It demonstrates how a valid, well-intentioned request was handled without damaging the stakeholder relationship or the program timeline.

### The Request

**Submitted by:** Marcus Okafor (Director of Product)
**Date:** June 18, 2024
**Method:** Slack message to TPM, followed by email

> *"Hey — the CX team has been asking for a real-time call analytics dashboard. Live call volume, queue depth, agent availability, and abandonment rate. They need this for Wave 3 go-live. I know it's more work but I think it's important. Can we scope it? I'm thinking it's a few weeks of work."*

### Step 1 — Acknowledge and Set Expectations

TPM responded within 4 hours:

> *"Got it Marcus — makes sense as a need. I'll put together an impact assessment by Thursday EOD before we make any decisions. The estimate of 'a few weeks' might be in the right ballpark but let me have engineering look at it properly. I'll send you something concrete Thursday."*

---

### Step 2 — Impact Assessment

**Engineering estimate (Priya Nair, June 20, 2024):**

To build a real-time call analytics dashboard with the requested features requires:
- Kinesis Data Streams configuration for real-time CTR event streaming
- Lambda processing function for event transformation
- Amazon QuickSight dashboard with live dataset
- IAM roles, S3 data lake for historical data
- Testing and documentation

**Engineering estimate: 6–8 weeks. Not 'a few weeks.'**

| Impact Item | Detail |
|---|---|
| Engineering effort | 6–8 weeks (2 engineers) |
| Schedule impact | Wave 3 slips 6–8 weeks — from Dec 15 to Feb 2025 |
| Budget impact | +$38,000 (infrastructure + engineering time) |
| Dependencies added | Kinesis + QuickSight — 2 new AWS services not in current IaC scope |
| Risk introduced | New services in scope = new drift risk, new security surface |

---

### Step 3 — Decision Framework Applied

**Question 1: Is it in the approved program charter?**
No. The program charter approved in January 2024 scopes to voice infrastructure migration. Analytics and reporting capabilities are explicitly listed as Phase 2 work.

**Question 2: Does it block Wave 3 go-live?**
No. CX can use AWS Connect's native reporting for queue metrics and call volume during the 90-day period after go-live. The data exists — it just won't be in a custom dashboard.

**Question 3: Can it be phased — delivered post-Wave 3 without blocking go-live?**
Yes. The CTR event stream (Kinesis) is already being built as part of Wave 2 (Lambda CTR Processor). The dashboard layer sits on top of data that will exist. This is a presentation layer, not a data infrastructure problem.

**Question 4: What is the business risk of deferring 90 days?**
Low. CX team currently has no real-time analytics on the legacy Cisco UCM system. Any dashboard — even one delivered 90 days after Wave 3 go-live — is an improvement over the current state.

**Decision: CR-007 DEFERRED from program scope.**

---

### Step 4 — Communicate the Decision

The following was sent to Marcus Okafor on June 21, 2024:

---

**From:** [TPM Name]
**To:** Marcus Okafor
**Date:** June 21, 2024
**Subject:** CR-007 Real-Time Analytics Dashboard — Impact Assessment + Decision

---

Marcus,

Thanks for raising CR-007. Here's the impact assessment I promised.

**Engineering reality check:** This is a 6–8 week effort, not a few weeks. It requires Kinesis Data Streams, a Lambda processing layer, QuickSight dashboard configuration, and a new S3 data lake. Priya has the full estimate attached.

**What this means for the program:** If we accept this into Wave 3 scope, go-live moves from December 15 to February 2025. That's an additional $57,000 in legacy PBX costs we've already been fighting to eliminate.

**My recommendation: Defer CR-007 to Phase 2.**

Here's why this is actually good news for you: The CTR event streaming infrastructure (Kinesis) is already being built in Wave 2. By the time we're ready to build the dashboard layer in Q4, the hard part will already be done. Phase 2 dashboard will be faster and cheaper because of the groundwork we're laying now.

**What CX gets at Wave 3 go-live:** AWS Connect has native queue metrics, call volume, and agent availability reporting built in. It won't be a custom dashboard but it covers the core use case for the first 90 days.

I've created a Phase 2 charter stub for CR-007 and assigned it to Priya for discovery in Q4 2024. It's on the roadmap — just not in this program.

Wave 3 schedule is protected. No budget impact.

Let me know if you want to sync on the Phase 2 scope before I finalize the charter stub.

— [TPM Name]

---

### Step 5 — Outcome

| Item | Result |
|---|---|
| Wave 3 schedule | Protected — no change |
| Budget | No impact |
| Stakeholder relationship | Preserved — Marcus acknowledged the deferral was "the right call" |
| CR-007 Phase 2 charter | Created — assigned to Priya for Q4 2024 discovery |
| CX interim solution | AWS Connect native reporting — briefed to Diane Torres |

**Key lesson:** The decision was not "no" — it was "not now, and here's exactly when and how." That framing is what keeps stakeholder relationships intact while protecting the program.

---

## 7.4 Scope Change Summary

| Outcome | Count | Notes |
|---|---|---|
| Accepted | 1 | CR-004 — Option B carrier acceleration |
| Partially accepted | 1 | CR-006 — load testing accepted, perf suite deferred |
| Deferred to Phase 2 | 6 | All with Phase 2 charter or parking lot ticket created |
| Rejected outright | 0 | Every valid idea has a home — none were discarded |

**Total scope protection:** 6 change requests deferred, protecting an estimated **22+ weeks** of schedule and **$131,000+** in additional program costs.

---

*Back: [Technical Debt Register](./technical-debt-register.md) | Next: [AI Operations →](../ai-operations/prompt-library.md)*
