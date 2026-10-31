---
last_updated: 2026-05-12
sprint: swift_frontend_implementation
---

# Current Focus — BIRGE

## Аудит и стабилизация — 2026-10-31

Эта запись актуальнее майских статусов ниже. Ветка приложения `thesis/matching-lab`; в рабочем дереве были пользовательские изменения onboarding, их не следует перезаписывать.

- Matching-lab: baseline 0.2.0 ищет свободную минуту в окне, если предпочтительное время занято; предыдущие результаты baseline 0.1.0 не смешивать с новой версией. Добавлена отметка `matching_lab_dirty` и проверен smoke-run. Числа сценариев остаются синтетическими placeholders.
- Backend: добавлены ограничения production для JWT, OTP и незавершённой интеграции Kaspi, отзыв новых сессий, проверка владельца маршрута и транзакция при создании плана, блокировки переходов и уведомление об отмене. Проверку интеграции и сборки необходимо сверять с результатами текущего запуска; наличие кода не означает готовность production.
- Обновление logout: пассажирский и водительский iOS-клиенты запрашивают отзыв сессии до локального удаления токенов; backend закрывает WebSocket-соединения отозванной сессии на текущем экземпляре. Это подтверждено HTTP/WebSocket smoke-тестом с изолированными PostgreSQL и Redis. Для нескольких экземпляров нужен межсерверный механизм отзыва. Rate limit остаётся локальным для процесса. Нужны интеграционные тесты с PostgreSQL/Redis, проверка конкурентных переходов и согласование полной политики отмен.
- iOS: ошибка TCA при Xcode 27.2 устранена фиксацией TCA 1.26.1 и Sharing 2.8.2. Оба target собраны для arm64 iOS Simulator с `-skipMacroValidation`; для обычного запуска Xcode может попросить подтвердить обновлённый макрос. Runtime и тесты iOS не проверены.
- Backend: `swift test --skip-update` — 12 тестов прошли; изолированные PostgreSQL/Redis и HTTP smoke: чужой маршрут отклонён без частичного плана, свой план создан, отмена неназначенной поездки и отзыв refresh после logout работают. При недоступном Redis регистрация может оставить пользователя без токенов — требуется исправление. Не объявлять платёжную интеграцию, SMS delivery или реальную эксплуатацию готовыми.


## ▶ Active Lane: Swift Frontend Implementation

### Status as of 2026-05-12

Phase 0 passenger frontend pivot is **committed and verified**.

Recent commits on `feature/passenger-liquid-glass-ui`:
- `586f56a9` feat(core): add planned ride and support mock models, expand fixtures and design tokens
- `8beb7230` feat(passenger): implement subscription-corridor frontend pivot
- `d74c5267` feat(passenger): add planned ride lifecycle and support features
- `e5400a06` test(passenger): add and update tests for subscription-corridor pivot

Verification:
- ✅ BIRGEPassenger build passes
- ✅ BIRGEDrive build passes
- ✅ All BIRGEPassengerTests pass (zero failures)
- ✅ `git diff --check` clean
- ✅ Only `UserInterfaceState.xcuserstate` excluded from commits

Final mockups are accessible at:
- Passenger: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEMockups/passenger-polished/index.html`
- Driver: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEMockups/driver/index.html`

### Next Step

Execute Swift Frontend Master Plan starting with Task 1 (Batch 1).

Source plan: [[Technical_Department/iOS/Swift_Frontend_Master_Plan]]
Next tasks: [[Technical_Department/iOS/Swift_Frontend_Next_Tasks]]
Task template: [[Technical_Department/iOS/Swift_Frontend_Agent_Task_Template]]

Current batch:
1. Add shared `BIRGEStatusBadges.swift` + `BIRGETabBar.swift` (design system)
2. Extract `ProfileBasicsStepView` (onboarding decomposition)
3. Extract `TrustConsentStepView`
4. Extract `ProductIntroStepView`
5. Polish `SplashView` to match mockup

### What Is NOT Active Now

- Backend / Vapor work — paused until frontend contracts stabilize
- Driver app rebuild — starts after passenger P-A through P-F
- Real payments / Kaspi merchant — blocked on legal entity
- Real document upload — mock status only
- Broad UI rewrites — incremental polish only

---

## ▶ Previous Lane: Business Foundation (completed 2026-05-10)

### Цель: Business Foundation First

Новый железный фундамент зафиксирован в [[Context/Universal_AI_Handoff_BIRGE]] v6.0:

> BIRGE is not a taxi-first app. BIRGE is a transportation-as-a-subscription platform for recurring Almaty commuters.

Главный wedge:

```text
Tier 3: subscription commute
```

Текущая задача — **сначала полностью определить бизнес/продуктовую логику**, потом возвращаться к UI mockups и только после этого менять iOS/backend.

### What Changed (2026-05-07)

- BIRGE repositioned from “taxi/ride-hailing app” to **subscription commute platform**.
- On-demand ride is now fallback/acquisition only, not the core product.
- Passenger’s primary action is monthly route declaration, not “call taxi”.
- Driver monetization is fixed monthly subscription only: **zero per-ride commission ever**.
- MVP corridor is hub-to-hub: one pickup node, one dropoff node, no intermediate stops.
- First target corridor: Alatau City / Алатауский район -> Esentai / Al-Farabi, 07:15-08:30.
- Current UI does not fully match the new product foundation; UI mockups should pause until business flows are defined.
- Old self-filled survey data must not be used as real validation.

