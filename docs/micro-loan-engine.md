# PAYai – Micro-Loan Engine Design

## Purpose

The Micro-Loan Engine provides short-term bridge credit to consumers whose account balance is insufficient to cover a purchase at the point of sale. The loan covers the shortfall for the specific transaction, with repayment automatically collected on the consumer's next income deposit.

---

## Loan Product Specification

| Parameter | Value |
|---|---|
| Minimum loan | £/$/€ 1 |
| Maximum loan (initial) | £/$/€ 500 |
| Maximum loan (established customer) | £/$/€ 2,000 |
| Loan term | 1 – 30 days |
| Interest model | Daily flat rate (e.g. 0.5 % / day) |
| Repayment trigger | Next qualifying income deposit detected via open banking |
| Repayment fallback | Direct debit / scheduled bank transfer on day 7 |
| Late repayment fee | Capped (regulatory compliant) |
| Rollover | Not permitted (consumer protection) |

---

## Loan Lifecycle

```
[Consumer initiates payment]
         │
         ▼
[AI Risk Engine scores transaction]
         │
    ┌────▼────────────────────────────────┐
    │  Account balance ≥ payment amount? │
    │            YES                      │──────► Standard card path
    └────┬────────────────────────────────┘
         │ NO (balance shortfall detected)
         ▼
[Micro-Loan Engine: eligibility check]
         │
    ┌────▼─────────────────────────────────────┐
    │  Consumer has active PAYai account?      │
    │  Credit score within policy limits?      │
    │  No outstanding overdue loans?           │
    │  Transaction amount ≤ available limit?   │
    └────┬─────────────────────────────────────┘
         │ ALL CHECKS PASS
         ▼
[Loan originated & disbursed]
[Payment completed on behalf of consumer]
         │
         ▼
[Repayment watch: open-banking income detection]
         │
         ▼
[Income deposit detected → repayment collected]
[Loan closed · Interest earned · Credit limit reviewed]
```

---

## Eligibility Criteria

1. **KYC verified** – consumer identity confirmed at onboarding.
2. **AI risk score** ≥ threshold (set by risk policy, reviewed weekly).
3. **No overdue balance** on any PAYai loan.
4. **Transaction merchant category** not on restricted list (e.g. gambling, adult content).
5. **Total outstanding bridge loans** do not exceed the consumer's approved credit limit.

---

## Repayment Collection

| Method | Trigger | Priority |
|---|---|---|
| Open-banking push | Salary / benefit deposit detected in real time | 1 (preferred) |
| Direct debit | Scheduled on loan origination date + 7 days | 2 |
| Card charge | Registered debit/credit card on file | 3 |
| Manual transfer | Consumer-initiated bank transfer | 4 |

If repayment fails after exhausting all methods, the account enters a **soft collections** workflow with consumer support outreach before any third-party debt collection is initiated (regulatory requirement).

---

## Credit Limit Management

- Initial limit set by the AI Risk Engine at onboarding based on income, spending patterns, and credit bureau data.
- Limits reviewed automatically after every 5 successful repayments (upward revision) or any late repayment (downward revision).
- Consumers can voluntarily reduce their limit at any time via the app.

---

## Regulatory Compliance

- Consumer credit authorisation required in each operating jurisdiction (e.g. FCA Consumer Credit Licence in the UK).
- Affordability assessments conducted at origination in accordance with responsible lending rules.
- Interest and fees capped per local regulation (e.g. FCA high-cost short-term credit cap).
- Cooling-off period and early repayment rights provided as required.

---

## Data Model (simplified)

```
Loan
├── id              UUID
├── consumer_id     UUID (FK → Consumer)
├── transaction_id  UUID (FK → Transaction)
├── amount          DECIMAL(12,2)
├── currency        CHAR(3)
├── interest_rate   DECIMAL(5,4)   -- daily rate
├── origination_ts  TIMESTAMP
├── due_date        DATE
├── status          ENUM (ACTIVE, REPAID, OVERDUE, WRITTEN_OFF)
└── repayments[]    Repayment[]

Repayment
├── id              UUID
├── loan_id         UUID (FK → Loan)
├── amount          DECIMAL(12,2)
├── method          ENUM (OPEN_BANKING, DIRECT_DEBIT, CARD, MANUAL)
├── collected_ts    TIMESTAMP
└── status          ENUM (PENDING, COLLECTED, FAILED)
```
