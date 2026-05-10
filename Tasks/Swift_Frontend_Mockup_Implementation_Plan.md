---
status: active
version: v1.0
last_updated: 2026-05-10
owner: Arsen + Codex planner
scope: frontend_only_swiftui
---

# Swift Frontend Mockup Implementation Plan

This is the foundation for transferring the final BIRGE Open Design mockups into native SwiftUI frontend code before backend implementation resumes.

## 0. Core Decision

Build the iOS frontend first as a real native app shell with mock data and stateful flows.

Do not build a WebView wrapper around HTML mockups.
Do not wire real payments, real document upload, or new backend endpoints in this lane.
Do not hardcode product data directly inside SwiftUI view bodies.

The goal is:

```text
final Open Design prototype -> native SwiftUI/TCA frontend -> later backend contracts
```

## 1. Source Of Truth

App repo:

```text
/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp
```

Xcode project:

```text
/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger/BIRGEPassenger.xcodeproj
```

Swift targets:

```text
BIRGEPassenger
BIRGEDrive
BIRGECore
```

Final passenger mockups:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger-polished
```

Passenger gallery:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger-polished/index.html
```

Final driver mockups:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver
```

Driver gallery:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver/index.html
```

Business source of truth:

```text
Context/Universal_AI_Handoff_BIRGE.md
Business_Department/Product_Logic.md
Business_Department/Subscription_Strategy.md
Business_Department/Driver_Economics.md
Business_Department/Driver_Utilization_Risk.md
Business_Department/Legal_Compliance_Risk_Assessment.md
docs/Passenger_Product_Mockup_Roadmap.md
docs/Driver_Product_Mockup_Roadmap.md
```

## 2. Product Logic To Lock Before Coding

BIRGE is not taxi-first.

BIRGE is transportation-as-a-subscription for recurring Almaty commuters.

Passenger core:

- monthly recurring commute routes
- passenger enters/searches home/work addresses first
- BIRGE suggests nearby pickup/dropoff nodes after address input
- MVP route model: hub-to-hub, one pickup node, one dropoff node, no intermediate stops
- on-demand taxi is fallback/acquisition only, not the main home action
- passenger MVP launches with all three tiers: `solo_corridor`, `multi_corridor`, `flex_pack`
- `multi_corridor` is the recommended/default passenger plan
- Solo and Multi Corridor must not show per-ride pricing
- Flex Pack may use ride-pack language
- fallback taxi may show per-ride estimate because it is outside subscription

Driver core:

- driver app is an operating system for planned corridor work, not a taxi marketplace dashboard
- driver monetization is fixed monthly platform subscription
- zero per-ride commission ever
- driver can work with Yandex/InDrive in parallel
- BIRGE is a peak-hour income layer at MVP, not a full-day Yandex replacement
- driver subscription activates only when the first corridor becomes active
- minimum vehicle year for current product logic: 2000 or newer
- failed verification leads to correction/re-upload by default, not permanent rejection
- passenger late cancellation/no-show consumes a ride/slot and should protect the driver

Driver plans:

```text
peak_starter  9,900 KZT/month
starter       19,000 KZT/month
professional  28,000 KZT/month
premium       38,000 KZT/month
```

Legal/compliance rule:

```text
Frontend may show legal/privacy/payment states as mock UX.
Real launch, real passengers, real drivers, real payments, and real document handling are blocked until legal, insurance, offer, privacy, and data-residency issues are resolved.
```

## 3. Existing Technical Foundation

Keep the current stack:

| Area | Decision |
|---|---|
| UI | SwiftUI |
| State | TCA |
| Shared package | BIRGECore |
| Local cache | GRDB where persistence is needed |
| Networking | URLSession async/await behind dependencies |
| Auth storage | Keychain for real token/session-like secrets |
| Location | CoreLocation / MapKit |
| Minimum style | native iOS, no web prototype wrapper |

Current app already has:

- `BIRGEPassenger` target
- `BIRGEDrive` target
- shared `BIRGECore`
- `AppFeature`, `PassengerAppFeature`
- passenger OTP/auth base
- driver registration base
- design system files in `BIRGECore/Sources/BIRGECore/DesignSystem`
- TCA test target and existing reducer tests

Important current mismatch:

```text
Current passenger root flow checks only token existence.
New frontend needs phone-aware route decisions:
new phone -> registration/onboarding
known phone + completed profile -> main app
known phone + incomplete setup -> resume exact setup step
```

## 4. Frontend-Only Architecture Contract

All real product values must come from typed state, fixtures, or dependencies.

