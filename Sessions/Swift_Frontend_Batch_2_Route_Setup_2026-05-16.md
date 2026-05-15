---
date: 2026-05-16
tags: [birge, ios, swiftui, frontend, passenger, route-setup]
status: checkpoint
---

# Swift Frontend Batch 2 — Route Setup Extraction Checkpoint

## Scope

Passenger frontend lane continues on `feature/passenger-liquid-glass-ui` after Batch 1/Auth polish was committed and pushed.

This checkpoint covers the intermediate Batch 2 work before commit/push:

- route setup extraction from `OnboardingView.swift`
- shared schedule component infrastructure
- read-only Home visual QA
- parallel-agent workflow lesson: avoid putting the only heavy write task on one agent

## Implemented App Changes

### Route Setup Extraction

Kiro extracted the route setup steps from `OnboardingView.swift` into focused files:

- `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/RouteOriginStepView.swift`
- `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/PickupNodeStepView.swift`
- `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/RouteDestinationStepView.swift`
- `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/DropoffNodeStepView.swift`

`OnboardingView.swift` is reduced from the large all-in-one route implementation toward a host/composition file. The route steps now use the committed design-system primitives where practical:

- `BIRGERouteCanvas`
- `BIRGENodeSelectorList`

`OnboardingFeature.swift` was not changed.

Verification reported by Kiro:

- `git diff --check` passed
- focused `OnboardingFeatureTests` passed: 22/22
- build passed after compile fixes during implementation

### Schedule Components

Antigravity added a design-system-only schedule component file:

- `BIRGEPassenger/BIRGECore/Sources/BIRGECore/DesignSystem/BIRGEScheduleComponents.swift`

Public APIs:

- `BIRGEWheelTimePicker(hour: Binding<Int>, minute: Binding<Int>)`
- `BIRGEWeekdayRail(weekdays: [String], selectedWeekdays: Set<String>, onToggle: (String) -> Void)`
- `BIRGEFlexSegments(options: [Int], selected: Int, onSelect: (Int) -> Void)`

Verification reported by Antigravity:

- `git diff --check` passed
- no feature files changed
- full build deferred to integration

## Read-Only QA

Zed reviewed `HomeView.swift` against `P-04-home.html` and recommended keeping roadmap order:

1. finish route setup / schedule extraction first
2. then subscription polish
3. then Home polish

Main Home QA finding: current Home is structurally close but too dense compared to the mockup. Most Home fixes are view-only and should be handled later as a dedicated `HomeView.swift` pass.

## Current App Commit Plan

Commit code in two app commits:

1. `feat(core): add schedule selection components`
   - `BIRGEPassenger/BIRGECore/Sources/BIRGECore/DesignSystem/BIRGEScheduleComponents.swift`

2. `refactor(passenger): extract route setup step views`
   - `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingView.swift`
   - `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/RouteOriginStepView.swift`
   - `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/PickupNodeStepView.swift`
   - `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/RouteDestinationStepView.swift`
   - `BIRGEPassenger/BIRGEPassenger/Features/Auth/OnboardingSteps/DropoffNodeStepView.swift`

Do not commit:

- `BIRGEPassenger/BIRGEPassenger.xcodeproj/project.xcworkspace/xcuserdata/arsenabduhalyk.xcuserdatad/UserInterfaceState.xcuserstate`

## Verification Before Push

Run from app repo:

```bash
cd /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp

git diff --check

cd /Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger
xcodebuild build-for-testing -quiet \
  -project BIRGEPassenger.xcodeproj \
  -scheme BIRGEPassenger \
  -destination 'generic/platform=iOS' \
  -skipMacroValidation \
  CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""

xcodebuild test -quiet \
  -project BIRGEPassenger.xcodeproj \
  -scheme BIRGEPassenger \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
  -skipMacroValidation \
  -only-testing:BIRGEPassengerTests/OnboardingFeatureTests \
  CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""
```

## Parallel-Agent Lesson

This batch showed that parallel work did not speed delivery as much as expected because the only heavy writing task landed on Kiro while other agents had short side tasks. For future batches:

- split heavy work into similarly sized coding tasks
- do not give one agent the only bottleneck file if all progress depends on it
- prefer one agent per owned file or file family
- use read-only QA in parallel, but do not count it as implementation throughput
- run one integration build after several small tasks rather than every tiny view-only edit

## Next Recommended Step

After commit/push:

- wire `BIRGEWheelTimePicker`, `BIRGEWeekdayRail`, and `BIRGEFlexSegments` into the schedule step
- then extract/polish route review
- only after route setup and schedule are stable, continue to subscription polish and then Home polish