### Current Business Docs

- [[Business_Department/00_Business_Index]]
- [[Business_Department/Product_Logic]]
- [[Business_Department/Driver_Economics]]
- [[Business_Department/Driver_Utilization_Risk]]
- [[Business_Department/Legal_Compliance_Risk_Assessment]]
- [[Business_Department/Business_Open_Questions]]
- [[Tasks/Business_Sprint_0_Foundation]]
- [[Tasks/Technical_Subscription_Pivot]]
- [[Decisions/ADR-005_Subscription_Pivot]]

### Technical Audit Update (2026-05-08)

Claude Code produced [[Context/implementation_plan_]] after reading Dev Brain and app repo.

Key findings:
- 28 code review findings.
- Critical mismatches: old passenger subscription tiers, taxi-first Home CTA, one-off ride model, missing recurring route/monthly plan models.
- Reusable infrastructure is strong: auth, WebSocket, location, TCA, BIRGECore design system, ride FSM, Kaspi skeleton, Docker/CI.
- App repo now has uncommitted Claude Code changes: `TODO(subscription-pivot)` markers in critical files and driver tier pricing fixed to 19k/28k/38k.
- Business update after that: driver pricing now adds Peak Starter at 9,900 KZT, so app repo pricing must be reviewed again before committing implementation.

Current technical task:

```text
Review and intentionally commit or clean Claude Code Phase 0 app repo changes before starting Phase 1.
```

### Open Design Mockup Checkpoint (2026-05-09)

Passenger Open Design full-product pass is checkpointed:
- `improved/passenger/index.html` now contains 78 screens.
- Coverage includes registration, route setup, subscriptions, route management, ride-day lifecycle, edge cases, support/safety, proactive assistant states, and fallback taxi as a secondary flow.
- Passenger mockup source of truth: [[docs/Passenger_Product_Mockup_Roadmap]].
- UX designer feedback flagged the passenger mockups as visibly AI-generated: unclear click path, too much text, too many blocks.
- Audit saved in [[docs/Passenger_UX_Cleanup_Audit]].
- Cleanup prompt saved in [[Context/Open_Design_Passenger_UX_Cleanup_Prompt]].

Decision update: do **not** rewrite passenger mockups now. They took significant effort and remain useful as a beautiful product coverage prototype. Treat passenger screens as visual-quality reference, not final UX. The main lesson is to avoid repeating the same density problem in the driver app.

### Driver Business Logic Checkpoint (2026-05-09)

Business Session 7 confirmed the driver-side foundation:
- first driver persona: Серик, 32-42, full-time Almaty taxi driver, owns CNG car, values predictability over risky upside
- minimum vehicle year for MVP mockups/business logic: 2000 or newer
- driver subscription starts only when the first corridor becomes active
- failed verification leads to correction/re-upload, not permanent rejection by default
- passenger late cancellation/no-show consumes a ride/slot and should protect the driver
- parallel work with Yandex/InDrive is allowed during launch
- BIRGE driver MVP positioning: peak-hour income layer, not full-day Yandex replacement
- Peak Starter confirmed as fourth driver plan at 9,900 KZT/month
- legal consultation is still a blocker before real operations


### Driver Open Design Mockup Checkpoint (2026-05-09)

Driver Open Design full-product pass is checkpointed:
- `improved/driver/index.html` now contains 47 screens, `D-00` through `D-46`.
- Coverage includes driver onboarding, service zones, driver platform plans, verification, today/home, schedule, corridor operations, edge cases, earnings/payouts, support, safety, profile, settings, and privacy/data controls.
- Driver mockup source of truth: [[docs/Driver_Product_Mockup_Roadmap]].
- Open Design folder: `/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver`.
- Final polish pass added native row icons, circular driver avatars, profile destination `D-44-profile.html`, and bottom tab links for `Сегодня / График / Доход / Профиль`.
- Driver plan screen now includes Peak Starter `9,900 KZT/month` plus Starter `19,000`, Professional `28,000`, and Premium `38,000`.
- `D-06-service-area.html` now supports `Рядом с домом`, `точка выезда`, and `Не важно` so drivers can receive corridors near their morning starting point.
- Passenger files were not intentionally changed during driver work.

Decision update: driver mockups are ready as a product-coverage prototype. Do not keep prompting broad driver rewrites unless a specific visual/UX issue is found. Next passenger cleanup must be done in a separate working copy, not over `improved/passenger/`.

### Legal / Compliance Checkpoint (2026-05-09)

AI legal/compliance chat output was saved as [[Business_Department/Legal_Compliance_Risk_Assessment]].

Treat it as a risk map, not legal advice.

Top launch blockers before real passengers/drivers/payments:
- register ИП or ТОО
- consult licensed Kazakhstan transport lawyer on aggregator vs carrier/operator classification
- solve insurance risk; standard ОГПО may not cover paid passenger transportation
- draft public offer, refund/no-show policy, and passenger acceptance flow
- prepare privacy policy, personal data consents, location consent, and data residency/localization review

### Passenger Polished Open Design Checkpoint (2026-05-10)

Passenger Open Design polished full-product pass is now checkpointed:
- final working copy: `/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger-polished`
- gallery: `improved/passenger-polished/index.html`
- screen count preserved: 78 passenger screens
- original `improved/passenger/` and legacy files were not the target of this final polish lane
- checkpoint source of truth: [[docs/Passenger_Product_Mockup_Roadmap]]

