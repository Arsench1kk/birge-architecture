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
- `5bf3fcc1` — feat(rides): persist address labels
- `29903977` — feat(driver): add auth session flow
- `a12d75b6` — feat(driver): add ride assignment commands
- `be190fa7` — feat(driver): connect dashboard to profile api
- `c5012c77` — feat(driver): add driver profile api
- `85a66398` — feat(locations): broadcast driver updates
- `e778858b` — feat(driver): sync background locations
- `079ac13a` — feat(driver): add active ride lifecycle polish
- `136f91a0` — feat(driver): polish online offer surfaces
- `d83fec67` — feat(driver): add registration onboarding flow
- `98948bf4` — feat(payments): validate kaspi webhook signatures
- `061d6aaf` — feat(corridors): add passenger bookings list
- `17d44560` — feat(corridors): show passenger booking status
- `ab5237fe` — feat(corridors): persist passenger bookings
- `01955808` — feat(payments): add kaspi subscription handoff
- `50c3915f` — feat(subscriptions): connect passenger plans to api
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
- Vapor subscriptions API added: persisted `PassengerSubscription`, migration, authenticated overview endpoint, and activation endpoint.
- Passenger `APIClient` now exposes `fetchSubscriptions` and `activateSubscription`; `SubscriptionsFeature/View` uses live plans, active-since text, loading/error state, and activating CTA state.
- Vapor Payments module added for Kaspi checkout deep links plus append-only `payment_events` and idempotent webhook event insert.
- Passenger subscriptions now request Kaspi checkout before demo confirmation; the detail CTA shows a Liquid Glass Kaspi handoff card with SF Symbol and deep link.
- Corridor booking persistence added with `CorridorBooking`, migration, unique passenger/corridor constraint, and idempotent repeat booking response so seats are not decremented twice.
- Corridor detail UX now surfaces the successful booking state with a Liquid Glass confirmation card, booking id preview, refreshed seats/participants, and disabled repeat CTA.
- Passenger My Corridors list added: backend `/corridors/bookings`, iOS `APIClient.fetchCorridorBookings`, Liquid Glass booking cards, empty/error/loading states, and Home `Поездки` tab navigation.
- Kaspi webhook handling hardened with `KASPI_WEBHOOK_SECRET`, HMAC-SHA256 canonical payload validation, signature header fallback, and backend tests.
- Driver-side work started: BIRGEDrive now presents a 4-step registration onboarding flow before dashboard, covering D-03 personal info, D-03a vehicle, D-03b documents, and D-03c tier selection with Liquid Glass and SF Symbols.
- Driver D-05 online waiting and `driver/offer` surfaces polished in `DriverAppView`: map-style background, Liquid Glass navigation/working sheets, SF Symbol status icons, offer alert, countdown, AI match pill, route card, metrics grid, and passenger confirmation row.
- Driver D-07/D-08/D-09/D-10 lifecycle polished with active ride route sheet, pickup/boarding/in-progress states, boarding codes, passenger manifest, route progress, and completed ride summary/next-ride actions.
- Driver background GPS sync path added: active rides start `LocationClient`, completion/offline stops and syncs pending GRDB records, BIRGEDrive has background location plist keys, and Vapor stores authenticated location batches in `driver_location_records`.
- Location batches now broadcast the newest point as `ride.location_update` over `ride/<ride_id>` WebSocket channels for passenger map consumption.
- Backend Driver module added: authenticated `/api/v1/drivers/me` GET/PUT/POST persists D-03 registration/profile fields and `/api/v1/drivers/corridors/today` provides active corridor candidates plus estimated earnings for the driver dashboard.
- BIRGEDrive registration/dashboard now uses Driver API through `APIClient`: startup profile load, registration save, today corridors dashboard card, and no-token demo fallback until driver auth/session is implemented.
- Driver ride assignment/commands added: backend driver offers, accept/arrived/start/complete endpoints, canonical `ride.status_changed` WebSocket broadcast, and BIRGEDrive polling plus lifecycle command wiring.
- BIRGEDrive driver auth/session flow added: email login/register, shared API token storage, auth gate before registration/dashboard, and removal of no-token registration/offer fallback.
- Ride address labels persisted end-to-end: passenger request strings are sent to Vapor, stored on rides, returned in ride DTOs, and used by driver offers.
- Driver accepted-ride navigation polish added in `DriverAppView`: Liquid Glass maneuver cue, active navigation panel, ETA/speed/safety chips, route phase text, and SF Symbol direction indicators over the map.
- Full live backend Docker stack restored for Simulator testing: Swift 6.0 Docker image, Linux-compatible `swift-crypto` for Kaspi signatures, release-friendly Postgres configuration, `.dockerignore`, and Compose DB/Redis ports kept internal.

