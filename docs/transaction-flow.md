# PAYai – End-to-End Transaction Flow

## Overview

This document describes the complete journey of a PAYai payment from the moment a consumer initiates a purchase to the point where the merchant receives funds and the consumer's bridge loan (if used) is logged for repayment.

---

## Participants

| Participant | Role |
|---|---|
| **Consumer** | Cardholder / PAYai app user initiating the purchase |
| **Merchant POS / Checkout** | Physical terminal or online checkout page |
| **Acquirer** | Merchant's bank / payment processor |
| **Mastercard / Visa Network** | Card scheme routing payment authorisation to the correct issuer |
| **PAYai API Gateway** | PAYai's secure entry point for all payment messages |
| **AI Risk Engine** | Real-time credit and fraud scoring (< 200 ms) |
| **Micro-Loan Engine** | Issues bridge loan if consumer balance is short |
| **XRP Settlement Service** | Settles funds on the XRP Ledger (3–5 seconds) |
| **Notification Service** | Sends real-time push / SMS / email to consumer and merchant |

---

## Standard Flow (Balance Sufficient – No Bridge Loan Required)

```
Consumer                  Merchant POS         Mastercard/Visa         PAYai                XRP Ledger
   │                           │                      │                   │                      │
   │─── tap card / pay ───────►│                      │                   │                      │
   │                           │─── auth request ────►│                   │                      │
   │                           │                      │─── route to ─────►│                      │
   │                           │                      │    PAYai BIN       │                      │
   │                           │                      │                   │─ AI Risk score        │
   │                           │                      │                   │  (< 200 ms)           │
   │                           │                      │                   │  balance sufficient   │
   │                           │                      │◄── APPROVED ──────│                      │
   │                           │◄── approved ─────────│                   │                      │
   │◄── receipt / success ─────│                      │                   │                      │
   │                           │                      │                   │─── XRP settlement ──►│
   │                           │                      │                   │    (3–5 s finality)   │
   │◄───────────────────────────────────────────────── push notification ─│                      │
```

**Total time to authorisation: < 2 seconds**
**Total time to settlement: < 10 seconds**

---

## Bridge Loan Flow (Balance Insufficient – Micro-Loan Activated)

```
Consumer                  Merchant POS         Mastercard/Visa         PAYai                XRP Ledger
   │                           │                      │                   │                      │
   │─── tap card / pay ───────►│                      │                   │                      │
   │                           │─── auth request ────►│                   │                      │
   │                           │                      │─── route to ─────►│                      │
   │                           │                      │    PAYai BIN       │                      │
   │                           │                      │                   │─ AI Risk score        │
   │                           │                      │                   │  balance INSUFFICIENT  │
   │                           │                      │                   │─ Micro-Loan Engine    │
   │                           │                      │                   │  eligibility check    │
   │                           │                      │                   │  bridge loan APPROVED │
   │                           │                      │◄── APPROVED ──────│                      │
   │                           │◄── approved ─────────│                   │                      │
   │◄── receipt / success ─────│                      │                   │                      │
   │                           │                      │                   │─── XRP settlement ──►│
   │                           │                      │                   │    PAYai float ──►    │
   │                           │                      │                   │    Merchant acquirer  │
   │◄── push notification ─────────────────────────────────────────────── │                      │
   │    "PAYai covered £47.50. Repay by [date]"        │                   │                      │
```

---

## Repayment Flow

```
Consumer Bank               Open Banking API            PAYai
      │                            │                      │
      │─── salary deposited ──────►│                      │
      │                            │─── webhook event ───►│
      │                            │    "income detected"  │
      │                            │                      │─ Repayment collection
      │                            │                      │  initiated
      │◄── direct debit / payment ─────────────────────── │
      │    collected                │                      │
      │                            │                      │─ Loan status: REPAID
      │◄── push notification "Your PAYai loan of £47.50   │
      │    has been repaid. Your credit limit is restored" │
```

---

## Decline Flow (Bridge Loan Not Approved)

```
Consumer                  Merchant POS         Mastercard/Visa         PAYai
   │                           │                      │                   │
   │─── tap card / pay ───────►│                      │                   │
   │                           │─── auth request ────►│                   │
   │                           │                      │─── route to ─────►│
   │                           │                      │    PAYai BIN       │
   │                           │                      │                   │─ AI Risk score
   │                           │                      │                   │  balance insufficient
   │                           │                      │                   │─ Micro-Loan Engine
   │                           │                      │                   │  eligibility check
   │                           │                      │                   │  DECLINED (risk / limit)
   │                           │                      │◄── DECLINED (51) ─│
   │                           │◄── declined ─────────│                   │
   │◄── card declined message ─│                      │                   │
   │◄── push notification ─────────────────────────────────────────────── │
   │    "Payment declined. Visit app to see options."   │                   │
```

---

## Refund Flow

```
Merchant Dashboard                              PAYai                XRP Ledger
       │                                          │                      │
       │─── POST /payments/{id}/refunds ─────────►│                      │
       │                                          │─ validate refund      │
       │                                          │─ adjust bridge loan   │
       │                                          │  (if applicable)      │
       │                                          │─── XRP transfer ────►│
       │                                          │    PAYai ──► consumer │
       │◄── refund confirmation ──────────────────│                      │
       │                                          │                      │
                                             Consumer receives push notification:
                                             "Refund of £15.00 credited to your PAYai account"
```

---

## Timing Summary

| Step | Target Time |
|---|---|
| Authorisation (AI scoring + loan decision) | < 2 seconds |
| XRP Ledger settlement finality | 3–5 seconds |
| Merchant settlement notification (webhook) | < 6 seconds |
| Consumer push notification | < 6 seconds |
| Repayment collection (on income detection) | < 5 minutes from income deposit |
| Refund processing | < 10 seconds |

---

## Error Handling & Resilience

| Scenario | Handling |
|---|---|
| AI Risk Engine timeout (> 200 ms) | Fallback to rule-based scoring; decision within 2 s total |
| XRP Ledger unavailable | Fallback to traditional rail (SEPA/Faster Payments); settlement within T+0 domestic |
| Merchant webhook failure | Retry with exponential back-off (max 5 retries over 24 h); dashboard notification |
| Open-banking API unavailable | Repayment falls back to scheduled direct debit |
| Duplicate authorisation request | Idempotency key on all payment APIs; duplicate requests return cached response |
