---
sprint: 1
status: active
last_updated: 2026-04-22
---

# Backend Sprint 1 — Foundation

> Goal: работающий API сервер с Auth, Rides FSM, WebSocket hub.
> Параллельно с iOS Sprint 1.

---

## 🟡 Ready to Start

### [BE-001] Project Scaffold
- `go mod init github.com/birge/api`
- Структура папок (internal/, cmd/, pkg/, migrations/)
- Dockerfile (distroless, ~12MB)
- docker-compose.yml (postgres, redis, app)
- `.env.example` со всеми переменными

### [BE-002] Database Migrations
- `001_users.sql` — users, driver_profiles, passenger_profiles
- `002_rides.sql` — rides, ride_events (append-only FSM log)
- `003_payments.sql` — payment_events (append-only ledger)
- golang-migrate setup
**Architecture ref:** [[Architecture/Database_Schema_and_Migrations]]

### [BE-003] Auth Module
- OTP request/verify endpoints
- Redis OTP storage (5min TTL)
- JWT issue (access 60min, refresh 30d)
- JWT middleware
**Architecture ref:** [[Architecture/Security_and_Authentication]]

### [BE-004] Rides Module + FSM
- Ride CRUD endpoints
- FSM transitions с PostgreSQL Advisory Locks
- Transactional Outbox для WebSocket events
**Architecture ref:** [[Architecture/Ride_State_Machine]]

### [BE-005] WebSocket Hub
- Hub + Client структуры
- Redis Pub/Sub для multi-instance broadcast
- Ping/pong + reconnect handling
**Architecture ref:** [[Architecture/WebSocket_Hub_Architecture]]

---

## ✅ Done

*(пусто)*
