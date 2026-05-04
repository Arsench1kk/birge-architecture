---
status: active
created: 2026-05-04
project: BIRGE
branch: feature/passenger-liquid-glass-ui
focus: UI rescue + navigation cleanup
---

# UI + Navigation Rescue Audit

## Why This Exists
We are pausing new feature expansion and redirecting the current development lane toward view quality, native SwiftUI patterns, Liquid Glass consistency, and navigation sanity. The project is functionally strong enough for demo, but many screens currently feel hand-built, hardcoded, oversized, or visually inconsistent.

Core principle: mockups give product intent; SwiftUI implementation should be more native, calmer, more adaptive, and easier to maintain than the mockup.

## Current Snapshot
App repo: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp`

Largest files found during audit:
- `BIRGEDrive/App/DriverAppView.swift` — 1155 lines.
- `BIRGEPassenger/Features/Auth/OnboardingView.swift` — 698 lines.
- `BIRGEPassenger/Features/Ride/RideMapView.swift` — 575 lines.
- `BIRGEPassenger/Features/ProjectDemo/ProjectDemoView.swift` — 402 lines.
- `BIRGEPassenger/Features/Subscriptions/SubscriptionsView.swift` — 437 lines.
- `BIRGEDrive/App/DriverAppFeature.swift` — 659 lines.
- `BIRGEDrive/Features/DriverRegistrationView.swift` — 463 lines.

High-risk patterns observed:
- Many screens manually compose map-like backgrounds, route lines, cards, pills, and status components instead of reusing a shared component set.
- `DriverAppView` owns too many screen states: online waiting, offers, active ride, route guidance, completion, dashboards, metrics, and decorative map background.
- Passenger navigation is mostly one global `StackState<Path.State>` with many manual `append`, `removeLast`, and `removeAll` transitions.
- Driver root navigation mixes auth, registration, dashboard, ride lifecycle, and earnings in one root store.
- Several legacy/stub/demo states are still present near production paths.
- Fixed sizes and decorative layout are common: `frame(width:)`, `frame(height:)`, `offset`, `GeometryReader`, manual `Path`, and repeated gradients.

## P0: Navigation Rescue
This is the first structural fix. The UI cannot feel stable while routing is ad hoc.

### Problems
- `PassengerAppFeature` handles too many unrelated navigation edges in one reducer.
- Ride flow navigation is implicit stack surgery:
  - `RideRequest -> Searching` appends.
  - `Searching -> OfferFound` appends.
  - `OfferFound -> Ride` removes last, then appends.
  - decline/expiry/cancel often `removeAll`.
- Root auth/onboarding state currently depends on token presence and local root cases; this will conflict with future per-user onboarding state.
- Driver root switches between auth, registration, and dashboard with direct booleans rather than a clear route/state machine.
- No central route vocabulary for modal vs push vs root replacement.

### Target Shape
- Introduce explicit route/state concepts:
  - Passenger root: `splash`, `unauthenticated`, `needsOnboarding`, `main`.
  - Passenger main tabs/sections: `home`, `rides`, `profile`, `demo` as stable destinations.
  - Ride flow: dedicated child coordinator/state machine instead of raw stack mutation.
  - Driver root: `auth`, `registration`, `dashboard`, `activeRide` with explicit transitions.
- Keep `NavigationStack` for browse flows such as corridors/subscriptions/profile.
- Use sheet/full-screen presentation intentionally for transient flows such as offer found, confirm/cancel, and destructive actions.
- Add navigation reducer tests for each major flow.

### First Tasks
- Create `PassengerRoute` / `PassengerDestination` naming rules and document push vs modal vs root replacement.
- Split ride flow navigation out of `PassengerAppFeature` into a `PassengerRideCoordinatorFeature` or equivalent local pattern.
- Replace `removeAll()` flow resets with named actions like `returnHomeAfterRideCancel`, `finishRideFlow`, `restartRideSearch`.
- Add logout/root reset tests.
- Add driver root state enum instead of controlling the root through multiple booleans.

## P0: Design System Rescue
### Problems
- `BIRGECore` has useful primitives (`liquidGlass`, `BIRGEPrimaryButton`, `BIRGEGlassSheet`), but screens still repeatedly rebuild their own cards, route rows, icon chips, map overlays, and status pills.
- Button styles are mostly title-only; icon buttons and toolbar actions are often custom one-offs.
- `BIRGEPrimaryButton`/secondary/destructive still use fixed `54` height and older corner/shadow patterns, not a complete control family.
- The layout token set is too small for complex screens; this causes local hardcoding.
- Fonts use system styles, which is good, but some screens still use raw `.font(.headline)`, `.font(.caption)`, `.font(.system(size:))` without a clear reason.

### Target Shape
- Build a reusable SwiftUI component kit in `BIRGECore` before polishing screens:
  - `BIRGEIconButton`
  - `BIRGEToolbarButton`
  - `BIRGETextFieldRow`
  - `BIRGEListRow`
  - `BIRGESectionHeader`
  - `BIRGEEmptyState`
  - `BIRGEErrorState`
  - `BIRGELoadingState`
  - `BIRGEMapOverlayButton`
  - `BIRGERouteSummaryCard`
  - `BIRGERideStatusSheet`
  - `BIRGEOfferCard`
- Make repeated glass containers visually consistent and avoid nested cards inside cards.
- Use SF Symbols consistently for commands and states.
- Prefer system materials/toolbars/forms where they produce a more native result.

## P0: Driver UI Rescue
### Problems
- `DriverAppView.swift` is the largest UI risk: 1155 lines and too many responsibilities.
- It uses a fake map-like decorative background instead of a clear operational driver dashboard or real MapKit surface.
- Online/offline dashboard, offer alert, offer card, active ride sheet, route guidance, completed ride sheet, progress bar, passenger manifest, and status text helpers are all in one file.
- Driver logout/account switching is missing from visible UI, which hurts demo and QA.
- `DriverAppFeature` still contains mock earnings and comments like “simulate offer arriving” near live polling.

### Target Shape
Split driver UI into dedicated views:
- `DriverDashboardView`
- `DriverMapBackgroundView` or real `DriverMapView`
- `DriverTopBar`
- `DriverOnlineControlSheet`
- `DriverOfferBanner`
- `DriverOfferCard`
- `DriverActiveRideView`
- `DriverRouteGuidanceView`
- `DriverCompletedRideView`
- `DriverStatusComponents`

First improvement path:
1. Add visible driver account/logout menu.
2. Extract components without changing behavior.
3. Replace fake map decoration with either real MapKit or a simpler native dashboard surface.
4. Normalize active ride states and actions visually.
5. Rebuild driver registration with native form-like composition while keeping Liquid Glass accents.

## P0: Passenger Ride Flow Rescue
### Problems
- `RideRequest`, `Searching`, `OfferFound`, and `RideMap` do not yet feel like one coherent native ride flow.
- `RideMapView` is 575 lines and mixes map content, connection state, ride state sheets, driver info, cancellation, and styling helpers.
- Offer confirmation is pushed as a full navigation destination; it may be better as a sheet/overlay owned by the ride search flow.
- Some UI states use fixed sizes and manual decorations where native components would be more stable.

### Target Shape
- One coherent passenger ride flow visual language:
  - native MapKit background when map is important;
  - stable top overlay for connection/status;
  - glass bottom sheet for state-specific ride controls;
  - system sheet for confirmations/cancel reasons;
  - no random full-screen route jumps for transient offer states.
- Extract `RideMapView` into:
  - `RideMapContainer`
  - `RideStatusBanner`
  - `RideConnectionBanner`
  - `RideDriverAnnotation`
  - `RideBottomSheet`
  - `RideActionRow`
  - `RideCancelSheet`

First improvement path:
1. Polish `RideRequestView` as a real native request sheet.
2. Make `SearchingView` a compact live state overlay, not a disconnected screen from the map flow.
3. Convert `OfferFoundView` to a polished confirmation surface.
4. Split `RideMapView` into small state-specific subviews.

## P1: Passenger Home + Discovery Screens
### Problems
- Home includes fixed corridor card widths (`220`) and map/search overlays that can feel non-native on different screen sizes.
- Corridor, AI, subscriptions, and demo screens each invented their own section/card language.
- Demo Hub is valuable for defense, but should look like an instructor-facing diagnostics console, not a marketing screen.

### Target Shape
- Home should become a calm native dashboard:
  - real map or clear location surface;
  - stable bottom action area;
  - segmented sections for commute/corridors/subscription/demo;
  - consistent card width behavior through `LazyVGrid`/adaptive layouts.
- Corridor/subscription/AI screens should share list/detail/section components.
- Demo Hub should use dense, scannable diagnostic rows with proper refresh/seed/reset toolbar controls.

## P1: Onboarding Rescue
### Problems
- `OnboardingView.swift` is 698 lines and mixes intro slides, commute setup, route art, grid drawing, progress, buttons, and field components.
- It uses manual `Path`/grid drawing and large fixed hero areas, which can break on smaller devices or Dynamic Type.
- Future per-user onboarding work will need cleaner separation between passenger onboarding and driver onboarding.

### Target Shape
- Split onboarding into composable pages:
  - intro pages;
  - commute origin/destination;
  - schedule/time;
  - weekdays;
  - AI route summary;
  - completion.
- Use reusable form rows, chips, segmented controls, and native safe-area-aware CTA footer.
- Keep a single source of truth for progress and validation.

## P1: Driver Registration Rescue
### Problems
- `DriverRegistrationView.swift` is 463 lines and mixes auth-like field components, profile steps, vehicle color selector, documents, tier cards, and footer buttons.
- It can be more native as a stepper/form flow.

### Target Shape
- Use `Form`/`ScrollView` intentionally with custom glass only where it helps.
- Split into `PersonalDataStep`, `VehicleStep`, `DocumentsStep`, `TierStep`.
- Make validation messages clear and persistent.
- Show save/loading/error states in a consistent footer.

## P2: Legacy/Demo Cleanup
Keep demo features, but remove ambiguity around production paths.

Findings:
- `ActiveRideFeature/View` is still present as DEBUG legacy simulation.
- `CorridorOption.mock`, `MockDriver.mock`, and mock driver earnings are still used in some initial states.
- Comments in `PassengerAppFeature` still say “stub” for routes that are no longer stub-like.
- `DriverAppFeature` has `DriverEarnings.mock` and `simulate offer` wording near live offer polling.

Cleanup target:
- Rename demo-only state explicitly.
- Make fallback/mock data impossible to confuse with live production data.
- Keep previews/sample states isolated in `#Preview` or `PreviewSupport` files.

