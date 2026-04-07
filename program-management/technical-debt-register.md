# Section 6 — Technical Debt Strategy

**Document:** `program-management/technical-debt-register.md`
**Program:** Project NORTH STAR
**Version:** 2.1
**Last Updated:** 2024-07-12
**Owner:** Priya Nair (Engineering) + [TPM Name] (Program)
**Review Cadence:** Quarterly Debt Day + P1 items reviewed every sprint

---

## 6.1 Technical Debt Philosophy

Technical debt on Project NORTH STAR is not treated as a separate concern from delivery — it is tracked in the same Jira board as features, scored with the same priority system, and reviewed by the same stakeholders. Debt that is invisible is debt that doesn't get paid.

### Three Types of Debt on This Program

| Type | Definition | Example on NORTH STAR |
|---|---|---|
| **Intentional** | Conscious short-cut made under time pressure with a plan to fix | SIP credentials in env vars (TD-001) — done to meet Wave 1 deadline |
| **Accidental** | Discovered after the fact — no one chose it | Terraform local state (TD-003) — prototype left in place |
| **Architectural** | Design limitations that were known but deferred | IVR contact flows not in version control (TD-004) — AWS Connect limitation |

### Debt Management Rules

1. **Every intentional debt item must be logged at creation** — not after the fact
2. **P1 items are automatically added to the next sprint backlog** — they cannot be ignored
3. **15% sprint velocity is reserved for debt** — approximately 6 of 42 story points per sprint
4. **Debt is never deferred more than 2 sprints** without executive sign-off
5. **Each debt item carries an Impact Score** = (Security Risk × 2) + (Delivery Risk × 2) − Effort

---

## 6.2 Technical Debt Register

*Last reviewed: 2024-07-12 · Sprint 14 · Total items: 14 · P1 Open: 3 · P2 Open: 5 · Resolved: 6*

| ID | Debt Item | Origin | Priority | Security Risk (1-5) | Delivery Risk (1-5) | Effort (1-5) | Impact Score | Sprint Target | Status |
|---|---|---|---|---|---|---|---|---|---|
| TD-001 | **Hardcoded SIP credentials in Lambda env vars** — should be Secrets Manager | Speed-to-ship, Wave 1 | P1 | 5 | 4 | 2 | 16 | Sprint 15 | In Progress |
| TD-002 | **No automated rollback on DID porting failure** — manual intervention only | Scope cut, Wave 1 planning | P1 | 1 | 5 | 3 | 11 | Sprint 16 | Planned |
| TD-003 | **Terraform state in local backend** — no locking, not team-accessible | Prototype left in place | P1 | 2 | 5 | 1 | 13 | Sprint 15 | In Progress |
| TD-004 | **IVR contact flows not version-controlled** — JSON export only, no IaC | AWS Connect limitation | P2 | 2 | 4 | 5 | 7 | Q1 2025 | Backlog |
| TD-005 | **No integration tests for Lambda functions** — unit tests only | Time pressure, Wave 1 | P2 | 1 | 3 | 3 | 5 | Sprint 17 | Planned |
| TD-006 | **CloudWatch alarms not standardized** — ad-hoc naming and thresholds | Incremental additions | P2 | 1 | 3 | 2 | 6 | Sprint 18 | Backlog |
| TD-007 | **Real-time CNAM deferred** — static CSV push only | Trade-off decision DR-005 | P2 | 1 | 4 | 5 | 7 | Q1 2025 | Backlog |
| TD-008 | **DynamoDB capacity on-demand only** — no provisioned capacity planning | Not modeled in Wave 1 | P2 | 1 | 2 | 2 | 4 | Sprint 19 | Backlog |
| TD-009 | **No DR runbook for AWS Connect** — recovery steps undocumented | Documentation gap | P3 | 2 | 3 | 2 | 8 | Q1 2025 | Backlog |
| TD-010 | **ALB access logs not shipped to S3** — no audit trail for traffic | Overlooked in setup | P3 | 2 | 1 | 1 | 5 | Sprint 18 | Backlog |

---

## 6.3 Debt Impact Scoring Model

**Formula: Impact Score = (Security Risk × 2) + (Delivery Risk × 2) − Effort**

| Score | Priority | Action |
|---|---|---|
| 12+ | P1 — Critical | Add to next sprint immediately. Cannot be deferred. |
| 7–11 | P2 — High | Schedule within 3 sprints. Review at Debt Day. |
| 4–6 | P3 — Medium | Quarterly review. Schedule when capacity allows. |
| 1–3 | P4 — Low | Log and monitor. No active scheduling needed. |

### Why This Formula Works

- **Security Risk × 2** — security exposure compounds over time and has external compliance consequences
- **Delivery Risk × 2** — unresolved debt that blocks a release milestone is operationally equivalent to a P1 bug
- **− Effort** — low-effort fixes get a priority boost; high-effort items are still prioritized but need more planning

