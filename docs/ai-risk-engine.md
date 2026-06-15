# PAYai – AI Approval & Risk Engine

## Purpose

The AI Risk Engine makes real-time credit and fraud decisions for every transaction processed by PAYai. Its primary objectives are:

1. **Maximise approval rates** for legitimate transactions by consumers and merchants.
2. **Minimise fraud** and default losses.
3. **Ensure fairness** – decisions must be explainable and free from unlawful bias.

---

## Performance Targets

| Metric | Target |
|---|---|
| Decision latency (p50) | < 50 ms |
| Decision latency (p99) | < 200 ms |
| Transaction approval rate (qualified consumers) | > 95 % |
| False positive fraud rate | < 0.5 % |
| Model retraining cadence | Daily (incremental) / Weekly (full) |

---

## Input Features

### Consumer signals
- Current account balance (open-banking, real-time)
- Average balance over trailing 30 / 90 days
- Income regularity and amount (open-banking)
- Existing PAYai loan balance and repayment history
- Account age and activity
- Credit bureau score (where available and consented)

### Transaction signals
- Transaction amount
- Merchant name, category code (MCC), and location
- Time of day and day of week
- Device fingerprint and IP geolocation
- Distance from typical purchase location
- Velocity (number of transactions in last 1 h / 24 h)

### Contextual signals
- Consumer's declared income date (to predict next repayment)
- Blacklist / watchlist checks (fraud, AML)
- Card network signals (Mastercard / Visa risk scores where available)

---

## Model Architecture

```
Input Features (normalised)
         │
         ▼
┌─────────────────────────────┐
│  Feature Engineering Layer  │
│  (real-time aggregations,   │
│   embedding lookups)        │
└──────────────┬──────────────┘
               │
       ┌───────┴────────┐
       │                │
       ▼                ▼
┌──────────────┐  ┌──────────────┐
│  Credit      │  │  Fraud       │
│  Score Model │  │  Score Model │
│  (XGBoost)   │  │  (XGBoost +  │
│              │  │   neural net)│
└──────┬───────┘  └──────┬───────┘
       │                 │
       └────────┬─────────┘
                │
                ▼
┌───────────────────────────────┐
│  Decision Engine              │
│  ─────────────────────────── │
│  credit_score ≥ threshold?    │
│  fraud_score < threshold?     │
│  balance check                │
│  regulatory affordability     │
│                               │
│  → APPROVE / APPROVE_WITH_    │
│    LOAN / DECLINE             │
└───────────────────────────────┘
```

---

## Decision Outcomes

| Outcome | Meaning |
|---|---|
| `APPROVE` | Consumer balance sufficient; standard card approval |
| `APPROVE_WITH_LOAN` | Balance insufficient; bridge loan approved; payment funded by PAYai |
| `DECLINE` | Risk score too high, loan limit exceeded, or fraud detected |

---

## Explainability

Every decision is logged with the top contributing features (SHAP values) to:
- Allow consumers to understand why a transaction was declined.
- Support regulatory audits and fairness reviews.
- Enable customer support to explain decisions clearly.

---

## Fairness & Bias Monitoring

- Monthly bias audits across age, gender, and postcode/zip-code groups.
- Adverse action notices provided to consumers on any decline (regulatory requirement).
- No use of protected characteristics as direct model inputs.

---

## Model Deployment

| Stage | Description |
|---|---|
| Training | Jupyter notebooks → MLflow experiment tracking |
| Validation | Offline A/B evaluation on holdout dataset |
| Staging | Shadow mode on 10 % of live traffic; compare against production model |
| Production | Canary rollout (5 % → 25 % → 100 %) with automatic rollback on KPI degradation |
| Monitoring | Real-time drift detection; alert if approval rate drops > 2 % or fraud rate rises > 0.1 % |

---

## API Contract

### Request
```json
POST /v1/risk/score
{
  "consumer_id": "uuid",
  "transaction": {
    "amount": 49.99,
    "currency": "GBP",
    "merchant_id": "uuid",
    "merchant_category_code": "5411",
    "channel": "contactless"
  },
  "device": {
    "fingerprint": "...",
    "ip": "..."
  }
}
```

### Response
```json
{
  "decision": "APPROVE_WITH_LOAN",
  "credit_score": 0.82,
  "fraud_score": 0.03,
  "approved_loan_amount": 49.99,
  "currency": "GBP",
  "latency_ms": 47,
  "decision_id": "uuid",
  "top_factors": [
    { "feature": "balance_30d_avg", "contribution": 0.31 },
    { "feature": "income_regularity", "contribution": 0.28 }
  ]
}
```