What changed in the final polish lane:
- screens were improved in batches, not rewritten as a reduced MVP
- the visual language moved away from generic card stacks toward concrete compositions: route canvases, subscription passes, ticket/receipt surfaces, account ledgers, timeline rails, native wheel picker, payment carousels, icon action rails, map/node canvases, and compact status strips
- repeated UI problems were fixed: nested cards, unclear action hierarchy, meaningless badges, overexplaining copy, AI-looking reasoning sections, card/row monotony, edge padding regressions, off-center icons, tab/action alignment, and text collisions
- payment language now supports Apple Pay, saved bank card, Kaspi, and card option where relevant; Kaspi is not forced during registration
- first-route setup now follows the correct product logic: user enters/searches home/work address first, then BIRGE suggests nearby pickup/dropoff nodes
- schedule selection now uses an iOS-style wheel time picker plus flexibility selector instead of an awkward custom slider
- fallback taxi remains a secondary flow outside subscription

Final design lesson for Open Design prompting:
```text
Do not ask Open Design to “make it beautiful” or only “remove cards”.
Name the replacement composition explicitly: route canvas, ticket/pass, ledger, rail, picker, carousel, live activity surface, or native form.
```

Decision update: passenger polished mockups are ready as the current visual/product prototype for later SwiftUI mapping. Further Open Design work should be targeted QA only unless a specific product gap is found.

### Swift Frontend Planning Checkpoint (2026-05-10)

Frontend-only Swift transfer plan is now documented in [[Tasks/Swift_Frontend_Mockup_Implementation_Plan]].

Prompt handoff for the separate prompt-writing chat is saved in [[Context/Swift_Frontend_Prompt_Architect_BIRGE]].

New implementation direction:
- build native SwiftUI/TCA frontend from the final Open Design mockups
- keep backend, real payments, and real document upload out of this lane
- centralize mock data in fixtures/dependencies instead of hardcoding inside SwiftUI views
- replace token-only root routing with phone/profile-aware auth decisions
- support `new phone -> registration`, `known complete phone -> main app`, and `known incomplete phone -> resume setup`
- preserve both targets: `BIRGEPassenger` and `BIRGEDrive`
- use Apple-native APIs first: SwiftUI, MapKit, CoreLocation, PhotosUI where needed, Keychain, UserNotifications, SF Symbols, Foundation formatters
- add GitHub libraries only with explicit justification; current stack already has TCA, Dependencies, Concurrency Extras, and GRDB

### Immediate Priority Order

1. Treat passenger polished mockups as checkpointed; only targeted QA fixes unless a specific product gap is found.
2. Treat driver mockups as checkpointed; only targeted QA fixes unless a specific product gap is found.
3. Use [[Tasks/Swift_Frontend_Mockup_Implementation_Plan]] as the foundation for the frontend-only SwiftUI lane.
4. Review/commit or clean Phase 0 technical pivot changes from Claude Code before implementing the new frontend lane.
5. Add frontend mock environment/fixtures, then fix phone/profile-aware auth routing.
6. Implement passenger and driver SwiftUI flows in small prompt batches, not broad rewrites.
7. Define concrete KZT pricing for all three MVP passenger subscription tiers before real payment/backend wiring.
8. Prepare real legal consultation packet using [[Business_Department/Legal_Compliance_Risk_Assessment]].
9. Resume backend design for `RecurringRoute`, `MonthlyCommutePlan`, passenger subscription plans, and commute plan route linkage after frontend contracts stabilize.

### Tech Lane Status

UI + Navigation Rescue is not deleted. It is **paused as the primary lane** until the business/product flow is clear.

Allowed tech work during this pause:
- bug fixes
- build/test stabilization
- documentation cleanup
- small safety improvements

Avoid:
- new product screens
- major UI redesigns
- new backend product features
- anything that bakes in taxi-first assumptions

---

## Previous Tech Lane: UI + Navigation Rescue

UI + Navigation Rescue was the previous active lane. Keep its notes for continuity, but do not expand UI/product scope until [[Tasks/Business_Sprint_0_Foundation]] produces the passenger and driver business flows. First reference for any resumed UI cleanup: [[Tasks/UI_Navigation_Rescue_Audit]].

Previous lane: Passenger Liquid Glass UI по итоговым mockups.

### Session summary (2026-05-05, Dev Brain Source of Truth Cleanup)
- Promoted the Dev Brain source-of-truth rules into [[CLAUDE]] and [[00_Index]] with canonical paths, target names, and precedence order.
- Updated current context files so iOS and Backend agent briefings match the live app repo: `BIRGEPassenger`, `BIRGEDrive`, `BIRGECore`, Swift Vapor `birge-vapor`, query-token ride WebSocket, and restored Docker/local backend status.
- Cleaned stale architecture references that still used Go/internal snippets for JWT, rate limiting, ride status transitions, outbox polling, data pipeline, matching, Kaspi HMAC, and open-question impacts.
- Updated [[docs/CLAUDE_for_mockups]] from old prompt-era status into current UI/mockup guidance for the active UI + Navigation Rescue lane.
- Added [[Context/Universal_AI_Handoff_BIRGE]] as a self-contained handoff prompt for Claude, Gemini, ChatGPT, NotebookLM, or any AI chat without access to local files.
- Verification: Dev Brain `git diff --check` passed and stale-reference search for known outdated terms returned clean for current source-of-truth files.

