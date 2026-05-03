---
date: 2026-05-03
tags: [birge, task-complete, auth, token-refresh, ios-017]
status: done
commit: 7714b0f9
branch: feature/token-refresh-ios017
---

# Task 5 Complete — Token Refresh + Auth Recovery (IOS-017)

## Result
✅ Vapor: POST /auth/refresh implemented
✅ Vapor: OTP verify returns both accessToken + refreshToken
✅ iOS: KeychainClient stores refresh token (birge_refresh_token)
✅ iOS: APIClient retries on 401 with refresh, one-flight coordinated
✅ iOS: AppFeature handles authExpired → clears keychain → OTP screen
✅ iOS: Simulator tests: 36 passed, 0 failed, 2 skipped
✅ Committed: 7714b0f9 → feature/token-refresh-ios017 pushed to origin

## Key Implementation Details
- Concurrent 401 requests: one-flight actor serializes refresh (no duplicate calls)
- Retry policy: 401 → refresh → retry once → on failure emit authExpired
- Toast on session expiry: "Сессия истекла. Войдите снова."
- Vapor JWTMiddleware: returns 401 (not 500) on expired token
- Vapor tests: 5 passed

## Files Changed (11 files, +670 -100)
- BIRGECore/.../APIClient.swift — refresh interceptor + retry
- BIRGECore/.../AuthSessionClient.swift — new auth session event client
- BIRGEPassenger/App/AppFeature.swift — authExpired → OTP navigation
- BIRGEPassenger/App/AppView.swift — subscription wiring
- BIRGEPassenger/Features/Auth/KeychainClient.swift — refresh token storage
- BIRGEPassenger/Features/Auth/OTPFeature.swift — saves both tokens
- BIRGEPassengerTests/APIClientRefreshTests.swift — new tests
- BIRGEPassengerTests/AppFeatureTests.swift — authExpired tests
- BIRGEPassengerTests/OTPFeatureTests.swift — dual token save tests
- birge-vapor/.../JWTMiddleware.swift — 401 on expiry
- birge-vapor/Tests/AppTests/AuthTests.swift

## Verification
- swift test birge-vapor: ✅ 5 passed
- xcodebuild test simulator iPhone 17: ✅ 36 passed, 0 failed, 2 skipped
- Clean worktree used: unrelated workspace edits untouched

## Phase Progress Update
| Phase | Before | After |
|---|---|---|
| 1 — Auth + OTP | 90% | 100% |
| 3 — Ride State Machine | 70% | 75% |

## Next Session
- Merge all feature branches → main via PRs
- Task 6: Connection-loss banner "Нет соединения" (visible in UI)
- Task 7: Payment UI / Kaspi state
- Task 8: Driver background GPS tracking (IOS-018)

## [[Links]]
- [[Task4_LinkerFix_2026-05-03]]
- [[Task3_ProfileAuthMe_2026-05-03]]
- [[Task2_WebSocket_2026-05-03]]
- [[Task1_RideAPI_2026-05-03]]
- [[Project_Status_2026-05-03]]
- [[Git_Commit_Push_Rules]]
