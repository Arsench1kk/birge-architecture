---
date: 2026-05-03
tags: [birge, task-complete, websocket, vapor, tca]
status: done
commit: cd8f2f65
branch: feature/websocket-ride-matched
---

# Task 2 Complete — WebSocket ride_matched → RideFeature

## Result
✅ Vapor: ws/ride/:rideId route + query-token JWT auth
✅ Vapor: ride-scoped WSHub + ride_matched broadcast
✅ iOS: WebSocketClient reconnect 3 attempts, 2s/4s/8s backoff
✅ iOS: SearchingFeature subscribes with real rideId + keychain token
✅ iOS: ride_matched → PassengerAppFeature → RideFeature
✅ iOS: RideMapView shows real driver info, preserved across REST recovery
✅ Committed: cd8f2f65 → feature/websocket-ride-matched pushed to origin

## Files Changed (14 files, +668 -113)
- BIRGECore/.../LiveWebSocketClient.swift — reconnect policy 2s/4s/8s
- BIRGECore/.../WebSocketClient.swift — interface cleanup
- BIRGEPassenger/App/PassengerAppFeature.swift — rideMatched delegate → RideFeature
- BIRGEPassenger/Features/Auth/KeychainClient.swift
- BIRGEPassenger/Features/Ride/RideFeature.swift — ride socket auth + Нет соединения banner
- BIRGEPassenger/Features/Searching/SearchingFeature.swift — WS connect on appear
- BIRGEPassenger/Features/Searching/SearchingView.swift
- BIRGEPassengerTests/SearchingFeatureTests.swift — new reducer tests
- BIRGEPassengerTests/PassengerAppFeatureTests.swift
- BIRGEPassengerTests/RideFeatureTests.swift
- birge-vapor/.../WSController.swift — ride-scoped channels
- birge-vapor/.../WSHub.swift — connect/disconnect/broadcast
- birge-vapor/Sources/App/routes.swift — debug/match-ride endpoint
- birge-vapor/Tests/AppTests/AuthTests.swift

## Verification
- swift test birge-vapor: ✅ passed
- xcodebuild build-for-testing generic/platform=iOS: ✅ passed
- Simulator test: ⚠️ blocked by SwiftNavigation/CasePathsCore linker issue (pre-existing)
- BIRGECore swift test: ⚠️ blocked by macOS platform mismatch (pre-existing)

## Known Blockers (pre-existing, not introduced)
- SwiftNavigation/CasePathsCore linker issue blocks simulator test run
- BIRGECore macOS platform mismatch blocks standalone swift test

## Phase Progress Update
| Phase | Before | After |
|---|---|---|
| 2 — WebSocket | 45% | 75% |
| 3 — Ride State Machine | 65% | 70% |

## Next Task
Task 3 — GET /auth/me + ProfileView real data
OR
Task 3 — Fix SwiftNavigation linker issue to unblock simulator tests

## [[Links]]
- [[Task1_RideAPI_2026-05-03]]
- [[Project_Status_2026-05-03]]
- [[Git_Commit_Push_Rules]]