### Session summary (2026-05-05, UI + Navigation Rescue)
- Started the rescue lane with Passenger root/navigation cleanup before visual rewrites.
- Renamed Passenger root state `onboarding` -> `needsOnboarding` to match the planned root vocabulary.
- Replaced anonymous `path.append/removeLast/removeAll` ride-flow mutations in `PassengerAppFeature` with named state helpers: `openRideRequest`, `startRideSearch`, `showOfferFound`, `showRideAfterOfferConfirmed`, `resetRideFlow`, and related browse destinations.
- Added `RideFlowResetReason` so decline/expiry/cancel/completion resets are explicit and searchable.
- Added `AppFeatureNavigationTests` for splash -> needs onboarding, onboarding -> OTP, and logout -> OTP session reset.
- Expanded `PassengerAppFeatureTests` for ride request back, search cancel, ride completion dismissal, and profile logout delegation.
- Expanded the shared `BIRGECore` UI component kit in small files: icon/toolbar/map buttons, text field/list rows, section headers, loading/empty/error states, and initial ride/route/offer building blocks.
- Smoke-used the new components in `MyCorridorsView`, `ProfileView`, and `ProjectDemoView` without starting a broad screen rewrite.
- Pushed app commit `5a4a24b5 feat(core): expand shared UI component kit`.
- Started Driver UI rescue by extracting dashboard/offer surfaces from `DriverAppView` into focused SwiftUI files: `DriverDashboardSurfaces`, `DriverOnlineControlSheet`, `DriverOfferViews`, and `DriverRidePrimitives`.
- `DriverAppView.swift` dropped from 1155 lines to 634 lines without changing the live driver offer/ride actions or reducer flow.
- Pushed app commit `87a442bc refactor(driver): extract dashboard components`.
- Continued Driver UI rescue by extracting active ride lifecycle surfaces from `DriverAppView` into `DriverActiveRideViews`, `DriverActiveRideDetails`, `DriverCompletedRideSheet`, and `DriverRideFormatting`.
- `DriverAppView.swift` is now 160 lines; active ride actions still delegate to the existing `DriverAppFeature` actions, so live accept/arrived/start/complete flow is unchanged.
- Pushed app commit `9a0d8905 refactor(driver): extract active ride surfaces`.
- Added a visible BIRGEDrive account menu with destructive logout confirmation.
- Replaced stored driver root booleans with explicit `DriverAppFeature.State.Root` cases: `auth`, `registration`, `dashboard`, and `activeRide`.
- Driver logout now resets route/state, clears local access/refresh/user tokens through `TokenRefreshClient.clearTokens()`, clears dashboard/ride state, and stops/syncs driver location tracking without adding backend flow.
- Pushed app commit `20ed72a1 refactor(driver): add account logout routing`.
- Verification: `git diff --check` passed; `xcodebuild build-for-testing -quiet -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'generic/platform=iOS' -skipMacroValidation CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""` passed. Direct simulator test run is blocked locally because Xcode currently does not list a concrete iOS Simulator device.
- Additional verification: `xcodebuild build-for-testing -quiet -project BIRGEPassenger.xcodeproj -scheme BIRGEDrive -destination 'generic/platform=iOS' -skipMacroValidation CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""` passed after the shared core bridge update, Driver dashboard extraction, Driver active ride extraction, and Driver logout/root routing cleanup.

