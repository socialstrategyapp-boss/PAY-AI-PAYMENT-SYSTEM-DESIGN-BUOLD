# PAYai – Card Network Integration (Mastercard / Visa)

## Purpose

PAYai issues a co-branded payment card (virtual and physical) through integration with Mastercard and/or Visa. This gives consumers a universally accepted payment instrument backed by PAYai's bridge-credit facility, and enables PAYai to increase transaction approval rates across all partner merchants who already accept Mastercard / Visa.

---

## Card Product Types

| Product | Description |
|---|---|
| **PAYai Virtual Card** | Instant-issue digital card for in-app and online payments; Apple Pay / Google Pay compatible |
| **PAYai Physical Card** | Contactless debit/prepaid card delivered by post; works at any Mastercard or Visa terminal worldwide |
| **PAYai Credit Card** | Full revolving credit card (Phase 3); credit limit backed by PAYai's AI underwriting |

---

## Integration Architecture

```
Consumer
   │ Tap / swipe / online checkout
   ▼
Merchant POS / Online Checkout
   │ Authorisation request (ISO 8583)
   ▼
Acquirer
   │
   ▼
Mastercard / Visa Network
   │ Authorisation request forwarded to PAYai BIN range
   ▼
┌────────────────────────────────────┐
│   PAYai Card Issuing Service       │
│   (via BaaS partner: Marqeta /    │
│    Railsr / Thredd)                │
│                                    │
│   ┌──────────────────────────────┐ │
│   │  AI Risk Engine decision     │ │
│   │  (approve / approve+loan /   │ │
│   │   decline)                   │ │
│   └──────────────────────────────┘ │
└────────────────────────────────────┘
   │ Authorisation response
   ▼
Mastercard / Visa Network
   │
   ▼
Acquirer → Merchant
```

---

## BIN Sponsorship

- PAYai will obtain a **Bank Identification Number (BIN)** range from Mastercard and/or Visa.
- Initial route: sponsor bank or **Banking-as-a-Service (BaaS)** partner holds the principal membership; PAYai acts as programme manager.
- Target (Phase 3+): direct principal membership with Mastercard / Visa for full issuer control.

### BaaS Partner Options

| Partner | Network | Notes |
|---|---|---|
| Marqeta | Mastercard, Visa | Modern card issuing platform; real-time authorisation webhooks |
| Railsr (formerly Railsbank) | Mastercard | UK/EU focus; FCA regulated |
| Thredd (formerly GPS) | Mastercard, Visa | Global; ISO 27001 certified |

---

## Network Token Infrastructure

- Enrol all PAYai cards in **Mastercard Digital Enablement Service (MDES)** and **Visa Token Service (VTS)**.
- Network tokens replace the 16-digit PAN with a merchant-specific token, improving:
  - **Approval rates** (tokens survive card reissue; no declines due to stale card numbers).
  - **Security** (token cannot be used at other merchants even if intercepted).
  - **Conversion** for recurring subscriptions and saved-card payments.

---

## Authorisation Processing

### Message Format
- **ISO 8583** for real-time card authorisation messages (legacy merchant POS).
- **ISO 20022** for clearing and settlement.

### PAYai Authorisation Flow

```
1. Receive auth request from Mastercard/Visa network (< 2 s timeout)
2. Validate card status (active, not blocked)
3. Call AI Risk Engine (target: < 200 ms)
   ├── APPROVE → respond with authorisation code
   ├── APPROVE_WITH_LOAN → fund via Micro-Loan Engine → respond approved
   └── DECLINE → respond with decline reason code
4. Log transaction to audit database
5. Emit event to Payment Orchestration Service
```

### Response Codes
PAYai maps internal decisions to standard Mastercard/Visa response codes:

| PAYai Decision | ISO 8583 Response Code |
|---|---|
| APPROVE | `00` – Approved |
| APPROVE_WITH_LOAN | `00` – Approved |
| DECLINE (insufficient credit) | `51` – Insufficient funds |
| DECLINE (fraud) | `59` – Suspected fraud |
| DECLINE (blocked merchant) | `57` – Transaction not permitted |

---

## Settlement & Clearing

- **Clearing**: PAYai submits clearing files to Mastercard/Visa at end of day (ISO 20022 pain.001).
- **Settlement**: Funds settled between acquirer and PAYai issuer via the card network settlement window (T+1 for domestic, T+2 for international).
- XRP Ledger used for **internal** settlement between PAYai entities; card network settlement follows standard network timelines.

---

## Interchange Revenue

- PAYai earns **interchange fees** on every transaction (typically 0.2 %–1.5 % of transaction value depending on card type and region).
- Under EU/UK interchange caps: consumer debit ≤ 0.2 %, consumer credit ≤ 0.3 %.
- Higher interchange available for premium or business card tiers.

---

## Card Controls (Consumer App)

Consumers can manage their PAYai card directly in the app:

- Freeze / unfreeze card instantly.
- Set spending limits (daily, per-merchant, per-category).
- Block specific merchant categories (e.g. gambling, alcohol).
- Enable / disable contactless, online, or international payments.
- View real-time transaction feed with push notifications.

---

## Compliance

| Requirement | Standard |
|---|---|
| Card data security | PCI-DSS Level 1 |
| Card scheme rules | Mastercard Rules / Visa Core Rules |
| Consumer protection | UK Consumer Duty / EU PSD2 |
| Card issuing licence | Via BaaS sponsor bank (e-money / credit institution authorisation) |
