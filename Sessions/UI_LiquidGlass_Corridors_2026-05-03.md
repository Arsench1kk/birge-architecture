---
date: 2026-05-03
tags: [birge, ios, passenger, liquid-glass, mockups, corridors]
status: active
branch: feature/passenger-liquid-glass-ui
---

# Passenger Liquid Glass + Corridors — 2026-05-03

## Summary
Implemented and pushed the first major passenger UI pass from the final mockups. The SwiftUI implementation should use the mockups as product direction, then improve the result with native platform behavior, Liquid Glass, and SF Symbols.

## Commits pushed
- `9a58800a` — feat(core): add liquid glass design components
- `dcbdf02c` — feat(passenger): add splash and onboarding flow
- `aa5e1da3` — feat(passenger): rebuild home with corridor previews
- `642f0127` — feat(passenger): polish active ride boarding code
- `6700e06c` — feat(passenger): refresh ride completion experience
- `6f074e02` — feat(core): use native glass effect when available
- `55732eb7` — refactor(passenger): replace ride emoji with sf symbols
- `f1150b11` — fix(passenger): resolve onboarding and boarding build issues
- `a9d12867` — feat(passenger): add corridor list and detail flow

## Implemented
- Native Liquid Glass support in `BIRGECore` for iOS 26+, with material fallback.
- Passenger Splash and Onboarding screens.
- Rebuilt Home with AI match/corridor previews.
- BoardingCode and RideComplete UI polish.
- Ride/RideRequest SF Symbol cleanup instead of mockup emoji.
- Corridor list and corridor detail TCA features/views.
- Home → corridor list/detail navigation in `PassengerAppFeature`.

## Verification
- ✅ `git diff --check` passed.
- ✅ `xcodebuild build -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds.
- ⚠️ `xcodebuild test` is still blocked before test execution by `SwiftNavigation.framework` linker errors for `CasePathsCore.CasePathable` / `AnyCasePath`.

## Next
1. Build P-08 `OfferFoundView`.
2. Implement Vapor `/api/v1/corridors` and connect corridor UI to real API data.
3. Add RideMap disconnection banner and remaining ride lifecycle events.
4. Fix or isolate the `SwiftNavigation` / `CasePathsCore` test linker blocker.

## Agent Reminder
Before the next iOS UI task, read [[Context/Current_Focus]], [[docs/CLAUDE_for_mockups]], and the relevant `docs/mockups/` HTML.
