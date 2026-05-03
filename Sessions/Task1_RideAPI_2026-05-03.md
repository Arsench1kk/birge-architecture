---
date: 2026-05-03
tags: [birge, task-complete, ride-api, vapor, tca]
status: done
commit: 8d7af6d8
branch: feature/ride-request-api
---

# Task 1 Complete — POST /rides Real API

## Result
✅ Vapor: POST /api/v1/rides → 201 + real ride_id
✅ iOS: RideRequestFeature calls real API
✅ iOS: PassengerAppFeature receives real ride_id
✅ iOS: SearchingFeature uses real ride_id
✅ PostgreSQL: ride row created (1b398fbd-520d-4b33-a92e-417e2e27591e, requested)
✅ Committed: 8d7af6d8 → feature/ride-request-api

## Files Changed (12 files, +423 -170)
- BIRGECore/Sources/BIRGECore/Models/RideModels.swift — new shared DTOs
- BIRGECore/Sources/BIRGECore/Network/APIClient.swift — fatalError → real URLSession
- BIRGEPassenger/Features/RideRequest/RideRequestFeature.swift — calls POST /rides
- BIRGEPassenger/App/PassengerAppFeature.swift — receives real ride_id
- BIRGEPassenger/Features/Searching/SearchingFeature.swift — uses real ride_id
- BIRGEPassenger/Features/Searching/SearchingView.swift — UI updates
- birge-vapor/Sources/App/Modules/Rides/RidesDTO.swift
- birge-vapor/Sources/App/Modules/Rides/RidesService.swift — saves to PostgreSQL
- birge-vapor/Sources/App/Modules/Rides/RidesController.swift
- birge-vapor/Sources/App/Models/Ride.swift
- birge-vapor/Sources/App/Migrations/AddRideRequestAPIFields.swift
- birge-vapor/Sources/App/configure.swift

## Environment
- Vapor running: localhost:8080
- PostgreSQL + Redis containers: running
- Verified: swift build + xcodebuild + live OTP + POST /rides + Postgres query

## Phase Progress Update
| Phase | Before | After |
|---|---|---|
| 1 — Auth + OTP | 75% | 80% |
| 3 — Ride State Machine | 50% | 65% |

## Next Task
Task 2 — WebSocket ride_matched
Branch: feature/websocket-ride-matched
Goal: SearchingFeature subscribes to ws/ride/<rideId>, handles ride_matched → RideFeature

## [[Links]]
- [[Project_Status_2026-05-03]]
- [[UI_Design_System_2026-05-02]]
- [[Git_Commit_Push_Rules]]
