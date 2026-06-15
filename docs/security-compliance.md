# PAYai – Security Controls & Regulatory Compliance

## Overview

Security and regulatory compliance are foundational to PAYai. As a payment service provider handling consumer financial data and credit, PAYai must meet stringent requirements across data security, consumer protection, anti-money laundering, and financial regulation.

---

## Data Security

### Encryption
| Data State | Standard |
|---|---|
| In transit | TLS 1.3 (all external and internal service communication) |
| At rest | AES-256 (all databases, object storage, backups) |
| Card data (PAN) | Tokenised immediately at ingress; PAN never stored in plain text |
| Encryption key management | AWS CloudHSM / HashiCorp Vault; keys rotated every 90 days |

### Access Control
- Role-Based Access Control (RBAC) for all internal systems.
- Multi-Factor Authentication (MFA) mandatory for all employee access to production systems.
- Principle of least privilege: services only have permissions required for their function.
- Privileged Access Management (PAM) with just-in-time access for production databases.

### Network Security
- Web Application Firewall (WAF) on all public-facing endpoints.
- DDoS protection (AWS Shield Advanced / Cloudflare).
- VPC isolation: payment services run in private subnets with no direct internet access.
- All inter-service communication within VPC (no public internet traversal).

---

## PCI-DSS Compliance

PAYai targets **PCI-DSS Level 1** certification (required for > 6 million transactions/year).

| PCI-DSS Requirement | PAYai Implementation |
|---|---|
| Install and maintain a firewall | AWS Security Groups + WAF |
| Do not use vendor-supplied defaults | CIS Benchmark hardened AMIs; no default passwords |
| Protect stored cardholder data | Tokenisation; no PAN storage |
| Encrypt transmission of cardholder data | TLS 1.3 end-to-end |
| Use and regularly update anti-virus | AWS GuardDuty + EDR on all nodes |
| Develop and maintain secure systems | Secure SDLC; SAST/DAST in CI/CD pipeline |
| Restrict access to cardholder data | RBAC + PAM |
| Assign unique IDs to each person | Individual accounts; no shared credentials |
| Restrict physical access | Cloud-only infrastructure (no on-premise card data) |
| Track and monitor all access | Centralised logging (CloudTrail, Splunk) |
| Regularly test security systems | Annual penetration tests; quarterly vulnerability scans |
| Maintain an information security policy | Written policy; annual staff training |

---

## Consumer Identity Verification (KYC / eKYC)

All consumers must complete identity verification before accessing PAYai credit facilities:

| KYC Step | Method |
|---|---|
| Document verification | AI-powered ID document scan (passport, driving licence) |
| Biometric liveness | Selfie + liveness check vs. ID document photo |
| Sanctions / PEP screening | Real-time check against OFAC, UN, EU, HM Treasury sanctions lists |
| Address verification | Open-banking address match or utility bill upload |
| Ongoing monitoring | Periodic re-verification; continuous transaction monitoring |

---

## Anti-Money Laundering (AML) & Counter-Terrorism Financing (CTF)

- **Transaction monitoring**: All transactions screened in real time against rules and ML models for suspicious activity.
- **Suspicious Activity Reports (SARs)**: Automated flagging with compliance team review; SARs filed with relevant authority (NCA in UK, FinCEN in US) where required.
- **Customer Due Diligence (CDD)**: Risk-based approach; enhanced due diligence for high-risk customers or jurisdictions.
- **Source of funds**: Documented for accounts above a risk threshold.

---

## Regulatory Licences & Authorisations

| Jurisdiction | Licence Required | PAYai Approach |
|---|---|---|
| United Kingdom | FCA Consumer Credit Authorisation + E-Money Institution (EMI) or Payment Institution (PI) licence | Phase 1: partner with FCA-authorised BaaS provider; Phase 2: apply for own licence |
| European Union | Central Bank authorisation (EMI / Credit Institution) | Via EU BaaS partner; own licence post-Series A |
| United States | State Money Transmitter Licences (MTLs) + potential state lending licences | Partner with chartered bank; NMLS registration |

---

## Data Protection & Privacy

| Regulation | Compliance Measures |
|---|---|
| GDPR (EU / UK) | Privacy by design; data minimisation; right to erasure; DPA agreements with all processors; DPO appointed |
| CCPA (California) | Consumer rights portal; opt-out of data sale; annual privacy audit |
| Open Banking (PSD2 / FDX) | Explicit consumer consent for open-banking data access; read-only API scopes; consent withdrawal supported |

---

## Secure Software Development Lifecycle (SSDLC)

| Phase | Security Activity |
|---|---|
| Design | Threat modelling (STRIDE) for new features |
| Development | Static Application Security Testing (SAST) in IDE + CI |
| Code review | Security-focused peer review; automated dependency scanning |
| Testing | Dynamic Application Security Testing (DAST); penetration testing |
| Deployment | Infrastructure-as-code security scanning (Checkov / tfsec) |
| Operations | Runtime security monitoring; vulnerability management programme |

---

## Incident Response

- **P1 (Critical) SLA**: Acknowledge within 15 minutes; contain within 2 hours.
- Dedicated Security Operations Centre (SOC) with 24/7 monitoring.
- Incident response runbooks for: data breach, DDoS, account takeover, fraudulent transaction spike.
- Mandatory breach notification: ICO (UK) / DPAs within 72 hours of confirmed personal data breach.
- Customer notification: within 72 hours of confirmed breach affecting their data.

---

## Third-Party Risk Management

- All third-party vendors with access to PAYai systems or data undergo security assessment before onboarding.
- PCI-DSS compliance required for all vendors handling card data.
- Annual vendor security reviews; contractual right to audit.
- Supply chain security: dependency scanning (Dependabot / Snyk) with automatic PR creation for vulnerable dependencies.