---

## 6.4 P1 Debt Detail

### TD-001 — Hardcoded SIP Credentials in Lambda

**What it is:** Three Lambda functions (CNAM lookup, port webhook, CTR processor) have SIP trunk credentials stored as plaintext environment variables in the Lambda configuration. These credentials allow anyone with Lambda read access to make SIP calls on our carrier accounts.

**Why it happened:** During Wave 1, the team was under deadline pressure to get BYOC routing operational. Secrets Manager integration was descoped as a "we'll fix it after go-live" item. It was never formally logged as debt until the Wave 1 retrospect.

**Business impact:** This is a SOC 2 Type II finding. If unresolved before our Q1 2025 audit, it will appear as an open finding and could affect our certification renewal. It also creates financial exposure — credential theft could result in fraudulent SIP calls billed to our account.

**Fix:** Migrate all three Lambda functions to retrieve credentials from AWS Secrets Manager at runtime. Add IAM policy scoping to restrict Secrets Manager access to only the Lambda execution roles that need it.

**Sprint 15 target:** Priya Nair. Estimate: 3 days.

---

### TD-002 — No Automated Rollback on DID Porting Failure

**What it is:** When a DID porting operation fails mid-process (e.g., carrier rejects the port after the number has been partially released), there is no automated recovery path. The current process requires manual intervention from Priya Nair or James Whitfield, taking an average of 45 minutes to 4 hours depending on the failure type.

**Why it happened:** Automated rollback was in the original Wave 1 scope. It was descoped in Sprint 6 when the team hit a velocity shortfall and needed to cut scope to meet the Wave 1 deadline.

**Business impact:** During Wave 2, where we are porting 2,358 DIDs, a porting failure during a batch operation could leave 50–100 numbers in an indeterminate state. With manual recovery only, this creates a multi-hour customer impact window.

**Fix:** Build a Lambda-based rollback function that monitors porting webhook events and automatically triggers carrier re-submission or port reversal based on failure type. Add a DynamoDB state machine to track port status and recovery actions.

**Sprint 16 target:** Priya Nair + James Whitfield. Estimate: 5 days.

---

### TD-003 — Terraform State in Local Backend

**What it is:** The Terraform state file for all AWS Connect infrastructure is stored on Priya Nair's local machine. There is no remote backend, no state locking, and no team access. Any other engineer running `terraform apply` would create a new state file, causing AWS resource duplication or deletion.

**Why it happened:** The infrastructure was initially prototyped by one engineer. The prototype became production. The local state was never migrated.

**Business impact:** This is the single biggest operational risk in our IaC posture. If Priya's machine is unavailable (illness, departure — see R-006), no one else can safely modify infrastructure. It also means there is no CI/CD pipeline that can validate infrastructure changes.

**Fix:** Migrate to S3 remote backend with DynamoDB locking. Already designed — see ADR-003.

**Sprint 15 target:** Priya Nair. Estimate: 2 days. (Highest ROI item on the board.)

---

## 6.5 Debt Trend and Delivery Impact

| Sprint | P1 Open | P2 Open | Resolved This Sprint | Notes |
|---|---|---|---|---|
| Sprint 10 | 3 | 6 | 1 | TD-010 resolved (ALB logging) |
| Sprint 11 | 3 | 6 | 1 | TD-008 partially resolved |
| Sprint 12 | 3 | 5 | 2 | TD-005, TD-006 partially addressed |
| Sprint 13 | 3 | 5 | 0 | Wave 2 pressure — debt work paused |
| **Sprint 14** | **3** | **5** | **0** | Wave 2 continues — P1s scheduled S15/S16 |

**Key insight:** Zero debt resolved in Sprints 13 and 14 due to Wave 2 carrier delay consuming capacity. P1 items are now overdue. Sprint 15 is ring-fenced: 18 of 42 story points (43%) are dedicated to TD-001 and TD-003. This is above the standard 15% allocation and has been communicated to Marcus Okafor (Product) as a temporary reduction in feature delivery.

---

## 6.6 Debt Communication to Stakeholders

The following was sent to Marcus Okafor before Sprint 15 planning:

> **Subject: Sprint 15 — Reduced Feature Capacity (Debt Sprint)**
>
> Marcus — heads up before sprint planning. Sprint 15 will have reduced feature capacity. We're dedicating 18 of 42 points to two P1 debt items (TD-001: credentials, TD-003: Terraform state) that are now overdue and have compliance and operational consequences if we delay further.
>
> Feature points available this sprint: ~24 (vs. normal ~36).
> Expected to return to full capacity Sprint 16.
>
> I'll flag the 3 lowest-priority features in the backlog that we should consider pushing. Happy to sync before planning if you want to align on priorities.
>
> — [TPM Name]

---

*Back: [Technical Trade-offs](../docs/04-technical-tradeoffs.md) | Next: [Scope Change Log →](./scope-change-log.md)*
