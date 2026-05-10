---
last_updated: 2026-05-08
---

# Backend Agent Context — BIRGE

> Читай этот файл перед любой Backend / Vapor задачей.

## Product Foundation

BIRGE is now defined as **transportation-as-a-subscription** for recurring Almaty commuters.

Backend work should preserve the current Vapor architecture, but new product features should wait until [[Business_Department/Product_Logic]] and [[Tasks/Business_Sprint_0_Foundation]] define:
- recurring route model
- monthly commute plan/subscription logic
- hub-to-hub corridor rules
- underfilled corridor fallback policy

On-demand rides remain useful for demo/fallback, but they are not the core product wedge.

Technical pivot plan: [[Tasks/Technical_Subscription_Pivot]].

Planned additive backend entities:

```text
RecurringRoute
MonthlyCommutePlan
CommutePlanRoute
```

Planned subscription plan IDs:

```text
solo_corridor
multi_corridor
flex_pack
```

Do not remove current ride/corridor/payment infrastructure; extend it.

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
| Status | `swift build`/`swift test` ✅; Docker Compose live stack restored for local app testing |

## Project Structure
```
birge-vapor/
├── Package.swift
├── Sources/App/
│   ├── entrypoint.swift
│   ├── configure.swift
│   ├── routes.swift
│   ├── Models/          { User, Driver, Ride, Corridor, PaymentEvent, PassengerSubscription, ... }
│   ├── Modules/
│   │   ├── Auth/        { AuthController, AuthService, AuthDTO }
│   │   ├── Rides/       { RidesController, RidesService, RidesDTO }
│   │   ├── Corridors/   { CorridorsController, CorridorsService, CorridorsDTO }
│   │   ├── Subscriptions/
│   │   ├── Payments/
│   │   ├── Locations/
│   │   ├── Drivers/
│   │   ├── Demo/        { dev-only defense/demo API }
│   │   └── WebSocket/   { WSController, WSHub }
│   ├── Migrations/      { users, driver profiles, rides, corridors, bookings, payments, locations }
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
- Dev-only Demo routes are registered only outside production.

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
Current WebSocket route: `/ws/ride/:rideId?token=<jwt>`
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
