---
status: active
version: v5.0
last_updated: 2026-04-21
phase: 1
---

# [[Security and Authentication]]

## 1. Identity Management — OTP + JWT

Authentication is strictly phone-number based, aligned with Kazakhstan market norms.

### Flow

```
1. User submits phone number
        │
        ▼
2. Go backend generates 6-digit OTP (crypto/rand)
   Stored in Redis with 5-minute TTL
        │
        ▼
3. OTP delivered via SMS (local KZ gateway)
        │
        ▼
4. User submits OTP
        │
        ▼
5. Backend validates OTP from Redis
        │
        ▼
6. JWT issued:
   - Access Token: 60-minute expiry
   - Refresh Token: 30-day expiry, single-use
        │
        ▼
7. iOS Keychain stores Refresh Token
   (hardware-protected secure enclave)
```

### JWT Middleware (Go)

```go
func JWTMiddleware(cfg *config.Config) gin.HandlerFunc {
    return func(c *gin.Context) {
        // Validate Bearer token
        token, err := jwt.ParseWithClaims(parts[1], &BIRGEClaims{}, keyFunc)
        if err != nil || !token.Valid {
            c.AbortWithStatusJSON(401, gin.H{"error_code": "INVALID_TOKEN"})
            return
        }
        // Check logout blacklist in Redis
        if blacklisted { ... }

        // Inject user_id and role — no DB lookup required
        c.Set("user_id", claims.UserID)
        c.Set("user_role", claims.Role)
        c.Next()
    }
}
```

Nginx validates JWT at the gateway before requests reach Go — malformed tokens never consume backend resources.

---

## 2. Driver KYC Pipeline

Four-stage verification (Kazakhstan regulatory requirement):

| Stage | Method | Automation |
|---|---|---|
| National ID upload | Document scan | OCR extraction |
| Facial similarity | Live selfie vs ID photo | ML model — flags < 85% confidence for manual review |
| Vehicle documents | Registration + insurance upload | Manual verification |
| Manual review | Operations team | For all cases flagged by automated stages |

See [[Open_Questions.md]] OQ-005 for the 85% threshold governance decision.

---

## 3. Rate Limiting — Redis Sliding Window

Sliding window prevents burst attacks at window boundaries (fixed window does not).

```go
// pkg/ratelimit/limiter.go
key := fmt.Sprintf("rl:%s:%s", c.FullPath(), userID)
now := time.Now().UnixMilli()
windowStart := now - window.Milliseconds()

pipe := rdb.Pipeline()
pipe.ZRemRangeByScore(ctx, key, "-inf", strconv.FormatInt(windowStart, 10))
pipe.ZAdd(ctx, key, redis.Z{Score: float64(now), Member: now})
pipe.ZCard(ctx, key)
pipe.Expire(ctx, key, window)
```

| Endpoint type | Limit |
|---|---|
| Passenger endpoints | 60 requests/minute |
| Driver location updates | 20 requests/minute (enforces 5s ping interval) |

---

## 4. Fraud Detection

| Vector | Detection | Response |
|---|---|---|
| GPS Spoofing | Impossible displacement check (>120 km/h since last ping) | 3 flags in 60s → automatic ride assignment pause |
| Phantom rides | Graph-based scoring: trajectory plausibility, driver-passenger matching frequency, payment patterns | Manual review triggered |
| Kaspi webhook forgery | HMAC-SHA256 signature validation with rotating API keys | Request rejected, alert fired |
| Payment replay | `ON CONFLICT (kaspi_transaction_id) DO NOTHING` in DB | Duplicate silently ignored |

---

## Related Files
- [[Infrastructure_and_Deployment.md]] — Nginx JWT validation, rate limiting
- [[Payment_and_Financial_Architecture.md]] — Kaspi webhook security
- [[Database_Schema_and_Migrations.md]] — payment_records idempotency
- [[Open_Questions.md]] — KYC threshold governance