## Visual QA Requirements
Every UI rescue slice must include:
- `xcodebuild build` for `BIRGEPassenger` or `BIRGEDrive` as relevant.
- At least two simulator screenshots: compact iPhone and large iPhone.
- Check text fitting, safe areas, keyboard behavior, Dynamic Type sanity, and dark/light mode if time allows.
- For map screens: verify map is visible, overlays do not block primary controls, and bottom sheets do not hide essential content.
- For navigation changes: reducer tests for route transitions.

## Recommended Execution Order
1. Navigation vocabulary and route cleanup document/code skeleton.
2. Design system component kit expansion in `BIRGECore`.
3. DriverAppView extraction and logout/menu addition.
4. Passenger ride flow rescue: RideRequest -> Searching -> OfferFound -> RideMap.
5. Home/dashboard polish.
6. Onboarding split and polish.
7. Demo Hub dense diagnostics polish.
8. Subscriptions/corridors consistency pass.

## Acceptance Criteria For This Rescue Lane
- No main production view file should remain above roughly 300-350 lines unless there is a strong reason.
- Navigation transitions are named and tested, not anonymous stack surgery.
- Driver and Passenger root flows have clear state machines.
- Repeated UI primitives live in `BIRGECore`, not copied across screens.
- Screens use SF Symbols and native SwiftUI controls where available.
- UI works on compact and large iPhone simulators without overlapping text or incoherent bottom sheets.
- Demo flow looks visually credible enough to show without apologizing for the UI.
