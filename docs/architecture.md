# PAYai – System Architecture

## Overview

PAYai is a cloud-native, event-driven platform composed of loosely coupled microservices. Each service owns its data store and communicates via an internal message bus (async) or REST/gRPC (sync low-latency paths).

---

## High-Level Component Diagram

```
                        ┌────────────────────────────────────────────────┐
                        │              External Clients                  │
                        │  Mobile App · Web App · POS Terminal           │
                        └──────────────────┬─────────────────────────────┘
                                           │ HTTPS / WebSocket
                                           ▼
                        ┌────────────────────────────────────────────────┐
                        │           PAYai API Gateway                    │
                        │   (Kong / AWS API Gateway)                     │
                        │   Auth (OAuth 2.0 / PKCE) · Rate Limiting      │
                        │   TLS 1.3 · WAF · DDoS Protection              │
                        └──┬─────────────┬────────────────┬──────────────┘
                           │             │                │
              ┌────────────▼──┐  ┌───────▼──────┐  ┌────▼───────────────┐
              │  Auth Service │  │ Payment Orch.│  │  Notification Svc  │
              │  (JWT / MFA)  │  │  (Saga coord)│  │  (Push/SMS/Email)  │
              └───────────────┘  └──────┬───────┘  └────────────────────┘
                                        │
              ┌─────────────────────────┼───────────────────────────────┐
              │                         │                               │
    ┌─────────▼──────┐  ┌──────────────▼──────────┐  ┌────────────────▼──┐
    │  AI Risk Engine│  │   Micro-Loan Engine      │  │  Card Issuing Svc │
    │  (ML inference │  │   (credit decisioning,   │  │  (virtual/physical│
    │   < 200 ms)    │  │    disbursement,         │  │   Mastercard/Visa)│
    └────────────────┘  │    repayment)            │  └───────────────────┘
                        └──────────────┬───────────┘
                                       │
                        ┌──────────────▼───────────────┐
                        │   XRP Ledger Settlement Svc  │
                        │   (XRPL node, ODL, validator) │
                        └──────────────────────────────┘
                                       │
                        ┌──────────────▼───────────────┐
                        │  Payment Network Bridge      │
                        │  (ISO 8583 / ISO 20022)      │
                        │  Mastercard · Visa · Acquirer │
                        └──────────────────────────────┘
                                       │
                        ┌──────────────▼───────────────┐
                        │   Merchant Gateway           │
                        │   (REST API · Webhooks)      │
                        └──────────────────────────────┘
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| API Gateway | Kong Gateway / AWS API Gateway |
| Backend services | Node.js (TypeScript) / Python (ML services) |
| Message bus | Apache Kafka |
| Databases | PostgreSQL (transactional), Redis (cache/sessions), MongoDB (audit logs) |
| ML platform | Python, scikit-learn, XGBoost, deployed via MLflow |
| Blockchain | XRP Ledger (XRPL), xrpl.js / xrpl-py SDKs |
| Card issuing | Marqeta or Railsr (BaaS) |
| Infrastructure | AWS (EKS, RDS, ElastiCache, MSK) |
| CI/CD | GitHub Actions, ArgoCD |
| Observability | Prometheus, Grafana, OpenTelemetry, PagerDuty |

---

## Service Responsibilities

### API Gateway
- Single ingress point for all external traffic.
- Handles authentication (JWT validation), rate limiting, and TLS termination.
- Routes requests to appropriate downstream microservices.

### Payment Orchestration Service
- Coordinates the end-to-end payment saga (authorisation → loan decision → settlement → notification).
- Implements the **Saga pattern** with compensating transactions for rollback on failure.

### AI Risk Engine
- Real-time scoring of every transaction (< 200 ms p99 latency).
- Inputs: open-banking balance, transaction history, merchant category, time-of-day, device signals.
- Output: approval/decline recommendation + credit limit for bridge loan.

### Micro-Loan Engine
- Manages loan lifecycle: origination, disbursement, repayment, collections.
- Integrates with open-banking APIs to detect salary credits and trigger repayment.

### XRP Ledger Settlement Service
- Runs a validated XRPL node.
- Executes on-demand liquidity (ODL) transfers for cross-border payments.
- Monitors transaction finality and emits settlement events.

### Card Issuing Service
- Issues virtual and physical Mastercard / Visa prepaid or credit cards.
- Manages card lifecycle: provisioning, tokenisation, controls, disputes.

### Merchant Gateway
- Provides REST APIs and webhooks for merchant integration.
- Delivers real-time settlement notifications and reconciliation reports.

---

## Deployment Architecture

- **Multi-region active-active** deployment on AWS (primary regions: EU-WEST-1, US-EAST-1).
- **Kubernetes (EKS)** for container orchestration with horizontal pod autoscaling.
- **99.99 % SLA** for the payment critical path.
- **Blue/green deployments** for zero-downtime releases.

---

## Data Flow Security

1. All data in transit encrypted with TLS 1.3.
2. All data at rest encrypted with AES-256.
3. PAN (Primary Account Number) data tokenised immediately at ingress; raw PAN never stored.
4. Secrets managed via AWS Secrets Manager / HashiCorp Vault.