Allowed:

- mock auth client
- mock user registry
- mock route/corridor repository
- mock subscription repository
- mock payment status
- mock document upload status
- mock support tickets
- mock driver verification status
- local fixture JSON or Swift fixture factories

Not allowed:

- hardcoded route names/prices/statuses directly in SwiftUI views
- static screens that are not connected by reducer state
- backend calls from view bodies
- business logic hidden in `Button` closures
- raw HTML import or WebView implementation
- real Kaspi/card charge flow in this lane
- real driver document upload/storage in this lane

Preferred shape:

```text
View -> Store -> Feature reducer -> dependency protocol -> mock/live implementation
```

Examples:

```swift
@Dependency(\.authClient) var authClient
@Dependency(\.passengerRouteClient) var passengerRouteClient
@Dependency(\.subscriptionClient) var subscriptionClient
@Dependency(\.driverOnboardingClient) var driverOnboardingClient
```

The live implementations may remain TODO/stubbed while mock implementations drive the frontend.

## 5. Auth And Entry State Machine

Use phone number as the first identity key.

Passenger entry:

```text
launch
-> splash
-> phone
-> otp
-> auth decision
```

Auth decision table:

| Condition | Next state |
|---|---|
| no local session | phone login |
| OTP verified, phone not found in mock registry | passenger registration |
| OTP verified, phone found, profile incomplete | resume passenger setup step |
| OTP verified, phone found, profile complete, no route | first route setup |
| OTP verified, phone found, active subscription/route exists | passenger home |
| session exists but mock profile incomplete | resume setup |
| session exists and mock profile complete | passenger home |

Driver entry:

```text
launch
-> splash
-> phone
-> otp
-> auth decision
```

Driver auth decision table:

| Condition | Next state |
|---|---|
| no local session | driver phone login |
| OTP verified, driver not found | driver onboarding |
| driver profile incomplete | resume exact onboarding step |
| verification submitted/pending | verification review/status |
| verification failed | correction/re-upload flow |
| verification approved and first corridor not active | waiting for first corridor |
| active corridor exists | driver home |

Implementation note:

```text
Do not use only access-token existence as the root app decision.
The root decision must include user role, phone, and profile/onboarding completion state.
```

## 6. Mock Data Layer

Create a named fixture layer, not random per-screen constants.

Recommended module layout:

```text
BIRGECore/Sources/BIRGECore/MockData/
  BIRGEProductFixtures.swift
  PassengerFixtures.swift
  DriverFixtures.swift
  RouteFixtures.swift
  SubscriptionFixtures.swift
```

Alternative if the project prefers app-target ownership:

```text
BIRGEPassenger/Fixtures/
BIRGEDrive/Fixtures/
```

Rule:

```text
If data appears on more than one screen, it belongs in fixtures or a dependency response model.
If data appears only once but has business meaning, still prefer a fixture/model over inline view literals.
```

Mock data categories:

- user profile
- phone session
- onboarding progress
- route draft
- pickup/dropoff nodes
- recurring routes
- monthly commute plan
- passenger subscription plans
- payment methods
- billing/receipt history
- ride-day timeline state
- support/safety tickets
- driver profile
- vehicle profile
- document upload states
- service areas/corridor preferences
- driver subscription plans
- driver schedule/corridor runs
- earnings/payout states

## 7. Native Apple APIs And Libraries

Prefer Apple frameworks first:

| Need | Preferred API |
|---|---|
| UI | SwiftUI |
| Maps/routes | MapKit |
| Location permissions | CoreLocation |
| Notifications mock/permission UI | UserNotifications |
| Photo/document placeholders | PhotosUI where needed |
| Secure token/session storage | Security / Keychain |
| Formatting | Foundation `FormatStyle`, `Date.FormatStyle`, `MeasurementFormatter` |
| Symbols/icons | SF Symbols |
| Accessibility | SwiftUI accessibility labels/identifiers |

Existing GitHub dependencies to keep:

- Point-Free TCA
- Point-Free Dependencies
- Point-Free Concurrency Extras
- GRDB

Possible additions only if justified:

| Library | Use only if |
|---|---|
| PhoneNumberKit | Kazakhstan `+7` phone formatting/validation becomes painful with a small local formatter |
| swift-snapshot-testing | visual regression snapshots become part of CI/QA |
| SFSafeSymbols | SF Symbol names become error-prone and the team wants compile-time safety |

Avoid adding for this lane unless there is a strong reason:

- Lottie
- Kingfisher
- SwiftUIIntrospect
- custom animation frameworks
- custom map/render engines
- large design system packages