### Active branch
- App repo: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp`
- Branch: `feature/passenger-liquid-glass-ui`
- Remote: pushed to GitHub

### Session summary (2026-05-03, Passenger Liquid Glass UI)
- Read final mockups/context from `docs/` and codex prompts.
- Created grouped commits for existing local UI work and pushed them.
- Added native iOS 26 Liquid Glass support in BIRGECore with material fallback for older iOS.
- Rebuilt/polished passenger Splash, Onboarding, Home, BoardingCode, RideComplete, RideMap, and RideRequest surfaces.
- Replaced visible passenger ride emoji/stickers with SF Symbols where applicable.
- Added real TCA navigation for corridor list and corridor detail screens from Home.
- Added `CorridorListFeature/View` and `CorridorDetailFeature/View`; they now load live corridor data through `APIClient`.
- Added P-08 OfferFound confirmation flow: `Searching → OfferFound → RideMap` on confirm, decline/expiry returns to searching.
- Implemented Vapor `/api/v1/corridors` with default seeded corridors and passenger booking endpoint.
- Connected Passenger Home, corridor list, and corridor detail screens to live `APIClient.fetchCorridors` / `bookCorridor` calls with loading/error states.
- Pushed app commit `2fd2a124 feat(corridors): connect passenger corridors to API`.
- Improved RideMap WebSocket recovery UI and production lifecycle event aliases.
- Pushed app commit `51a890d7 feat(ride): improve websocket recovery events`.
- Unblocked the `SwiftNavigation` / `CasePathsCore` linker failure by pinning a compatible package graph and adding explicit runtime package products.
- Pushed app commit `cf9265e3 fix(tests): unblock passenger package linking`.
- Fixed `OTPFlowE2ETests` for the current splash-first app startup and restored full passenger XCTest suite.
- Pushed app commit `e8a38820 test(passenger): align otp flow tests with splash startup`.
- Added P-23 `AIExplanationFeature/View` with Liquid Glass cards, SF Symbols, Home AI pill navigation, CTA into corridor list, and reducer coverage.
- Pushed app commit `da9001cf feat(passenger): add ai explanation screen`.
- Rebuilt onboarding into an 8-page flow: 3 intro slides plus P-03a–P-03e commute setup for origin, destination, times, weekdays, and AI route summary.
- Pushed app commit `fe208327 feat(passenger): add commute setup onboarding`.
- Added P-17/P-18 subscriptions flow with TCA state for plans, detail, activation, and Home subscription navigation.
- Pushed app commit `9f06a2a4 feat(passenger): add subscriptions flow`.
- Implemented Vapor `/api/v1/subscriptions` overview/activation API with persisted passenger subscription state.
- Connected `SubscriptionsFeature/View` to live subscription plans and activation through `APIClient`, with loading/error/activating states.
- Pushed app commit `50c3915f feat(subscriptions): connect passenger plans to api`.
- Added Vapor Payments module for Kaspi checkout deep links and append-only/idempotent webhook events.
- Connected subscription activation UI to Kaspi handoff before demo confirmation.
- Pushed app commit `01955808 feat(payments): add kaspi subscription handoff`.
- Added persisted corridor booking records with passenger/corridor uniqueness and idempotent repeat booking behavior.
- Pushed app commit `ab5237fe feat(corridors): persist passenger bookings`.
- Updated CorridorDetail UX to show persisted booking status, booking id, refreshed seats/participants, and disabled re-join CTA after success.
- Pushed app commit `17d44560 feat(corridors): show passenger booking status`.
- Added `/api/v1/corridors/bookings` and Passenger `MyCorridorsFeature/View` reachable from the Home `Поездки` tab.
- Pushed app commit `061d6aaf feat(corridors): add passenger bookings list`.
- Hardened Kaspi webhook handling with `KASPI_WEBHOOK_SECRET`, HMAC-SHA256 signature validation, header fallback support, and backend unit tests.
- Pushed app commit `98948bf4 feat(payments): validate kaspi webhook signatures`.
- Started driver-side Sprint 1 gaps with D-03 registration onboarding: personal data, vehicle, documents, and subscription tier steps before dashboard.
- Pushed app commit `d83fec67 feat(driver): add registration onboarding flow`.
- Polished driver D-05 online waiting and `driver/offer` surfaces in `DriverAppView` with a map-style background, Liquid Glass top/bottom overlays, SF Symbols, offer alert, match pill, route card, metrics grid, and passenger confirmation row.
- Pushed app commit `136f91a0 feat(driver): polish online offer surfaces`.
- Polished driver D-07/D-08/D-09/D-10 lifecycle in `DriverAppView` and `DriverAppFeature`: active ride route sheet, pickup/boarding/in-progress state panels, boarding codes, passenger manifest, progress bar, and completed ride summary with next-ride action.
- Pushed app commit `079ac13a feat(driver): add active ride lifecycle polish`.
- Added driver background GPS sync path: BIRGEDrive starts `LocationClient` tracking on accepted rides, stops/syncs on completion/offline, enables driver background location Info.plist keys, and Vapor now exposes authenticated `POST /api/v1/locations/bulk` backed by `driver_location_records`.
- Pushed app commit `e778858b feat(driver): sync background locations`.
- Added live driver location broadcast: `/locations/bulk` now emits the latest batch point as canonical `ride.location_update` on `ride/<ride_id>` WebSocket channels, matching passenger `RideFeature` parsing.
- Pushed app commit `85a66398 feat(locations): broadcast driver updates`.
- Added backend Driver module: authenticated `/api/v1/drivers/me` GET/PUT/POST for registration/profile persistence and `/api/v1/drivers/corridors/today` for driver dashboard corridor candidates.
- Pushed app commit `c5012c77 feat(driver): add driver profile api`.
- Connected BIRGEDrive registration/dashboard to Driver API: registration saves through `APIClient.updateDriverProfile`, app startup loads `/drivers/me`, dashboard loads `/drivers/corridors/today`, and no-token demo fallback keeps local preview usable until driver auth exists.
- Pushed app commit `be190fa7 feat(driver): connect dashboard to profile api`.
- Added driver ride assignment/command path: backend exposes driver offers plus accept/arrived/start/complete endpoints, broadcasts canonical `ride.status_changed`, and BIRGEDrive now polls/accepts/sends lifecycle commands through `APIClient`.
- Pushed app commit `a12d75b6 feat(driver): add ride assignment commands`.
- Added BIRGEDrive driver auth/session flow: email login/register screen, APIClient `auth/login`/driver `auth/register`, token storage through shared refresh client, and removed no-token demo fallback from registration/offers.
- Pushed app commit `29903977 feat(driver): add auth session flow`.
- Persisted ride address labels: passenger ride requests now send origin/destination names, Vapor stores them on rides, and driver offers prefer these labels over coordinate fallback.
- Pushed app commit `5bf3fcc1 feat(rides): persist address labels`.
- Added BIRGEDrive accepted-ride navigation polish: active maneuver cue, route guidance panel, phase labels, ETA/speed/safety chips, and SF Symbol direction markers over the map.
- Pushed app commit `5fa70d6a feat(driver): add navigation guidance polish`.
- Restored full live backend Docker stack: Vapor image now builds on Swift 6.0, Kaspi HMAC uses Linux-compatible `swift-crypto`, Postgres config is compiler-friendly in release, `.dockerignore` keeps container context small, and Compose no longer exposes Postgres/Redis host ports.
- Pushed app commit `ef003825 fix(backend): restore vapor docker live stack`.
- Fixed live auth manual testing blockers: iOS decodes backend `userId`, unauthenticated 401/409 responses preserve Vapor reason messages, and driver registration now returns `409 Phone already registered` instead of a Postgres duplicate-key 500.
- Pushed app commit `f9485d50 fix(auth): align live login responses`.
- Stabilized BIRGEDrive manual QA: location tracking no longer enables background updates unless `UIBackgroundModes` actually contains `location`, preventing the simulator abort; driver registration completion now requires real profile fields so new driver accounts see personal/vehicle/documents/tier onboarding.
- Stabilized live Passenger search / BIRGEDrive offers path: WebSocket handler registration now stays on socket event loop, broadcasts send through each socket loop, driver offer decisions persist accepted/declined state, offers are limited to fresh unassigned requests, decline hides offers per driver, and Passenger search accepts canonical `driver_accepted` events.
- Verification passed: Vapor `swift build`, Vapor `swift test`, `BIRGEPassenger` build/test, `BIRGEDrive` build, Docker `compose build/up`, live OTP/ride/offers/decline/accept smoke, and raw WebSocket `101 Switching Protocols` smoke.
- Pushed app commit `eddea55d fix(driver): stabilize manual onboarding checks`.
- Fixed WebSocket connection bug: `LiveWebSocketActor` was yielding `.connected` immediately after `URLSessionWebSocketTask.resume()`, before the HTTP upgrade completed. If the handshake failed (bad JWT, 403, ride not found), the receive loop would error instantly, yielding `.disconnected` and showing the red "Нет соединения" banner to the passenger before the connection was ever truly established. Fixes: (1) `.connected` is now deferred until the first successful message is received; (2) `.disconnected` events are only yielded to UI if the connection was previously confirmed; (3) max retries increased 3→5, ping interval 5s→10s, backoff capped at 30s; (4) `SearchingFeature` now explicitly handles backend `{"type":"connected"}` and `{"type":"subscribed"}` control frames instead of trying to parse them as ride events; (5) comprehensive `os.log` debug logging added to both `LiveWebSocketActor` and `SearchingFeature` for live debugging.
- Fixed passenger WebSocket expired-token loop found in Docker logs (`JWTKit error: exp claim verification failed`). REST refresh was updating the in-memory `AccessTokenStore`, but WebSocket still read the old `birge_access_token` keychain value. `TokenRefreshClient`/`TokenRefreshTransport` now keep the legacy keychain access token in sync, and `SearchingFeature`/`RideFeature` proactively call `APIClient.refreshAccessToken()` before opening `/ws/ride/:rideId?token=...`, falling back to keychain only if refresh is unavailable.
- Verification: BIRGEPassenger build passes, BIRGEDrive build passes, all `BIRGEPassengerTests` pass (SearchingFeatureTests, RideFeatureTests, OTPFeatureTests, WebSocketClientTests, etc.).

### Verification
- ✅ `git diff --check` passed in app repo.
- ✅ `xcodebuild build -skipMacroValidation -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds.
- ✅ Focused `RideFeatureTests`, `OTPFeatureTests`, and `OTPFlowE2ETests` pass with `-skipMacroValidation`.
- ✅ Full `BIRGEPassengerTests` pass with `-skipMacroValidation` on installed `iPhone 17 Pro` simulator; live OTP E2E remains skipped unless `RUN_LIVE_OTP_E2E=1`.
- ✅ `PassengerAppFeatureTests` cover `Home → AI Explanation → Corridor List`.
- ✅ `OnboardingFeatureTests` cover commute setup paging, inputs, day selection, add-another-route, and finish delegate.
- ✅ `SubscriptionsFeatureTests` cover API loading, plan selection, activation, and detail dismissal.
- ✅ Vapor `swift build` succeeds after subscriptions API integration.
- ✅ Vapor `swift build` succeeds after Kaspi payments module integration.
- ✅ Vapor `swift build` succeeds after persisted corridor bookings integration.
- ✅ `CorridorDetailFeatureTests` cover booking status and refreshed corridor state after join.
- ✅ `MyCorridorsFeatureTests` cover booking list loading and selection navigation.
- ✅ Vapor `swift test` covers Kaspi webhook signature accept/reject behavior.
- ✅ `BIRGEDrive` builds after driver registration onboarding integration.
- ✅ `BIRGEDrive` builds after online waiting / driver offer Liquid Glass polish.
- ✅ `BIRGEDrive` builds after active ride lifecycle polish.
- ✅ `BIRGEDrive` and `BIRGEPassenger` build after driver background GPS sync work.
- ✅ Vapor `swift build` and `swift test` pass after `/locations/bulk` integration.
- ✅ Vapor `swift build`, `swift test`, and `BIRGEPassenger` build pass after location WebSocket broadcast.
- ✅ Vapor `swift build` and `swift test` pass after driver profile API integration.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after Driver API iOS hookup.
- ✅ Vapor `swift build`, Vapor `swift test`, `BIRGEDrive` build, and `BIRGEPassenger` build pass after driver ride assignment/commands.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after driver auth/session flow.
- ✅ Vapor `swift build`, Vapor `swift test`, `BIRGEPassenger` build, and `BIRGEDrive` build pass after ride address labels.
- ✅ `git diff --check` and `BIRGEDrive` build pass after driver navigation guidance polish.
- ✅ Vapor `swift build`, `swift build -c release`, `docker compose build vapor`, `docker compose up -d postgres redis vapor`, and local `POST /api/v1/auth/otp/request` return `200 OK` for the full live backend stack.
- ✅ Passenger OTP request/verify returns `200 OK` with `userId`; duplicate driver phone returns `409 Phone already registered`; `BIRGEPassenger` and `BIRGEDrive` builds pass after live auth response fixes.
- ✅ `BIRGEDrive` and `BIRGEPassenger` builds pass after driver location crash guard and onboarding completion gate fix.
- ✅ Vapor `swift build`/`swift test`, Passenger build/test, Drive build, Docker build/up, live ride offer decline/accept smoke, and raw WebSocket `101 Switching Protocols` pass after live ride search stability fix.
- ✅ `BIRGEPassenger` build passes, `BIRGEDrive` build passes, and all `BIRGEPassengerTests` pass after WebSocket connection bug fix (deferred `.connected`, control message handling, debug logging).
- ✅ `BIRGEPassenger` build, `BIRGEDrive` build, and full `BIRGEPassengerTests` pass after WebSocket access-token refresh synchronization.

