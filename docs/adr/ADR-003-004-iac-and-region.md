# ADR-003 — IaC Tooling: Terraform with Remote State Backend

**Status:** Accepted (supersedes initial local-state implementation)  
**Date:** 2024-03-28 (revised from original 2024-01-15)  
**Deciders:** Priya Nair, [TPM Name]  
**Context:** TD-003 remediation — local Terraform state identified as critical debt item

---

## Context

Wave 1 was completed with Terraform state stored locally on the lead engineer's machine. This created two critical problems:

1. **No team access** — only Priya Nair could safely run `terraform apply`; any other engineer would create state conflicts
2. **No locking** — concurrent runs would corrupt state, causing resource drift or duplicate resource creation

This was flagged as P1 technical debt item TD-003 during the Wave 1 retrospect.

---

## Decision

**Migrate Terraform state to S3 backend with DynamoDB state locking.**

```hcl
terraform {
  backend "s3" {
    bucket         = "acme-northstar-tfstate"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "northstar-tf-lock"
    encrypt        = true
  }
}
```

---

## Rationale

- S3 + DynamoDB is the AWS-native, Terraform-recommended pattern for team-based state management
- DynamoDB locking prevents concurrent apply operations
- S3 versioning enables state rollback if corruption occurs
- KMS encryption at rest meets our SOC 2 data handling requirements
- Zero additional tooling cost — S3 + DynamoDB costs ~$2/month at this state file size

**Alternatives considered:**
- Terraform Cloud: $20/user/month, introduces external dependency; rejected on cost and vendor lock-in
- GitLab-managed state: Not available in current GitLab tier

---

## Consequences

- All engineers can now run `terraform plan` safely
- CI/CD pipeline can run `terraform plan` on every PR as a validation gate
- State migration from local to S3 completed Sprint 15 with zero infrastructure disruption
- Added `terraform_lock` IAM policy to restrict state table writes to CI/CD role only

---
---

# ADR-004 — Single-Region Deployment with Multi-AZ

**Status:** Accepted  
**Date:** 2024-02-02  
**Deciders:** Sarah Chen (final), Priya Nair, [TPM Name]  
**Reviewed by:** Tom Reyes (Finance), Raj Patel (Security)

---

## Context

Architecture review flagged a risk: deploying AWS Connect in a single region (us-east-1) means a full regional outage would take down all inbound call handling. The engineering team proposed active-active multi-region (us-east-1 + us-west-2) as the gold standard.

Two options were formally evaluated:

| Option | Architecture | Monthly Cost Delta | Recovery Time | Complexity |
|---|---|---|---|---|
| A — Single-region, Multi-AZ | us-east-1 only, AZ redundancy | $0 | ~4 hrs (AZ failure) | Low |
| B — Active-active multi-region | us-east-1 + us-west-2 | +$18,000/mo | ~15 min (regional failure) | High |

---

## Decision

**Selected: Option A — Single-region (us-east-1) with Multi-AZ enabled.**

Multi-region (Option B) documented as a future investment for FY2025 budget cycle.

---

## Rationale

**For Option A:**
- AWS Connect SLA is 99.99% per region — regional outage probability is extremely low
- Legacy Cisco UCM ran at 99.7% — even single-region AWS Connect is a significant reliability improvement
- $18,000/month ($216,000/year) for multi-region is not in the approved program budget
- Business case for Option B requires board approval — not achievable within this program's mandate
- Multi-AZ within us-east-1 protects against the most common failure scenario (single AZ outage)

**Mitigations added to Option A:**
- Route 53 health checks with 30-second failover detection
- CloudWatch alarms at 99% availability threshold with PagerDuty P1 escalation
- Incident runbook documented for regional outage scenario
- Risk formally accepted by VP Engineering in writing (Decision Register DR-004)

**Against Option B:**
- $216K/year is not justified by the incremental reliability gain for our call volume
- Operational complexity doubles — two Connect instances, cross-region routing, dual IaC environments
- Deferred, not rejected — documented as ADR for FY2025 budget submission

---

## Consequences

- **Accepted risk:** 4-hour blast radius if us-east-1 suffers full AZ failure during business hours
- **Monitoring:** CloudWatch dashboard with real-time AZ health indicators
- **FY2025 action:** Tom Reyes to include multi-region infrastructure in FY2025 budget proposal
- This decision is logged in the Decision Register as DR-004 with Sarah Chen's written acceptance
