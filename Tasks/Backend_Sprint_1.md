---
sprint: 1
status: active
last_updated: 2026-05-02
---

# Backend Sprint 1 — Foundation

> Goal: работающий API сервер с Auth, Rides FSM, WebSocket hub.
> Параллельно с iOS Sprint 1.

---

## 🟡 Ready to Start

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

### [BE-V001] Fix Vapor build (JWT signer + Redis async) — Critical
- Status: DONE
- Completed: 2026-05-02
- Verification:
  - `cd /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/birge-vapor`
  - `swift build` → `Build complete!`
  - `swift run` → startup path verified; local boot stops on PostgreSQL auth (`password authentication failed for user "birge"`)
- Notes:
  - Swift 6 / Vapor 4 compatibility fixed for WebSocket handler, Redis future bridging, async `WebSocket.send`, and async app entrypoint lifecycle