### Next best steps
1. UI rescue next step: choose between Driver registration form rescue and Passenger ride flow rescue; Driver root/logout is now safe enough for demo QA. See [[Tasks/UI_Navigation_Rescue_Audit]].
2. Future planned: per-user onboarding state after OTP; see [[Tasks/Future_Demo_Stability_and_Onboarding]].
3. Reminder: live-flow findings from the defense review are already fixed; only revisit them if a regression appears.
4. Manually run Passenger and Drive against the live backend and verify the full UI scenario: passenger creates ride, driver declines, another driver accepts, passenger moves to offer/ride screen.
5. Replace current driver guidance placeholders with persisted route geometry / live MapKit directions.
6. Add persisted named pickup/destination inputs beyond current fixed coordinates.
7. Later: replace demo Kaspi deep link with real merchant API contract when credentials/spec are available.

### Agent reminder
Before continuing iOS UI work, always read [[docs/CLAUDE_for_mockups]] and the relevant HTML mockup. The mockup gives the product idea; SwiftUI implementation should improve it with native Liquid Glass and SF Symbols.
For onboarding/auth work, first read [[Tasks/Future_Demo_Stability_and_Onboarding]] so per-user onboarding state stays backend-driven and demo-visible.
For UI/navigation work, first read [[Tasks/UI_Navigation_Rescue_Audit]] and keep fixes native SwiftUI-first.


