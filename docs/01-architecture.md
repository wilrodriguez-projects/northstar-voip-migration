# Section 2 — System Architecture

**Document:** `docs/01-architecture.md`  
**Program:** Project NORTH STAR  
**Version:** 2.1  
**Last Updated:** 2024-07-12  
**Author:** Priya Nair (Lead Engineer) · Reviewed by: [TPM Name]  
**Status:** Approved — in use for Wave 2

---

## 2.1 Architecture Overview

Project NORTH STAR migrates all voice infrastructure from a co-located Cisco UCM cluster to AWS Connect. The architecture is designed around three principles:

- **Zero-downtime migration** — legacy and new systems run in parallel during each wave, with instant rollback capability
- **Infrastructure-as-Code first** — all AWS resources managed via Terraform; no manual console changes permitted post-Wave 1
- **Carrier independence** — BYOC (Bring Your Own Carrier) via Twilio Elastic SIP decouples the porting schedule from the AWS go-live

---

## 2.2 Full System Architecture

```
╔══════════════════════════════════════════════════════════════════════╗
║                     PSTN / CARRIER LAYER                            ║
║                                                                      ║
║   ┌─────────────────────┐      ┌──────────────────────────────┐    ║
║   │  Lumen Technologies  │      │  Bandwidth.com               │    ║
║   │  DID Blocks (primary)│      │  Toll-Free + DID (parallel   │    ║
║   │  6,200 DIDs          │      │  port — Wave 2 mitigation)   │    ║
║   └──────────┬──────────┘      └──────────────┬───────────────┘    ║
║              │ LOA / FOC                        │ LOA / FOC          ║
╚══════════════╪══════════════════════════════════╪════════════════════╝
               │                                  │
               ▼                                  ▼
╔══════════════════════════════════════════════════════════════════════╗
║               TWILIO ELASTIC SIP — BYOC INTERIM LAYER               ║
║                                                                      ║
║   • Active during Wave 2 porting (May–Sep 2024)                     ║
║   • Routes inbound calls to AWS Connect via SIP INVITE              ║
║   • Provides carrier abstraction — porting schedule decoupled       ║
║   • SIP credentials stored in AWS Secrets Manager (post TD-001 fix) ║
║                                                                      ║
╚═══════════════════════════════╤══════════════════════════════════════╝
                                │ SIP/RTP
                                ▼
╔══════════════════════════════════════════════════════════════════════╗
║                    AWS us-east-1 (Multi-AZ)                         ║
║                                                                      ║
║  ┌─────────────────────────────────────────────────────────────┐   ║
║  │                   ROUTE 53                                   │   ║
║  │   Health checks on Connect endpoint                         │   ║
║  │   Failover routing to secondary AZ                          │   ║
║  └──────────────────────────┬──────────────────────────────────┘   ║
║                              │                                       ║
║  ┌───────────────────────────▼──────────────────────────────────┐  ║
║  │              AWS CONNECT CLUSTER (Multi-AZ)                   │  ║
║  │                                                               │  ║
║  │   ┌─────────────────┐    ┌──────────────────────────────┐   │  ║
║  │   │  Contact Flows   │    │  Queues & Routing Profiles   │   │  ║
║  │   │  (34 IVR flows)  │    │  (Priority + skill-based)   │   │  ║
║  │   └────────┬─────────┘    └──────────────┬───────────────┘   │  ║
║  │            │                              │                    │  ║
║  │   ┌────────▼──────────────────────────────▼──────────────┐   │  ║
║  │   │           AMAZON CONNECT STREAMS API                  │   │  ║
║  │   │   Agent desktop integration · CTR events              │   │  ║
║  │   └───────────────────────────────────────────────────────┘   │  ║
║  └──────────────────────────┬────────────────────────────────────┘  ║
║                              │                                       ║
║  ┌───────────────────────────▼──────────────────────────────────┐  ║
║  │         APPLICATION LOAD BALANCER (blue/green)                │  ║
║  │   Weighted routing · Zero-downtime deployment                 │  ║
║  └──────────┬────────────────────────────────────────────────────┘  ║
║             │                                                        ║
║  ┌──────────▼──────────────────────────────────────────────────┐   ║
║  │                  LAMBDA FUNCTIONS                             │   ║
║  │                                                               │   ║
║  │  ┌─────────────────┐  ┌──────────────┐  ┌────────────────┐ │   ║
║  │  │ CNAM Lookup      │  │ Port Status  │  │ CTR Processor  │ │   ║
║  │  │ (static CSV)     │  │ Webhook      │  │ (event stream) │ │   ║
║  │  │ TD-007: interim  │  │              │  │                │ │   ║
║  │  └────────┬─────────┘  └──────┬───────┘  └───────┬────────┘ │   ║
║  └───────────╪────────────────────╪──────────────────╪──────────┘   ║
║              │                    │                   │              ║
║  ┌───────────▼──────┐  ┌──────────▼──────┐  ┌───────▼────────┐    ║
║  │   DYNAMODB        │  │  SECRETS MGR    │  │  KINESIS       │    ║
║  │   DID state table │  │  SIP creds      │  │  CTR stream    │    ║
║  │   Port history    │  │  API keys       │  │  (future dash) │    ║
║  └───────────────────┘  └─────────────────┘  └────────────────┘    ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
                                │
                    ┌───────────▼──────────────┐
                    │  LEGACY — CISCO UCM 11.5  │
                    │  Co-lo: Ashburn, VA        │
                    │  STATUS: Parallel (Wave 2) │
                    │  DECOMMISSION: Nov 2024    │
                    └──────────────────────────┘
```

