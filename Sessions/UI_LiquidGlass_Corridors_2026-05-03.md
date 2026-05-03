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
- `9f06a2a4` — feat(passenger): add subscriptions flow
- `fe208327` — feat(passenger): add commute setup onboarding
- `da9001cf` — feat(passenger): add ai explanation screen
- `e8a38820` — test(passenger): align otp flow tests with splash startup
- `cf9265e3` — fix(tests): unblock passenger package linking
- `51a890d7` — feat(ride): improve websocket recovery events
- `2fd2a124` — feat(corridors): connect passenger corridors to API
- `882230a1` — feat(passenger): add offer found confirmation flow
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
- P-08 `OfferFoundFeature/View` with countdown, Liquid Glass card, SF Symbols, and confirm/decline navigation.
- Vapor `Corridor` model, migration, controller/service/DTOs, seeded corridor list, and passenger booking endpoint.
- Passenger `APIClient` corridor DTOs plus live `GET /corridors` and `POST /corridors/:id/book` methods.
- Home, corridor list, and corridor detail screens now load live corridors with loading/error states.
- RideMap recovery banner now uses Liquid Glass, SF Symbols, spinner, and reconnect attempt context.
- `RideFeature` now accepts direct production lifecycle aliases: driver accepted/arriving/arrived, ride started/completed/cancelled.
- Package graph adjusted: `swift-case-paths` pinned to `1.5.6`, `swift-navigation` to `2.8.0`, and `CasePaths` / `Perception` / `PerceptionCore` are explicit target products to avoid Xcode 26 test-link gaps.
- `OTPFlowE2ETests` now matches the splash-first app startup: OTP flow tests start from unauthenticated state, and keychain restore is verified through `splashFinished`.
- P-23 AI explanation screen added as `AIExplanationFeature/View`: native SwiftUI scroll view with Liquid Glass hero/step/privacy cards, SF Symbols instead of the mockup robot/emoji, and CTA into corridor list.
- Home AI pill now opens AI explanation.
- P-03a–P-03e commute setup is now part of `OnboardingFeature/View`: origin, destination, times, weekdays, and final AI route summary.
- P-17/P-18 subscriptions flow added as `SubscriptionsFeature/View`: current plan, tier list, plan detail, comparison, local activation state, and Home subscription navigation.

## Verification
- ✅ `git diff --check` passed.
- ✅ `xcodebuild build -skipMacroValidation -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds.
- ✅ `RideFeatureTests` pass with `-skipMacroValidation`.
- ✅ `OTPFeatureTests` pass with `-skipMacroValidation`.
- ✅ `OTPFlowE2ETests` pass with `-skipMacroValidation`; live OTP success case is skipped unless `RUN_LIVE_OTP_E2E=1`.
- ✅ Full `BIRGEPassengerTests` pass with `-skipMacroValidation` on installed `iPhone 17 Pro` simulator.
- ✅ `PassengerAppFeatureTests` cover `Home → AI Explanation → Corridor List`.
- ✅ `OnboardingFeatureTests` cover commute setup paging, inputs, day selection, add-another-route, and finish delegate.
- ✅ `SubscriptionsFeatureTests` cover plan selection, activation, and detail dismissal.

## Next
1. Harden passenger backend integrations: payment/subscription API, corridor persistence policy, and richer booking UX.
2. Continue driver-side Sprint 1 gaps once passenger blocker/UI pass is stable.

## Agent Reminder
Before the next iOS UI task, read [[Context/Current_Focus]], [[docs/CLAUDE_for_mockups]], and the relevant `docs/mockups/` HTML.
