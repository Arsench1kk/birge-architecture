---
last_updated: 2026-05-02
---

# Backend Agent Context — BIRGE

> Читай этот файл перед любой Backend / Vapor задачей.

## Stack
| Component | Technology |
|---|---|
| API | Swift Vapor 4 |
| ORM | Fluent (PostgreSQL driver) |
| Cache | Redis 7 — OTP, JWT blacklist, rate limiting |
| Migrations | Fluent migrations (autoMigrate local, manual prod) |
| Auth | JWT (HS256) + OTP via Redis |
| Infra | Docker + docker-compose local, K3s prod |
| API prefix | /api/v1 (canonical) |
| Status | `swift build` ✅ on 2026-05-02; `swift run` reaches startup and requires valid PostgreSQL credentials to finish boot |

## Project Structure
```
birge-vapor/
├── Package.swift
├── Sources/App/
│   ├── entrypoint.swift
│   ├── configure.swift
│   ├── routes.swift
│   ├── Models/          { User.swift, Driver.swift, Ride.swift }
│   ├── Modules/
│   │   ├── Auth/        { AuthController, AuthService, AuthDTO }
│   │   ├── Rides/       { RidesController, RidesService, RidesDTO }
│   │   └── WebSocket/   { WSController, WSHub }
│   ├── Migrations/      { CreateUsers, CreateDriverProfiles, CreateRides }
│   ├── Middleware/      { JWTMiddleware, RateLimitMiddleware }
│   └── Support/         { AuthContext }
└── docker-compose.yml
```

## Vapor Conventions
- Modules live under `Sources/App/Modules/`
- DTOs live next to their module (`AuthDTO`, `RidesDTO`)
- Routing is registered in `routes.swift`
- Structured logging uses Vapor `Logger`
- Database access goes through Fluent models + migrations
- Auth middleware lives in `Sources/App/Middleware/JWTMiddleware.swift`
- Local E2E-only OTP logging currently exists in `Modules/Auth/AuthService.swift` and writes `/tmp/birge-otp.log`
- Idempotency is still a business rule for payment operations even where HTTP headers are not implemented yet

## Auth Flow
```
POST /auth/otp/request → Redis SET otp:<phone> <6digit> EX 300
POST /auth/otp/verify  → Redis GET, validate → issue JWT (60min) + Refresh (30d)
JWT claims: { user_id, role: "passenger"|"driver", exp }
`GET /auth/me` reads the Bearer token and returns the current user
```

## Ride FSM States
`requested → matched → driver_accepted → driver_arriving → passenger_wait → in_progress → completed`
PostgreSQL Advisory Lock guards state transitions — no double booking.
Full spec: [[Architecture/Ride_State_Machine]]

## WebSocket Hub
```
Redis Pub/Sub channel: ride:<ride_id>
All Vapor instances subscribe → fan out to connected clients
Ping from iOS: every 5s → backend responds with pong
GPS update interval: 10s (driver_arriving), 5s (in_progress)
```
Full spec: [[Architecture/WebSocket_Hub_Architecture]]

## Kaspi Payment Integration
- Webhook: `POST /payments/kaspi/webhook`
- Idempotency key: `kaspi_transaction_id` (OQ-002 pending — may need composite key)
- Ledger: append-only `payment_events` table, balance computed from events
- Never mutate payment records — only insert

## Files for deep dive
- [[Architecture/Backend_Architecture]] — Vapor modular monolith detail
- [[Architecture/Database_Schema_and_Migrations]] — schema, партиционирование
- [[Architecture/Payment_and_Financial_Architecture]] — Kaspi, ledger
- [[Architecture/Infrastructure_and_Deployment]] — K3s, ArgoCD
- [[Architecture/Open_Questions]] — проверь блокировки
