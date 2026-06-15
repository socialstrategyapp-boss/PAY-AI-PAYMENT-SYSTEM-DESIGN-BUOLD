# PAYai – XRP Ledger Settlement Layer

## Purpose

PAYai uses the **XRP Ledger (XRPL)** as its backbone for fast, low-cost settlement between parties. XRP enables near-instant finality (3–5 seconds) at a fraction of the cost of traditional payment rails (SWIFT, ACH, SEPA), making it ideal for the high-volume, low-margin micro-credit and payment use case at the heart of PAYai.

---

## Why XRP Ledger?

| Capability | XRP Ledger | Traditional Rails |
|---|---|---|
| Settlement time | 3–5 seconds | T+0 to T+2 days |
| Cost per transaction | < $0.001 | $0.25 – $25+ |
| Availability | 24 / 7 / 365 | Business hours / cut-off times |
| Scalability | 1,500+ TPS | Varies (often batch) |
| On-demand liquidity | Yes (ODL via Ripple) | Pre-funded nostro accounts required |
| Energy consumption | Carbon-neutral | High (varies) |

---

## XRPL Integration Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                    PAYai Settlement Service                   │
│                                                               │
│   ┌──────────────┐    ┌──────────────┐    ┌────────────────┐ │
│   │ Payment Event│    │ XRP Ledger   │    │ Settlement     │ │
│   │ Consumer     │───►│ Node (full   │───►│ Confirmation   │ │
│   │ (Kafka topic)│    │  validator)  │    │ Event Emitter  │ │
│   └──────────────┘    └──────┬───────┘    └────────────────┘ │
│                              │                               │
└──────────────────────────────│───────────────────────────────┘
                               │ XRPL P2P network
                               ▼
                    ┌─────────────────────┐
                    │   XRP Ledger        │
                    │   (Mainnet)         │
                    │   3–5 s finality    │
                    └─────────────────────┘
                               │
                    ┌─────────────────────┐
                    │  Ripple ODL / RippleNet│
                    │  (On-demand Liquidity) │
                    └─────────────────────┘
```

---

## Settlement Flows

### 1. Consumer-to-Merchant Settlement (Domestic)

When PAYai funds a bridge loan payment:

```
1. PAYai internal ledger debits PAYai float account
2. XRPL payment: PAYai wallet → Merchant acquirer wallet
   (amount in XRP or IOU settled in fiat via ODL)
3. XRPL confirms finality (~4 s)
4. PAYai internal ledger credits merchant acquirer
5. Settlement event emitted → Merchant Gateway notified
```

### 2. Cross-Border Settlement

For international transactions (using Ripple ODL):

```
1. PAYai initiates ODL transaction (source currency → XRP → destination currency)
2. Market maker on XRPL converts at real-time FX rate
3. Destination party receives local fiat (GBP, USD, EUR, etc.)
4. Total time: < 10 seconds end-to-end
5. Cost: < $0.01 vs $25–$50 for SWIFT wire
```

### 3. Consumer Repayment Settlement

```
1. Open-banking webhook: salary deposit detected
2. PAYai initiates repayment collection (direct debit / open-banking payment)
3. Collected funds transferred on XRPL to PAYai float pool
4. Loan marked REPAID; interest income recorded
```

---

## XRPL Node Operations

### Node Configuration
- PAYai operates as a **stock validator** node on XRPL Mainnet.
- Node participates in the XRPL consensus protocol (Federated Byzantine Agreement).
- Validator is listed on the default Unique Node List (UNL) after trust is established with the network.

### Validator Rewards
- XRPL validators do not receive block rewards from the protocol directly (unlike PoW/PoS chains).
- Revenue from validation comes from:
  1. **Network trust / credibility** – listed on UNL enables commercial ODL partnerships with Ripple.
  2. **Ripple partnership fees** – validator operators may negotiate ODL referral / volume arrangements.
  3. **Transaction fee destruction** – while fees are burned on XRPL, operating a node reduces reliance on third parties and eliminates per-query fees to external node providers.

### High Availability
- Primary validator: AWS EU-WEST-1
- Secondary (standby) validator: AWS US-EAST-1
- RTO: < 30 seconds automated failover
- Node key rotation: every 90 days per XRPL security best practices

---

## Wallet Management

| Wallet | Purpose |
|---|---|
| `PAYai Float Wallet` | Holds XRP / IOUs used to fund bridge loan payments |
| `Settlement Wallet` | Receives repayment funds from consumers |
| `Operational Wallet` | Pays XRPL transaction fees (funded with a small XRP reserve) |
| `Cold Storage Wallet` | Long-term XRP reserve; hardware security module (HSM) protected |

- All production wallets use **multi-signature** (multisig) accounts on XRPL.
- Private keys stored in AWS CloudHSM / Hardware Security Modules.
- Signing requires M-of-N key holders (e.g. 2-of-3 for operational, 3-of-5 for cold storage).

---

## Risk Management

| Risk | Mitigation |
|---|---|
| XRP price volatility | Minimise XRP holding time; convert to fiat immediately post-settlement via ODL |
| XRPL node downtime | Active-active dual node setup; fallback to traditional rails if XRPL unavailable |
| Smart contract bugs | XRPL uses no Turing-complete smart contracts; payment transactions are simple and auditable |
| Regulatory | XRP classification monitored per jurisdiction; legal counsel engaged for each market |

---

## SDK & Libraries

| Language | Library | Usage |
|---|---|---|
| TypeScript | `xrpl` (v3.x) | Payment submission, account management |
| Python | `xrpl-py` (v2.x) | ML pipeline settlement events, data analysis |

---

## Monitoring

- Real-time XRPL transaction monitoring via `xrpl` WebSocket subscriptions.
- Alerts on: transaction failure, ledger gap, node desync, balance below threshold.
- Dashboard: Grafana + custom XRPL metrics exporter.
