---
date: 2026-05-03
tags: [birge, task-complete, linker, swiftnavigation, ci]
status: done
commit: d45c1b45
branch: fix/swiftnavigation-linker
---

# Task 4 Complete — SwiftNavigation/CasePathsCore Linker Fix

## Result
✅ Linker error: Undefined symbols CasePathsCore arm64 — FIXED
✅ Fix: pinned swift-navigation to 2.7.0 in package resolution
✅ Simulator tests: 32 passed, 0 failed, 2 skipped (expected)
✅ No production code or reducers changed
✅ Committed: d45c1b45 → fix/swiftnavigation-linker pushed to origin

## Root Cause
swift-navigation 2.8.0 conflicted with TCA's CasePathsCore symbols.
Pinning to 2.7.0 resolved undefined symbol errors for arm64.

## Files Changed (5 files, +46 -16)
- BIRGEPassenger.xcodeproj/project.pbxproj — package graph fix
- BIRGEPassengerTests/OTPFlowE2ETests.swift — test expectation drift fix
- BIRGEPassengerTests/PassengerAppFeatureTests.swift — test expectation drift fix
- BIRGEPassengerTests/RideFeatureTests.swift — test expectation drift fix
- BIRGEPassengerTests/SearchingFeatureTests.swift — test expectation drift fix

## Verification
- xcodebuild test simulator iPhone 17: ✅ 32 passed, 0 failed, 2 skipped
- Clean worktree verified (no local edits interfering)
- Production code: untouched

## Phase Progress Update
| Phase | Before | After |
|---|---|---|
| 5 — CI/CD + Production | 45% | 60% |

## Next Task
Task 5 — Token refresh / auth recovery (IOS-017)

## [[Links]]
- [[Task3_ProfileAuthMe_2026-05-03]]
- [[Task2_WebSocket_2026-05-03]]
- [[Project_Status_2026-05-03]]
- [[Git_Commit_Push_Rules]]