## 8. UI Implementation Rules

General:

- one screen = one primary task
- one obvious primary CTA
- use native navigation, sheets, tabs, forms, segmented controls, pickers, and MapKit surfaces
- use SF Symbols, not emoji
- no pictographic emoji in production UI
- no AI-looking explanation blocks
- no long educational copy inside core action screens
- keep passenger visual polish but reduce density where Swift implementation needs clarity
- driver UI must stay calmer and less dense than passenger UI

SwiftUI:

- small focused subviews
- `NavigationStack` with enum routes
- `.sheet(item:)` for item-based sheets
- TCA reducers own business/navigation state
- views render state and send actions only
- every important screen gets at least one `#Preview`
- every interactive primary path gets accessibility identifiers

Design mapping:

- route canvas -> native map/route preview component
- subscription pass/ticket -> native SwiftUI pass component
- ledger -> native list/section with thin dividers
- timeline rail -> native vertical progress/timeline component
- wheel picker -> native picker or custom wheel-like SwiftUI control if needed
- payment carousel -> horizontal native selector
- live activity-like surface -> compact status panel

Do not translate HTML literally.

HTML mockups are visual/product references. SwiftUI implementation may improve structure if it keeps the product meaning.

## 9. Passenger Implementation Phases

### P0 - App Hygiene And Frontend Mode

Goal:

```text
Make the app ready for mock-driven frontend work.
```

Tasks:

- review current app repo diff and avoid committing accidental Xcode/user-state churn
- add `FrontendEnvironment.mock` or equivalent dependency setup
- add fixture layer
- define `AuthDecision` and onboarding progress models
- make passenger root decision phone/profile aware
- keep existing backend clients untouched behind dependencies

Acceptance:

- passenger app builds
- logout returns to phone login
- mock known/new phones route differently
- no backend required to run frontend

### P1 - Passenger Entry And First-Time Setup

Screens:

- `P-00-splash`
- `P-01-phone`
- `P-02-otp`
- `P-02a-profile-basics`
- `P-02c-trust-consent`
- `P-02d-product-intro`
- `P-03-onboarding`
- `P-03a-route-origin`
- `P-03b-route-destination`
- `P-03c-route-schedule`
- `P-03d-route-review`
- `P-03e-monthly-plan`

Acceptance:

- new phone can finish profile and first route draft
- address input comes before suggested nodes
- route draft survives navigation back/forward
- data lives in state/fixtures, not view constants

### P2 - Passenger Subscription And Activation

Screens:

- `subscription-selection`
- `subscription-kaspi`
- `subscription-success`
- `P-17`
- `P-18`
- `P-32` to `P-38`

Acceptance:

- all three passenger MVP tiers exist
- `multi_corridor` recommended
- payment is mock-only
- Apple Pay, saved card, Kaspi, and card option can be represented
- no real Kaspi integration

### P3 - Passenger Home And Route Management

Screens:

- `P-04-home`
- `P-05-search` as My Routes
- `P-24` to `P-31`

Acceptance:

- home primary action is commute/route management, not taxi request
- saved routes have real state models
- pause/low-density/waitlist are state variants, not separate fake screens only

### P4 - Passenger Ride-Day, Edge Cases, Support, Safety

Screens:

- `ride-request`
- `P-09` to `P-16`
- `P-39` to `P-54`

Acceptance:

- ride-day lifecycle uses explicit enum state
- boarding code is state-driven
- support/safety routes are reachable
- fallback taxi remains secondary

### P5 - Passenger Proactive Assistant And Fallback Taxi

Screens:

- `P-55` to `P-66`

Acceptance:

- assistant is contextual, not a main chatbot tab
- AI suggestions are mock insights, not live ML claims
- fallback taxi is clearly outside subscription

## 10. Driver Implementation Phases

### D0 - Driver Entry And Onboarding Foundation

Screens:

- `D-00-splash`
- `D-01-phone`
- `D-02-otp`
- `D-03-profile`
- `D-04-vehicle`
- `D-05-documents`
- `D-06-service-area`
- `D-07-subscription-plan`
- `D-08-verification-review`

Acceptance:

- new driver phone starts onboarding
- known incomplete driver resumes correct step
- vehicle year rule allows 2000+
- driver plans include Peak Starter, Starter, Professional, Premium
- plan copy says payment starts when first corridor activates
- no guaranteed income language
- no full-day replacement promise

### D1 - Driver Home, Schedule, Corridors

Screens:

- `D-09-home`
- `D-10-schedule`
- `D-11-corridor-detail`
- `D-12-monthly-commitment`
- `D-13-availability`
- `D-14-route-preferences`

Acceptance:

- driver home is planned corridor dashboard
- tabs are `Сегодня / График / Доход / Профиль`
- morning/evening peak-hour model is clear
- Yandex/InDrive parallel work is not blocked

### D2 - Driver Operations

Screens:

- `D-15` to `D-21`

Acceptance:

- no offer-shopping or bidding language
- corridor run has lifecycle states
- passenger boarding/no-show logic is explicit
- active corridor states can later connect to location/backend

### D3 - Driver Edge Cases

Screens:

- `D-22` to `D-29`

Acceptance:

- late/no-show/pickup change/replacement/vehicle issue/emergency/cancellation states are procedural and calm
- driver always sees next best action

### D4 - Driver Earnings, Payout, Account

Screens:

- `D-30` to `D-46`

Acceptance:

- earnings do not imply guaranteed income
- 0% commission appears where appropriate
- driver subscription status is separated from passenger payments
- profile/settings/privacy are reachable from tab structure

## 11. Two-Chat Workflow

Use two separate chats after this planning step.

### Chat A - Prompt Architect

Purpose:

```text
Write precise English coding prompts for the implementation chat.
```

This chat should not code directly.

Each prompt must include:

- goal
- source mockup files
- current repo files to read first
- exact files/modules likely to touch
- product logic constraints
- state/model requirements
- no-hardcode rules
- acceptance criteria
- build/test command
- documentation update requirement

### Chat B - Coding Chat

Purpose:

```text
Implement one small batch at a time in the app repo.
```

Rules:

- read the prompt from Chat A
- inspect current code before edits
- make scoped changes only
- keep frontend-only unless explicitly told otherwise
- update tests/previews when risk justifies it
- run build after each batch
- report changed files and verification

## 12. Prompt Batch Template

Every implementation batch should follow this size:

```text
1 feature foundation OR 3-5 tightly connected screens.
```

Too big:

```text
Implement all passenger screens.
Implement all driver screens.
Rewrite the whole app.
```

Good:

```text
Implement passenger phone/OTP auth decision with mock user registry.
Implement passenger first-route setup P-03a to P-03d.
Implement driver service area + plan selection D-06 to D-07.
Implement driver corridor operation states D-15 to D-21.
```

## 13. Verification Gates

Before any batch is considered done:

- app builds for the touched target
- main flow is reachable from launch
- no new backend dependency is required
- no raw product fixtures are hardcoded in SwiftUI views
- reducer tests exist for non-trivial state machines
- previews exist for major new screens
- visual structure matches the mockup intent, not necessarily every HTML pixel
- Dev Brain task/status doc is updated

Preferred build commands:

```bash
xcodebuild build-for-testing -quiet -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'generic/platform=iOS' -skipMacroValidation CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""
```

```bash
xcodebuild build-for-testing -quiet -project BIRGEPassenger.xcodeproj -scheme BIRGEDrive -destination 'generic/platform=iOS' -skipMacroValidation CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""
```

Working directory:

```text
/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger
```

## 14. Open Decisions To Keep Visible

Do not block frontend prototypes on these, but keep them isolated in fixtures/config:

- exact passenger KZT prices for `solo_corridor`, `multi_corridor`, `flex_pack`
- exact Flex Pack expiry/refund/unused ride rules
- exact passenger no-show compensation accounting for driver
- exact KYC rejection reasons and retry limits
- real Kaspi merchant/payment flow
- legal classification of paid shared rides in Kazakhstan
- insurance coverage for paid passengers in private cars
- production data residency architecture

## 15. First Recommended Implementation Order

1. Review current app repo diff and commit/clean Phase 0 subscription-pivot changes.
2. Add mock frontend environment and fixture layer.
3. Replace token-only root routing with phone/profile-aware auth decision.
4. Implement passenger entry + first-route setup.
5. Implement passenger subscription selection/payment mock/success.
6. Implement passenger home + route management core.
7. Implement driver entry + onboarding + verification review.
8. Implement driver home/schedule/corridor operations.
9. Add earnings/support/settings.
10. Run a final frontend QA pass against passenger and driver galleries.

## 16. Non-Negotiables

```text
Native SwiftUI, not HTML/WebView.
TCA state drives flows.
Mock data is centralized.
Views do not own business logic.
No real payments.
No real document upload.
No backend dependency for frontend preview.
No taxi-first primary passenger path.
No driver gig-marketplace language.
No guaranteed income claims.
```