## Verification
- ✅ `git diff --check` passed.
- ✅ `xcodebuild build -skipMacroValidation -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds.
- ✅ `RideFeatureTests` pass with `-skipMacroValidation`.
- ✅ `OTPFeatureTests` pass with `-skipMacroValidation`.
- ✅ `OTPFlowE2ETests` pass with `-skipMacroValidation`; live OTP success case is skipped unless `RUN_LIVE_OTP_E2E=1`.
- ✅ Full `BIRGEPassengerTests` pass with `-skipMacroValidation` on installed `iPhone 17 Pro` simulator.
- ✅ `PassengerAppFeatureTests` cover `Home → AI Explanation → Corridor List`.
- ✅ `OnboardingFeatureTests` cover commute setup paging, inputs, day selection, add-another-route, and finish delegate.
- ✅ `SubscriptionsFeatureTests` cover API loading, plan selection, activation, and detail dismissal.
- ✅ Vapor `swift build` succeeds after subscriptions API integration.
- ✅ Vapor `swift build` succeeds after Payments/Kaspi handoff integration.
- ✅ Vapor `swift build` succeeds after corridor booking persistence.
- ✅ `CorridorDetailFeatureTests` cover join success, booking id storage, and refreshed corridor state.
- ✅ `MyCorridorsFeatureTests` cover booking history loading and corridor selection.
- ✅ Vapor `swift test` covers Kaspi signature accept/reject behavior.
- ✅ `BIRGEDrive` build passes after driver registration onboarding integration.
- ✅ `BIRGEDrive` build passes after driver online waiting / offer polish.
- ✅ `BIRGEDrive` build passes after driver active ride lifecycle polish.
- ✅ `BIRGEDrive` and `BIRGEPassenger` build after driver background GPS sync work.
- ✅ Vapor `swift build` and `swift test` pass after `/locations/bulk` integration.
- ✅ Vapor `swift build`, Vapor `swift test`, and `BIRGEPassenger` build pass after location WebSocket broadcast.
- ✅ Vapor `swift build` and `swift test` pass after driver profile API integration.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after Driver API iOS hookup.
- ✅ Vapor `swift build`, Vapor `swift test`, `BIRGEDrive` build, and `BIRGEPassenger` build pass after driver ride assignment/commands.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after driver auth/session flow.
- ✅ Vapor `swift build`, Vapor `swift test`, `BIRGEPassenger` build, and `BIRGEDrive` build pass after ride address labels.
- ✅ `git diff --check` and `BIRGEDrive` build pass after driver navigation guidance polish.
- ✅ `swift build`, `swift build -c release`, `docker compose build vapor`, `docker compose up -d postgres redis vapor`, and local OTP request to `localhost:8080` pass after Docker live stack fixes.

## Next
1. Run Passenger and Drive in Simulator against the now-running live backend.
2. Replace current driver guidance placeholders with persisted route geometry / live MapKit directions.
3. Add persisted named pickup/destination inputs beyond current fixed coordinates.
4. Later: replace demo Kaspi deep link with real merchant API contract when credentials/spec are available.

## Agent Reminder
Before the next iOS UI task, read [[Context/Current_Focus]], [[docs/CLAUDE_for_mockups]], and the relevant `docs/mockups/` HTML.