---

## 2.3 Component Inventory

| Component | Type | Owner Team | Purpose | Wave Introduced |
|---|---|---|---|---|
| Lumen Technologies SIP trunk | External vendor | Telecom Ops | Primary DID carrier — 6,200 numbers | Existing |
| Bandwidth.com SIP trunk | External vendor | Telecom Ops | Parallel porting carrier (R-001 mitigation) | Wave 2 |
| Twilio Elastic SIP (BYOC) | Managed service | Platform Infra | Carrier abstraction during porting | Wave 1 |
| AWS Connect Cluster | AWS managed | Platform Infra | Call routing, IVR, agent management | Wave 1 |
| Contact Flows (IVR) | AWS Connect config | Platform Infra + QA | 34 call flows migrated from Cisco UCM | Wave 1–3 |
| Route 53 Health Checks | AWS DNS | Platform Infra | Endpoint monitoring, AZ failover | Wave 1 |
| ALB (blue/green) | AWS load balancer | Platform Infra | Zero-downtime deployments | Wave 1b |
| Lambda — CNAM Lookup | AWS serverless | Platform Infra | Caller ID display (static CSV, interim) | Wave 2 |
| Lambda — Port Status Webhook | AWS serverless | Telecom Ops | Real-time porting status updates | Wave 2 |
| Lambda — CTR Processor | AWS serverless | Platform Infra | Contact Trace Record event processing | Wave 2 |
| DynamoDB — DID State | AWS NoSQL | Platform Infra | DID assignment, porting history | Wave 1 |
| Secrets Manager | AWS security | Platform Infra | SIP credentials, API keys (replaces TD-001) | Wave 2 |
| Kinesis Data Streams | AWS streaming | Platform Infra | CTR event stream (feeds future dashboard) | Wave 2 |
| Cisco UCM 11.5 | Legacy PBX | Telecom Ops | Parallel operation during migration | Decommission Wave 3 |
| Terraform (IaC) | DevOps tooling | Platform Infra | All AWS resource provisioning | Wave 1 |
| GitHub Actions (CI/CD) | DevOps tooling | Platform Infra | IaC validation, Lambda deployment | Wave 1b |

---

## 2.4 Data Flow — Inbound Call (Wave 2 State)

