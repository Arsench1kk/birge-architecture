---
status: planned
created: 2026-05-04
project: BIRGE
---

# Future Plan — Demo Stability + Per-User Onboarding

## Why This Exists
This file is the compact next-agent context for post-defense hardening. It collects the live-flow findings, what has already been fixed, and the next production goal: real per-user onboarding after OTP login.

## Already Fixed In Live Flow
- `RideFeature` ignores backend WebSocket control frames like `{ "type": "connected" }` and `{ "type": "subscribed" }`.
- `OfferFoundFeature` decline/expiry calls backend ride cancel instead of only closing the local screen.
- `RidesService.driverAccept` uses a SQL row lock so two drivers cannot both accept the same ride.
- `BIRGEDrive` uploads/syncs driver GPS every 5 seconds while tracking is active.
- API/WebSocket base URL is configurable through `BIRGE_API_BASE_URL`; simulator default remains `http://localhost:8080/api/v1`.
- Docker Vapor image must be rebuilt after backend route changes. Demo API health check: unauthenticated `GET /api/v1/demo/state` should return `401 Unauthorized`, not `404 Not Found`.

## Future Goal: Per-User Onboarding State
Current risk: login via different phone numbers can feel like one shared onboarding state if iOS relies on global local flags or backend auth does not expose onboarding status clearly.

Desired behavior:
- OTP login/register tells iOS whether this backend user is new or still needs onboarding.
- New passenger account opens passenger commute onboarding.
- New driver account opens driver personal/vehicle/documents/subscription onboarding.
- Existing completed user goes straight to Home/Dashboard.
- Incomplete onboarding resumes at the correct step.

## Demo-Fix Option: 1.5-3 Hours
Use this if the defense needs the first-time flow quickly.

- Add `isNewUser` or `needsOnboarding` to auth/current-user response.
- Route new passenger/driver users into the correct onboarding after OTP.
- Persist completion minimally, preferably backend-backed enough for Demo Hub visibility.
- Add focused reducer tests for fresh user, completed user, and logout/login.

## Production-Fix Option: 4-8 Hours
Use this for a stable app contract instead of a presentation-only patch.

- Backend persists real onboarding status:
  - passenger: `onboarding_completed` plus commute preferences/routes.
  - driver: `driver_profiles` completion/KYC fields, vehicle fields, documents, and subscription tier.
- `/auth/me` and OTP/login responses return a canonical `onboardingStatus`.
- `AppFeature` root navigation is driven by backend state, not global local memory.
- Passenger onboarding saves commute answers to backend.
- Driver onboarding saves profile data to `driver_profiles`.
- Demo Hub shows where onboarding/profile/preferences data is stored.

## Full Defense-Ready Target: 1 Working Day
This is the safer goal if time allows.

- Different phone numbers create/use different backend users.
- Onboarding does not repeat after completion.
- Logout/login does not leak onboarding state between accounts.
- Passenger and driver onboarding are separated by role.
- Demo DB tab clearly shows saved passenger preferences and driver profile rows.

## Open Future Tasks
- Implement passenger commute preferences persistence endpoint/table if existing corridor data is not enough.
- Implement driver profile completion status contract.
- Add onboarding status to Vapor Auth DTOs and iOS `APIClient` DTOs.
- Remove or reset app-wide local onboarding flags on logout.
- Add Demo Hub section for current user onboarding/profile status.
- Add manual QA script: fresh passenger phone, fresh driver phone, repeat login, incomplete onboarding resume.

## Acceptance Criteria
- Fresh passenger phone sees commute onboarding exactly once.
- Fresh driver account sees personal/vehicle/documents/subscription onboarding until complete.
- Different phone numbers map to different backend users and do not share local onboarding state.
- Completed users skip onboarding after relaunch and after logout/login.
- Demo Hub database tab shows the saved profile/preferences state used by the app.

## Review Findings That Led Here
- `RideFeature` treated backend control frames as ride errors. Fixed.
- Passenger offer decline/expiry was local-only. Fixed.
- Driver accept could race between two drivers. Fixed.
- Driver GPS was not uploaded live during tracking. Fixed.
- Debug builds using `localhost` break on physical iPhone. Configurable base URL added; set Mac LAN IP or tunnel for real-device demo.
