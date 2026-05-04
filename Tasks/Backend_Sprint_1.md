---
sprint: 1
status: active
last_updated: 2026-05-04
---

# Backend Sprint 1 — Foundation

> Goal: работающий API сервер с Auth, Rides FSM, WebSocket hub.
> Параллельно с iOS Sprint 1.

---

## 🟡 Ready to Start


### [BE-FUTURE-001] Auth onboarding state contract
- Status: PLANNED
- Ref: [[Future_Demo_Stability_and_Onboarding]]
- Goal: expose canonical per-user onboarding status in OTP/auth responses and persist passenger/driver completion data.
- Demo priority: new phone numbers should map to distinct users and Demo Hub should show where onboarding/profile data is stored.

### [BE-V002] Auth hardening
- Remove OTP logging
- Phone normalization to E.164
- OTP attempt counter in Redis (max 3 attempts)
- Refresh token revocation
- JWT_SECRET fail-fast if missing in prod

### [BE-V003] iOS ↔ Vapor DTO contract
- One canonical AuthResponseDTO (accessToken, role, userId)
- Match exactly between iOS AuthClient.swift and Vapor AuthDTO.swift
- iOS base URL config: local=localhost:8080, prod=api.birge.kz

### [BE-V004] iOS Keychain auth
- Replace UserDefaults token with Keychain
- KeychainClient TCA dependency
- AppFeature reads from Keychain on launch

### [BE-V005] Rides MVP
- POST /api/v1/rides (create)
- GET /api/v1/rides/mine (list)
- PATCH /api/v1/rides/:id/cancel
- Status transitions: requested → cancelled only for Phase 1
- DB indexes on passenger_id, status

### [BE-V006] WebSocket ride rooms
- Auth on connect (JWT query param)
- Register by userID + rideID
- Typed events: ride.updated, driver.location, ride.cancelled
- Ping/pong heartbeat

---

## ✅ Done

### [BE-V007] Defense Demo API + ride offer stability
- Status: DONE
- Completed: 2026-05-04
- Delivered:
  - Dev-only JWT-protected `GET /api/v1/demo/state`, `POST /api/v1/demo/seed`, `POST /api/v1/demo/reset`
  - Live table snapshots for users, rides, driver_profiles, corridors, corridor_bookings, passenger_subscriptions, payment_events, driver_location_records, driver_ride_decisions
  - Redis OTP/session/blacklist summaries without secret values
  - Deterministic AI corridor scoring explanation for defense demo
  - `RidesService.driverAccept` row lock to avoid double accept races
- Verification: `swift build` and `swift test` pass on 2026-05-04.


### [BE-V001] Fix Vapor build (JWT signer + Redis async) — Critical
- Status: DONE
- Completed: 2026-05-02
- Verification:
  - `cd /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/birge-vapor`
  - `swift build` → `Build complete!`
  - `swift run` → startup path verified; local boot stops on PostgreSQL auth (`password authentication failed for user "birge"`)
- Notes:
  - Swift 6 / Vapor 4 compatibility fixed for WebSocket handler, Redis future bridging, async `WebSocket.send`, and async app entrypoint lifecycle
