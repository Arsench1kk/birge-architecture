---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1
---

# [[Payment and Financial Architecture]]

## 1. Economic Model

The core structural advantage: instead of a per-ride commission (15–25%), BIRGE charges a **fixed monthly driver subscription**. This realigns incentives — a driver on BIRGE maximises earnings by completing more rides, not by gaming surge algorithms.

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

### Passenger Tiers

- **Corridor Subscription:** Fixed monthly commute pass — guaranteed availability, no surge pricing, additional 10–15% discount
- **Pay-as-you-go (Tier 2):** Dynamic per-ride corridor pricing
- **On-demand (Tier 1):** Standard taxi pricing

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

```go
// internal/payments/kaspi.go
func ValidateWebhookSignature(body []byte, receivedSig string, key string) bool {
    mac := hmac.New(sha256.New, []byte(key))
    mac.Write(body)
    expectedSig := hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(expectedSig), []byte(receivedSig))
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
2. Go backend calculates driver payout immediately
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
3. Go backend records as `pending_reconciliation`
        │
        ▼
4. Weekly automated job:
   - Aggregates total cash collected by driver
   - Subtracts corridor guarantee subsidies owed TO the driver
   - Net balance applied against upcoming subscription fee
```

---

## 5. Corridor Performance Guarantee

For the first 90 days on any new corridor, BIRGE guarantees a minimum payment of **1.5× the solo ride rate** if the corridor run has fewer than 2 confirmed passengers. This removes driver financial risk during corridor ramp-up, directly addressing the core hesitation found in driver interviews.

---

## Related Files
- [[Security_and_Authentication.md]] — Kaspi webhook HMAC validation
- [[Ride_State_Machine.md]] — `completed` state triggers earnings calculation
- [[WebSocket_Hub_Architecture.md]] — real-time earnings broadcast to iOS
- [[Database_Schema_and_Migrations.md]] — payment_records schema
- [[Network_Effects_and_Growth.md]] — economic model and corridor density