## ▶ Цель: Sprint 1 stabilization for demo + auth loop

### Стратегия
UI пассажира и водителя уже собран.
По умолчанию тесты опираются на mock/test зависимости, а live OTP E2E включается только явно.

### Порядок экранов
1. OTP Auth экран (телефон → код → войти)
2. Home экран пассажира (карта + кнопка "Вызвать")
3. Экран поиска водителя (анимация поиска)
4. Экран активной поездки (водитель едет, ETA)
5. Экран завершения (рейтинг)
6. Driver Dashboard (`BIRGEDrive` target)

### Last session (2026-04-27)
- Task: Full MVP complete — Passenger + Driver apps
- Screens done: OTP, Home, RideRequest, Searching, ActiveRide, 
  RideComplete, Profile, DriverDashboard, Earnings
- Build: ✅ Clean (both targets)
- Notes: All 5+ MVP features complete. Ready for demo.

### Session summary (2026-05-02, backend)
- Task: Backend Vapor build recovery for Swift 6.3.1 / Vapor 4.x
- Result: `birge-vapor` now builds cleanly with `swift build`
- Verified: WebSocket route handler, Redis async/Future compatibility, JWT middleware path, and async entrypoint lifecycle
- Runtime: `swift run` reaches startup, then stops on local PostgreSQL authentication for user `birge`

### Session summary (2026-05-02, iOS tests)
- Task: restore `BIRGEPassengerTests` linking and verify OTP test flow
- Result: `xcodebuild test -scheme BIRGEPassenger` now succeeds
- Verified:
  - `BIRGEPassengerTests` links Point-Free products correctly (`ComposableArchitecture`, `Dependencies`, `CasePathsCore`, `ConcurrencyExtras`)
  - `BIRGEPassengerTests/OTPFeatureTests` pass
  - `OTPFlowE2ETests.testOTPFlowSuccess` is now opt-in via `RUN_LIVE_OTP_E2E=1`
- Live E2E prerequisites:
  - local Vapor on `http://localhost:8080`
  - PostgreSQL + Redis
  - Vapor writes `/tmp/birge-otp.log`

### Session summary (2026-05-02, iOS WebSocket)
- Task: IOS-014 — WebSocketClient TCA Dependency
- Result: production-grade `WebSocketClient` implemented in BIRGECore
- Delivered:
  - `WebSocketClient.swift` — interface struct, `WebSocketEvent`/`WebSocketMessage`/`WebSocketError` enums, `DependencyKey` with `liveValue`/`testValue`
  - `LiveWebSocketClient.swift` — actor-isolated `URLSessionWebSocketTask` with 5s ping, exponential backoff (1→2→4→8→16→30s), cancellation safety
  - `WebSocketClient+DependencyValues.swift` — TCA dependency registration
  - `WebSocketClientTests.swift` — 5 unit tests using controllable mock