```
1. INBOUND CALL arrives at DID (e.g. +1-800-555-0100)
        │
        ▼
2. LUMEN / BANDWIDTH routes to Twilio Elastic SIP trunk
        │
        ▼
3. TWILIO sends SIP INVITE to AWS Connect endpoint
   [SIP credentials retrieved from Secrets Manager]
        │
        ▼
4. ROUTE 53 health check confirms Connect endpoint healthy
        │
        ▼
5. AWS CONNECT receives call → matches DID to Contact Flow
        │
        ├── DID lookup → DynamoDB (DID state table)
        │
        ├── CNAM lookup → Lambda (static CSV response)
        │
        └── Routes to appropriate IVR flow
                │
                ▼
6. IVR CONTACT FLOW executes
   (e.g. "Press 1 for Sales, 2 for Support...")
        │
        ├── Self-service: handled in IVR, call ends
        │
        └── Agent queue: routes to next available agent
                │
                ▼
7. AGENT DESKTOP receives call via Connect Streams API
        │
        ▼
8. CALL ENDS → CTR (Contact Trace Record) generated
        │
        ▼
9. LAMBDA (CTR Processor) streams event to Kinesis
   [Future: real-time analytics dashboard — CR-008, deferred]
```

---

## 2.5 Infrastructure-as-Code Structure

All AWS resources are managed via Terraform. State is stored in S3 with DynamoDB locking (resolved TD-003).

```
terraform/
├── main.tf                    # Root module
├── variables.tf               # Environment variables
├── outputs.tf                 # Export values
├── modules/
│   ├── connect/
│   │   ├── main.tf            # AWS Connect instance config
│   │   ├── contact_flows.tf   # IVR flow definitions
│   │   └── queues.tf          # Routing queues
│   ├── networking/
│   │   ├── vpc.tf             # VPC, subnets, security groups
│   │   ├── alb.tf             # Application Load Balancer
│   │   └── route53.tf         # DNS + health checks
│   ├── compute/
│   │   ├── lambda_cnam.tf     # CNAM lookup function
│   │   ├── lambda_webhook.tf  # Port status webhook
│   │   └── lambda_ctr.tf      # CTR processor
│   └── data/
│       ├── dynamodb.tf        # DID state + port history tables
│       ├── kinesis.tf         # CTR event stream
│       └── secrets.tf         # Secrets Manager config
└── environments/
    ├── dev/
    ├── staging/
    └── prod/
```

---

## 2.6 Dependencies & Constraints

| Dependency | Type | Risk Level | Owner | Notes |
|---|---|---|---|---|
| Lumen LOA processing | External / Carrier | HIGH | James W. | 90-day slip — see R-001, dependency-log.md |
| Bandwidth.com FOC dates | External / Carrier | MEDIUM | James W. | Parallel port strategy — Aug 15 target |
| AWS Connect service limits | Platform constraint | LOW | Priya N. | Concurrency limit raised via support ticket |
| CNAM vendor API readiness | External / Vendor | MEDIUM | James W. | Not production-ready — static fallback active |
| Cisco UCM parallel operation | Internal / Legacy | LOW | James W. | Costs $28,500/mo — decommission gates on Wave 3 |
| SIP engineer availability | Staffing | MEDIUM | Sarah C. | Single-person knowledge risk — see R-006 |
| IVR test coverage | Internal / QA | MEDIUM | Dana K. | At 61% — must reach 80% before Wave 3 gate |

---

## 2.7 Security Architecture

| Control | Implementation | Status |
|---|---|---|
| SIP credential storage | AWS Secrets Manager (migrated from env vars) | In Progress — Sprint 15 |
| Network isolation | VPC with private subnets for Lambda + DynamoDB | Complete |
| IAM least privilege | Per-function execution roles, no wildcard policies | Complete |
| Console change prevention | SCP blocking manual AWS Console edits to Connect | In Progress — Sprint 15 |
| IaC enforcement | GitHub Actions gate — Terraform plan required on all PRs | Complete |
| Secrets rotation | Automated 90-day rotation via Secrets Manager | Planned — Sprint 17 |
| CloudTrail logging | All API calls logged to S3 + CloudWatch | Complete |

---

*Back: [Program Overview](./00-program-overview.md) | Next: [Technical Trade-offs →](./04-technical-tradeoffs.md)*
