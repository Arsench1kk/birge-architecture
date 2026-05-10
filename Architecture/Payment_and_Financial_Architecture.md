---
status: active
version: v5.0
last_updated: 2026-05-07
phase: 1
---

# [[Payment and Financial Architecture]]

## 1. Economic Model

The core structural advantage: instead of a per-ride commission (15–25%), BIRGE charges a **fixed monthly driver subscription**. This realigns incentives — a driver on BIRGE maximises earnings by completing more rides, not by gaming surge algorithms.

2026-05-07 business foundation: driver subscription with zero per-ride commission is non-negotiable. However, all income comparison tables below are planning models until validated with real driver interviews and real corridor occupancy data.

### Driver Income Comparison

| Component | Yandex Go | BIRGE |
|---|---|---|
| Gross monthly ride value | 1,288,000 ₸ | 1,380,000 ₸ (higher occupancy) |
| Platform fees | 244,720 ₸ (19%) | 34,000 ₸ (fixed) |
| Fuel costs | 190,000 ₸ | 175,000 ₸ (less deadheading) |
| Maintenance + insurance | 90,000 ₸ | 90,000 ₸ |
| **Net monthly income** | **763,280 ₸** | **1,081,000 ₸** |
| **vs Yandex Go** | — | **+41.6%** |

### Driver Subscription Tiers

| Tier | Monthly Fee | Features | Target |
|---|---|---|---|
| Starter | 19,000 ₸ | On-demand access, basic dashboard | Part-time (<10 rides/day) |
| Professional | 28,000 ₸ | On-demand + corridor, priority matching, analytics, tax receipts | Full-time (10–20 rides/day) |
| Premium | 38,000 ₸ | All + guaranteed corridor, dedicated support, early corridor access | High-volume (20+ rides/day) |

**Launch discount:** All new drivers receive Professional tier at Starter price (19,000 ₸/month) for the first 6 months.

### Passenger Subscription Tiers

Passenger subscription packages are separate from service tiers.

| Tier | Name | Payment Logic | Status |
|---|---|---|---|
| Тир 1 | Solo Corridor | Monthly subscription for one route/time slot | MVP |
| Тир 2 | Multi Corridor | Monthly subscription covering all pre-declared corridors | MVP recommended/default |
| Тир 3 | Flex Pack | Fixed pack of N rides/month; fixed per-ride amount deducted from balance | MVP |

Confirmed payment decisions:
- Тир 1 and Тир 2 have zero per-ride friction after payment.
- Тир 3 is pack-balance based and exists for unpredictable schedules.
- MVP launches with all three passenger subscription tiers.
- On-demand taxi is fallback outside subscription, not a subscription tier.
- Exact KZT prices are TBD in Pricing & Unit Economics.

---

## 2. Kaspi Integration

Kaspi.kz handles >70% of digital retail transactions in Kazakhstan. Deep integration is an absolute requirement, not optional.

### Payment Flows

| Flow | Mechanism |
|---|---|
| In-app payment | Kaspi Merchant API direct charge |
| Physical QR | Kaspi QR code generation → passenger scans |
| Driver subscription | Recurring billing via Kaspi API on anniversary date |
| Cash (outer districts) | Driver confirms cash amount → `pending_reconciliation` status |

### Webhook Security

Every Kaspi webhook is validated before any account is credited:

```swift
// Sources/App/Modules/Payments/KaspiWebhookSignature.swift
func validateWebhookSignature(body: ByteBuffer, receivedSignature: String, secret: String) -> Bool {
    let expectedSignature = HMAC<SHA256>.authenticationCode(
        for: body.readableBytesView,
        using: SymmetricKey(data: Data(secret.utf8))
    )
    return constantTimeCompare(hex(expectedSignature), receivedSignature)
}
```

Rotating API keys for webhook validation — see [[Security_and_Authentication.md]].

### Idempotency

Network retries must never double-credit a driver's balance:

```sql
INSERT INTO payment_records (kaspi_transaction_id, amount_tenge, ...)
VALUES ($1, $2, ...)
ON CONFLICT (kaspi_transaction_id) DO NOTHING;
```

See [[Open_Questions.md]] OQ-002 for idempotency key format decision.

---

## 3. Driver Earnings Ledger — Three-Tier Settlement

### Real-Time (on ride completion)
1. [[Ride State Machine]] transitions ride to `completed`
2. Backend calculates driver payout immediately
3. Event broadcast via [[WebSocket Hub Architecture]] to iOS app
4. Driver sees updated session earnings — no additional HTTP fetch required

### Daily Settlement
A scheduled PostgreSQL job runs nightly at **21:45** to aggregate all completed fares. At **22:00**, accumulated earnings are automatically transferred to the driver's Kaspi account.

### Monthly Billing
Driver subscription fee charged automatically on anniversary date via Kaspi recurring billing API. This prevents involuntary churn from failed payments.

---

## 4. Cash Reconciliation

For outer Almaty districts where cash remains common on on-demand fallback rides:

```
1. Driver completes cash ride
        │
        ▼
2. iOS app prompts driver to confirm exact cash amount received
        │
        ▼
3. Backend records as `pending_reconciliation`
        │
        ▼
4. Weekly automated job:
   - Aggregates total cash collected by driver
   - Subtracts corridor guarantee subsidies owed TO the driver
   - Net balance applied against upcoming subscription fee
```

---

## 5. Candidate Corridor Performance Guarantee

Older docs proposed a 90-day guarantee with a minimum payment of **1.5× the solo ride rate** when a corridor run has fewer than 2 confirmed passengers.

Current status:

```text
Candidate strategy, not finalized.
```

Before this appears in UI, driver contracts, or financial forecasts, decide affordability, qualification rules, fraud prevention, payout mechanism, and legal risk.

---

## Related Files
- [[Security_and_Authentication.md]] — Kaspi webhook HMAC validation
- [[Ride_State_Machine.md]] — `completed` state triggers earnings calculation
- [[WebSocket_Hub_Architecture.md]] — real-time earnings broadcast to iOS
- [[Database_Schema_and_Migrations.md]] — payment_records schema
- [[Network_Effects_and_Growth.md]] — economic model and corridor density