- BIRGECore linked to all targets (BIRGEPassenger, BIRGEDrive, BIRGEPassengerTests)

### Session summary (2026-05-02, iOS Location)
- Task: IOS-015 — LocationClient TCA Dependency
- Result: production-grade `LocationClient` implemented in BIRGECore
- Delivered:
  - `LocationClient.swift` — interface struct, `LocationUpdate`, `LocationTrackingID`, `DependencyKey` with `liveValue`/`testValue`
  - `LiveLocationClient.swift` — actor-isolated `CLLocationManager` wrapper with `LocationDelegate` bridge, offline-first GRDB writes
  - `LocationSyncService.swift` — stateless batch sync utility (200 records/batch), injectable upload closure
  - `LocationClient+DependencyValues.swift` — TCA dependency registration
  - `LocationClientTests.swift` — 5 unit tests (stream, stop, sync marks synced, batch groups of 200, idempotent)
- Note: `POST /locations/bulk` endpoint not yet defined — upload closure is a no-op placeholder

### Session summary (2026-05-02, iOS RideFeature)
- Task: IOS-016 — RideFeature State Machine
- Result: `RideFeature` implemented for passenger production ride flow
- Delivered:
  - `Coordinate` wrapper in BIRGECore for `CLLocationCoordinate2D` interoperability
  - `RideEvent` WebSocket payload models for `ride.status_changed`, `ride.location_update`, `ride.eta_updated`
  - Stub `APIClient` dependency with `fetchRide` and `cancelRide`
  - `RideFeature` 7-state TCA reducer: `requested → matched → driverAccepted → driverArriving → passengerWait → inProgress → completed`
  - `RideMapView` with live MapKit driver annotation, pickup pin, status pill, and bottom sheet
  - `PassengerAppFeature` navigation updated: `Searching → Ride → RideComplete`
  - `RideFeatureTests` added with 7 reducer tests, including WebSocket JSON decoding, cancel, reconnect, and disconnect recovery
- Verification:
  - Fixed Xcode Cmd+B errors in `RideFeature`:
    - removed `ReducerOf<Self>` circular reference by using `some Reducer<State, Action>`
    - replaced custom cancellation marker types with namespaced `String` IDs to satisfy `Hashable & Sendable`
  - Verified: `xcodebuild build -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds
  - Verified: `xcodebuild build -scheme BIRGEDrive -destination 'platform=iOS Simulator,name=iPhone 17 Pro'` succeeds
  - Attempted: `xcodebuild test -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -only-testing:BIRGEPassengerTests/RideFeatureTests`
  - Focused tests are still blocked before `RideFeatureTests` execute by third-party package linker failure in `swift-navigation`
  - Error: undefined `CasePathsCore.CasePathable` / `CasePathsCore.AnyCasePath` symbols while linking `SwiftNavigation.framework`
  - Note: `iPhone 16 Pro` simulator is not installed locally; used installed `iPhone 17 Pro`

### Next session
- Task: IOS-017 — API Client + Token Refresh
- Done: replaced stub `APIClient.liveValue` with authenticated URLSession implementation
- Done: added `TokenRefreshClient` dependency with in-memory access token, Keychain refresh token, 401 refresh-and-retry, and token clearing on auth expiry
- Done: wired `GET /rides/:id`, `PATCH /rides/:id/cancel`, `POST /locations/bulk`, OTP auth endpoints, `POST /auth/refresh`, `GET /auth/me`, and `POST /rides`
- Done: `xcodebuild build -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -quiet` succeeds after API client work
- Remaining: proactive refresh-before-expiry behavior needs a JWT expiry source/decoder
- Remaining: WebSocket auth header
- First test blocker to resolve: `SwiftNavigation` / `CasePathsCore` linker failure before focused RideFeature tests can run
- Model: GPT 5.4/5.5 High for Xcode package/linker triage, then Claude Opus 4.6 for API architecture review


- Vault настроен (v5.1)
- BIRGE.code-workspace создан
- RULES.md в корне проекта


### Session summary (2026-05-04, in-app defense demo)
- Task: In-App Demo Mode for project defense + live-flow stabilization.
- Delivered: `ProjectDemoFeature/View` in `BIRGEPassenger` with Overview, Database, Live scenario, and AI tabs.
- Backend: dev-only JWT-protected `/api/v1/demo/state`, `/api/v1/demo/seed`, `/api/v1/demo/reset` returning live Postgres table snapshots, Redis key summaries, and deterministic AI corridor scoring.
- Stability: passenger/ride WebSocket control frames are ignored, OfferFound decline/expiry cancels backend ride, driver accept uses row lock, driver GPS sync runs every 5 seconds while tracking, API/WebSocket base URL is configurable through `BIRGE_API_BASE_URL`.
- Verification: Vapor `swift build` and `swift test` pass; `BIRGEPassenger` and `BIRGEDrive` simulator builds pass; full `BIRGEPassengerTests` pass on iPhone 17 Pro with `-skipMacroValidation`.
- Demo note: For physical iPhone, set `BIRGE_API_BASE_URL` to Mac LAN/tunnel URL; simulator default remains `http://localhost:8080/api/v1`.
