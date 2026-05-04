# Session — Live Ride Search / Driver Offers Stability (2026-05-04)

## Goal
Stabilize the full live Passenger → Driver ride search path after manual QA found server disconnects, stale driver offers, and repeated offers after decline.

## Implemented
- Added persisted `driver_ride_decisions` with unique `(ride_id, driver_id)` for accepted/declined decisions.
- Added `POST /api/v1/rides/:rideID/driver/decline`.
- Filtered driver offers to fresh unassigned `requested` rides only, with 15 minute TTL and per-driver declined exclusion.
- Guarded driver accept against expired/unavailable/previously declined rides and persisted accepted decisions.
- Made WebSocket broadcasts send on each socket event loop and registered text/close handlers from the socket event loop.
- Added `APIClient.declineDriverRide` and wired BIRGEDrive decline to backend instead of local-only dismissal.
- Updated passenger searching to treat canonical `ride.status_changed` with `driver_accepted` as a matched driver event.

## Verification
- `swift build` passed for Vapor.
- `swift test` passed for Vapor: 11 tests.
- `xcodebuild build` passed for `BIRGEPassenger` on iPhone 17 Pro simulator.
- `xcodebuild build` passed for `BIRGEDrive` on iPhone 17 Pro simulator.
- `xcodebuild test` passed for `BIRGEPassenger`; new `SearchingFeatureTests.testDriverAcceptedStatusEventDelegatesDriverInfo` passed.
- `docker compose build vapor` passed with Swift 6.0 image.
- `docker compose up -d postgres redis vapor` started live stack.
- Live smoke: OTP request/verify, ride create, driver offers, decline hides offer for same driver, second driver still sees offer, second driver accepts, accepted ride disappears from offers.
- Raw WebSocket HTTP upgrade returned `101 Switching Protocols`; vapor container remained running.

## Notes
- TTL for driver offers is 15 minutes.
- Decline is persisted per driver+ride and does not hide the ride from other drivers.
- Old `requested` rides are currently filtered, not deleted; cleanup can be added later.
