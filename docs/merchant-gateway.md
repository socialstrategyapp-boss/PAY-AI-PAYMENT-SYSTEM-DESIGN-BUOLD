# PAYai – Merchant & Retail Partner Gateway

## Purpose

The Merchant Gateway is the integration layer between PAYai and the merchants, supermarkets, and retail partners who accept PAYai as a payment method. It provides APIs, webhooks, and a management dashboard for merchants to integrate PAYai payments, receive real-time settlement notifications, and access reconciliation data.

---

## Merchant Value Proposition

| Benefit | Detail |
|---|---|
| **Higher approval rates** | PAYai's bridge-credit means fewer declines at checkout; more completed sales |
| **Faster settlement** | XRP Ledger settlement in seconds vs. T+2 bank transfers |
| **Lower chargebacks** | AI fraud detection reduces fraudulent transactions |
| **Real-time notifications** | Instant webhook events on payment authorisation and settlement |
| **Analytics dashboard** | Approval rate trends, revenue uplift, reconciliation reports |

---

## Integration Options

### 1. REST API (full integration)
For supermarkets and large retailers building a fully custom checkout experience.

### 2. Hosted Payment Page (HPP)
A PAYai-hosted checkout page for online retailers. Minimal integration effort; PCI-DSS scope offloaded to PAYai.

### 3. SDK / Plugin
Ready-made plugins for popular e-commerce platforms (Shopify, WooCommerce, Magento) and point-of-sale systems.

### 4. Card-Present (NFC / contactless)
For physical retail, PAYai works transparently via the Mastercard / Visa network. No additional integration required beyond standard card acceptance.

---

## REST API Reference

### Base URL
```
https://api.payai.com/v1/merchant
```

### Authentication
All requests must include a ****** in the `Authorization` header:
```
Authorization: ******
```

---

### Initiate Payment

```
POST /v1/merchant/payments
```

**Request**
```json
{
  "amount": 47.50,
  "currency": "GBP",
  "consumer_token": "ctkn_xxxxxxxxxxxx",
  "merchant_reference": "ORDER-20260615-001",
  "description": "Weekly grocery shop",
  "return_url": "https://yourstore.com/payment/result"
}
```

**Response**
```json
{
  "payment_id": "pay_xxxxxxxxxxxx",
  "status": "AUTHORISED",
  "amount": 47.50,
  "currency": "GBP",
  "bridge_loan_used": true,
  "authorised_at": "2026-06-15T10:55:00Z",
  "merchant_reference": "ORDER-20260615-001"
}
```

---

### Get Payment Status

```
GET /v1/merchant/payments/{payment_id}
```

**Response**
```json
{
  "payment_id": "pay_xxxxxxxxxxxx",
  "status": "SETTLED",
  "amount": 47.50,
  "currency": "GBP",
  "settled_at": "2026-06-15T10:55:04Z",
  "xrp_txn_hash": "A1B2C3D4E5F6..."
}
```

---

### Payment Statuses

| Status | Description |
|---|---|
| `PENDING` | Payment initiated, awaiting authorisation |
| `AUTHORISED` | AI risk engine approved; funds reserved |
| `SETTLED` | XRP Ledger settlement confirmed |
| `DECLINED` | Payment declined by risk engine |
| `REFUNDED` | Merchant-initiated refund processed |
| `DISPUTED` | Consumer has raised a dispute / chargeback |

---

## Webhook Events

Merchants register webhook endpoints to receive real-time event notifications.

### Register Webhook

```
POST /v1/merchant/webhooks
```
```json
{
  "url": "https://yourstore.com/payai/webhook",
  "events": ["payment.authorised", "payment.settled", "payment.declined", "refund.processed"],
  "secret": "your_webhook_signing_secret"
}
```

### Event Payload Example: `payment.settled`

```json
{
  "event": "payment.settled",
  "payment_id": "pay_xxxxxxxxxxxx",
  "amount": 47.50,
  "currency": "GBP",
  "merchant_reference": "ORDER-20260615-001",
  "settled_at": "2026-06-15T10:55:04Z",
  "xrp_txn_hash": "A1B2C3D4E5F6...",
  "timestamp": "2026-06-15T10:55:04Z"
}
```

All webhook payloads are signed with HMAC-SHA256 using the merchant's `webhook_secret`. Merchants must verify the `X-PAYai-Signature` header before processing the event.

---

## Refunds

Merchants can issue full or partial refunds via the API:

```
POST /v1/merchant/payments/{payment_id}/refunds
```
```json
{
  "amount": 15.00,
  "currency": "GBP",
  "reason": "Item returned by consumer"
}
```

Refunds are settled back to the consumer's PAYai account (and the bridge loan is adjusted accordingly) via XRPL within seconds.

---

## Merchant Dashboard

The PAYai Merchant Dashboard (web) provides:

- **Real-time transaction feed** – all payments, refunds, and disputes.
- **Approval rate analytics** – PAYai approval rate vs. industry benchmark over time.
- **Revenue uplift report** – estimated additional revenue from PAYai-funded transactions.
- **Settlement reports** – daily settlement summaries; downloadable CSV for reconciliation.
- **Dispute management** – submit evidence for chargeback disputes.

---

## Onboarding a New Merchant

1. Merchant completes online application and KYB (Know Your Business) verification.
2. PAYai conducts risk assessment and approves merchant category.
3. API credentials and webhook secret issued.
4. Integration testing in sandbox environment.
5. Go-live approval and production credentials issued.

---

## Supermarket & Large Retail Partner Integration

For large retail partners (e.g. supermarket chains), PAYai offers enhanced integration:

- **Dedicated account manager** and integration engineer.
- **Co-marketing** – PAYai branding at checkout (in-store signage, website badge).
- **Volume pricing** – negotiated transaction fees based on monthly payment volume.
- **Exclusive features** – early access to new PAYai features (e.g. loyalty rewards integration, BNPL tiers).
- **Data insights** – anonymised spending insights to help partners with inventory and demand planning.
