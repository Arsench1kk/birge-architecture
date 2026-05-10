---
status: active
version: v5.0
last_updated: 2026-05-05
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
2. Backend generates 6-digit OTP with a cryptographically secure random generator
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

### JWT Middleware (Swift Vapor)

```swift
// Sources/App/Middleware/JWTMiddleware.swift
struct JWTMiddleware: AsyncMiddleware {
    func respond(to request: Request, chainingTo next: AsyncResponder) async throws -> Response {
        let token = try bearerToken(from: request)
        let payload = try request.jwt.verify(token, as: BIRGEJWTPayload.self)

        // Future hardening: check logout blacklist in Redis.
        request.auth.login(AuthContext(
            userID: payload.userID,
            role: payload.role
        ))

        return try await next.respond(to: request)
    }
}
```

Nginx can validate JWT at the gateway later; Phase 1 canonical validation lives in Vapor middleware.

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

```swift
// Sources/App/Middleware/RateLimitMiddleware.swift
let key = "rl:\(request.url.path):\(userID)"
let now = Date().timeIntervalSince1970 * 1000
let windowStart = now - windowMilliseconds

try await request.redis.zremrangebyscore(key, min: "-inf", max: "\(windowStart)")
try await request.redis.zadd(key, items: [(score: now, value: "\(now)")])
let count = try await request.redis.zcard(key)
try await request.redis.expire(key, after: .seconds(windowSeconds))
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
