---
last_updated: 2026-04-22
---

# Backend Agent Context — BIRGE

> Читай этот файл перед любой Go/Backend задачей.

## Stack
| Component | Technology |
|---|---|
| API | Swift Vapor 4 |
| Database | PostgreSQL 16 (Fluent ORM) |
| Cache / RT | Redis 7 — OTP storage, JWT blacklist, rate limiting |
| Migrations | Fluent auto-migrations |
| Observability | Vapor Logger + Prometheus (Phase 2) |
| Infra | Docker, docker-compose (local), K3s (production) |

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

## Go Conventions
- Errors: `fmt.Errorf("rides.service.StartRide: %w", err)`
- Logging: `log.Info().Str("ride_id", id).Msg("ride started")` — zerolog
- No ORM — only raw `pgx` queries
- All handlers return: `c.JSON(status, gin.H{"data": ..., "error": nil})`
- Idempotency: all POST endpoints accept `X-Idempotency-Key` header

## Auth Flow
```
POST /auth/otp/request → Redis SET otp:<phone> <6digit> EX 300
POST /auth/otp/verify  → Redis GET, validate → issue JWT (60min) + Refresh (30d)
JWT claims: { user_id, role: "passenger"|"driver", exp }
Nginx validates JWT before requests reach Go — 401 never hits app code
```

## Ride FSM States
`requested → matched → driver_accepted → driver_arriving → passenger_wait → in_progress → completed`
PostgreSQL Advisory Lock guards state transitions — no double booking.
Full spec: [[Architecture/Ride_State_Machine]]

## WebSocket Hub
```
Redis Pub/Sub channel: ride:<ride_id>
All Go instances subscribe → fan out to connected clients
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
- [[Architecture/Backend_Architecture]] — Go монолит детально
- [[Architecture/Database_Schema_and_Migrations]] — schema, партиционирование
- [[Architecture/Payment_and_Financial_Architecture]] — Kaspi, ledger
- [[Architecture/Infrastructure_and_Deployment]] — K3s, ArgoCD
- [[Architecture/Open_Questions]] — проверь блокировки
