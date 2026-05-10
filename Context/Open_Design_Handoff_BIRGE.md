---
version: v2.1
last_updated: 2026-05-09
purpose: Universal handoff context for continuing BIRGE Open Design work in another AI chat
---

# BIRGE Open Design Handoff

Use this file when starting a new AI chat dedicated to Open Design, mockups, visual design, design-system iteration, and screen improvement for BIRGE.

This file is intentionally self-contained. The new AI chat may not know the previous conversation history.

---

## Role For The New AI Chat

You are continuing Open Design work for **BIRGE**, a transportation-as-a-subscription platform for recurring Almaty commuters.

Your job is to help improve imported HTML/CSS mockups into higher-quality mobile app screens that can later be translated into SwiftUI.

Primary focus:
- UI/UX mockups for monthly commute subscription flows
- screen-by-screen product clarity
- native iOS feel
- monthly route declaration and route-plan review
- hub-to-hub corridor clarity
- trust, reliability, and safety cues
- design system consistency
- practical Almaty / Kazakhstan market realism

Do not turn app screens into landing pages. These are product screens, not marketing pages.

---

## Canonical Project Paths

App repo:

```text
/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp
```

Dev Brain / source of truth:

```text
/Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi
```

Open Design source:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design
```

Open Design desktop app:

```text
/Users/arsenabduhalyk/Projects/DesignLab/Open Design.app
```

Open Design BIRGE legacy mockups project:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups
```

Original Dev Brain mockups archive:

```text
/Users/arsenabduhalyk/Documents/Dev_ArsensBrain/Projects/Birge Taxi/docs/mockups
```

Important rule:

```text
Do not edit the original Dev Brain mockups directly.
Use the Open Design working copy and save improved screens under improved/.
```

---

## How To Start Open Design

Run:

```bash
cd /Users/arsenabduhalyk/Projects/DesignLab/open-design
export PATH="$(printf '%s\n' "$PATH" | tr ':' '\n' | grep -v '^/Applications/Codex.app/Contents/Resources$' | awk '!seen[$0]++' | paste -sd: -)"
export PATH="$HOME/.local/bin:$PATH"
source "$HOME/.nvm/nvm.sh"
nvm use 24
hash -r
corepack pnpm tools-dev run web
```

Expected local URLs:

```text
Web UI:     http://127.0.0.1:58776/
Local API:  http://127.0.0.1:58774/
```

If `pnpm` is not found in a non-interactive terminal, use the `source "$HOME/.nvm/nvm.sh"` line above before running `corepack pnpm`.

Codex CLI must be available through the shim:

```text
~/.local/bin/codex -> /Applications/Codex.app/Contents/Resources/codex
```

Do not add `/Applications/Codex.app/Contents/Resources` directly to `PATH`, because it can override the nvm Node binary and break Open Design native dependencies.

---

## What Has Already Been Done

Open Design is installed at:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design
```

BIRGE design system was created at:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/design-systems/birge/DESIGN.md
```

Legacy BIRGE HTML/CSS mockups were copied from Dev Brain into Open Design as a separate project:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups
```

Open Design project name:

```text
BIRGE Legacy Mockups
```

Open Design project id:

```text
birge-legacy-mockups
```

The imported working copy lives here:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/legacy-birge-mockups
```

Driver mockup prompt is saved here:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/DRIVER_MOCKUPS_PROMPT.md
```

Passenger mockups decision:

```text
Do not rewrite passenger mockups now.
They are kept as a broad, visually polished product coverage prototype.
Use them as visual-quality reference only.
Do not copy their information density into driver mockups.
```

Driver mockup direction:

```text
As polished as passenger.
Much calmer and less dense.
One screen = one task.
One obvious primary CTA.
Max 3-4 major content blocks.
No long explanatory/product-pitch copy.
```

Driver pricing/positioning update:

```text
BIRGE Driver is a peak-hour income layer at MVP, not a full-day Yandex/InDrive replacement.
Drivers may keep using Yandex/InDrive outside BIRGE corridor windows.
Peak Starter is confirmed as a fourth driver plan at 9,900 KZT/month.
Driver subscription starts only when the first active corridor is ready.
```

Improved output folders:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/driver
```

The Open Design project also contains:

```text
README.md
IMPROVEMENT_PROMPT.md
index.html
legacy-birge-mockups/
improved/
```

---

## Product Context

BIRGE is **not taxi-first**.

BIRGE is a transportation-as-a-subscription product for recurring commuters in Almaty and Kazakhstan.

Core promise:

```text
Declare recurring monthly routes once.
BIRGE pre-plans drivers and co-passengers by corridor.
Pay a subscription through Kaspi.
Stop thinking about daily taxi ordering.
```

Current product direction:
- passenger app
- driver app
- subscription commute as the wedge
- passengers declare independent recurring routes
- hub-to-hub corridor model: one pickup node, one dropoff node, no intermediate stops in MVP
- on-demand ride only as fallback/acquisition, not the main product
- passenger subscription tiers: Solo Corridor, Multi Corridor, Flex Pack
- Multi Corridor is recommended/default for the first Данияр-type persona
- driver monetization is fixed monthly subscription only
- zero per-ride commission ever
- safety, reliability, and trust as core UX concerns
- future backend with Swift Vapor, PostgreSQL, Redis, WebSockets, OSRM/H3, and later ML
- iOS implementation target: SwiftUI, TCA, GRDB, native iOS polish

The Open Design work should help the iOS app become easier to implement and understand.

---

## Current Open Design Priority

Current active design lane:

```text
Passenger subscription commute mockup redesign
```

Current improved passenger screens:

```text
improved/passenger/P-04-home.html                 -> Today’s commute plan
improved/passenger/P-05-search.html               -> My monthly routes
improved/passenger/ride-request.html              -> Corridor detail / ride-day plan
improved/passenger/searching.html                 -> Corridor activation/status
improved/passenger/P-08-offer.html                -> Corridor ready confirmation
improved/passenger/subscription-selection.html    -> Subscription tier selection
```

These are useful but not complete. They mostly describe the post-subscription / corridor-ready state.

Next priority:

```text
Batch 1: First-time monthly commute onboarding
```

The next flow must show how a new passenger first declares monthly routes before choosing a subscription.

Target sequence:

```text
Welcome / positioning
-> Phone / OTP
-> Monthly commute route setup
-> Pickup/dropoff hub confirmation
-> Weekdays and time windows
-> Optional additional routes
-> Monthly route plan review
-> Corridor reliability / savings
-> Subscription selection
-> Kaspi activation
-> Today’s commute dashboard
```

Target device:

```text
iPhone 15 Pro first
```

Then check:

```text
small iPhone / iPhone SE layout risk
```

---

## Current Open Design File Plan

Already improved:

```text
improved/passenger/P-04-home.html
improved/passenger/P-05-search.html
improved/passenger/ride-request.html
improved/passenger/searching.html
improved/passenger/P-08-offer.html
improved/passenger/subscription-selection.html
```

Next files to redesign from legacy:

```text
legacy-birge-mockups/passenger/P-03-onboarding.html
legacy-birge-mockups/passenger/P-03a-commute-step1.html
legacy-birge-mockups/passenger/P-03b-commute-step2.html
legacy-birge-mockups/passenger/P-03c-commute-step3.html
legacy-birge-mockups/passenger/P-03d-commute-step4.html
legacy-birge-mockups/passenger/P-03e-commute-step5.html
```

Recommended improved output names:

```text
improved/passenger/P-03-onboarding.html
improved/passenger/P-03a-route-origin.html
improved/passenger/P-03b-route-destination.html
improved/passenger/P-03c-route-schedule.html
improved/passenger/P-03d-route-review.html
improved/passenger/P-03e-monthly-plan.html
```

Do not overwrite the files in `legacy-birge-mockups/`.

---

## Design Constraints

Keep intact:
- BIRGE product logic
- user-flow meaning
- screen purpose
- subscription commute positioning
- monthly route declaration
- pickup/dropoff hub intent
- Solo Corridor / Multi Corridor / Flex Pack subscription structure
- Kaspi payment context
- driver, safety, reliability, and corridor context

Improve:
- visual hierarchy
- native iOS feel
- map composition
- bottom-sheet ergonomics
- trust and safety clarity
- tappable controls
- compact but readable operational density
- SwiftUI implementation friendliness

Avoid:
- landing page style
- generic SaaS dashboard style
- decorative empty cards
- one-color monotone palette
- text overlap
- tiny unreadable labels
- taxi-first home screens
- making on-demand equal to subscription
- per-ride pricing language for Solo Corridor or Multi Corridor
- AI hype as the product promise
- changing the business logic without explicit approval
- editing Dev Brain originals

---

## Copy Language Direction

Use:

```text
Russian/Kazakh-market-friendly
```

This means:
- Russian UI copy is acceptable and preferred for first local-market mockups.
- Kazakhstan / Almaty context should feel natural.
- Avoid awkward literal translations.
- Keep copy short enough for mobile UI.
- If Kazakh appears, use it intentionally and sparingly unless asked to localize the full app.

Potential location examples:
- Dostyk Plaza
- Esentai Mall
- Mega Alma-Ata
- Al-Farabi
- Abay
- Satpayev
- Nazarbayev Ave
- Airport / ALA
- Kok-Tobe area
- Almaly / Bostandyk / Medeu districts

---

## Suggested Prompt For Open Design

Use this in the Open Design chat for one screen:

```text
You are improving a BIRGE mobile app mockup inside Open Design.

Project:
- BIRGE is a transportation-as-a-subscription product for recurring Almaty commuters.
- It is not taxi-first. On-demand taxi is only fallback/acquisition.
- The app has Passenger and Driver surfaces.
- The design target is premium native iOS, with a future SwiftUI implementation.
- Use the BIRGE design system direction already present in this Open Design workspace.

Task:
- Open and improve this file: <PATH_TO_LEGACY_HTML_SCREEN>.
- Preserve the screen's product intent, flow position, core content, and important labels.
- Improve layout, hierarchy, spacing, typography, affordances, iOS polish, and interaction clarity.
- Do not turn it into a landing page or marketing page.
- Keep it as a real usable app screen.
- Assume iPhone 15 Pro as the primary canvas.
- Use plausible Almaty route details if the screen needs pickup, destination, route, or map context.
- Preserve subscription-first logic: recurring routes, hub-to-hub pickup/dropoff, Kaspi subscription, and zero per-ride friction for Solo/Multi subscriptions.

Output:
- Create the improved screen as: <OUTPUT_PATH_UNDER_IMPROVED_FOLDER>.
- Keep the legacy file unchanged.
- Include all CSS needed for the improved screen to render correctly.
- Prefer a clean single-screen HTML artifact unless shared styles are clearly useful.

Quality bar:
- No overlapping text.
- No decorative empty cards.
- No one-color monotone palette.
- Controls should look tappable and native.
- Bottom sheets, route states, subscription cards, and corridor status cards should feel production-real.
- The design should be easy to translate into SwiftUI.
```

Example:

```text
Open and redesign legacy-birge-mockups/passenger/P-03a-commute-step1.html.
Create the improved version at improved/passenger/P-03a-route-origin.html.
Focus on monthly commute setup: explain that the user is creating a recurring route, choose Home / pickup hub near Alatau City, and keep taxi/on-demand out of the primary flow.
```

---

## Suggested First Full-Flow Prompt

Use this when asking a new AI chat to handle the next connected flow:

```text
Continue BIRGE Open Design work.

Redesign the first-time Passenger monthly commute onboarding flow using these legacy files:
- legacy-birge-mockups/passenger/P-03-onboarding.html
- legacy-birge-mockups/passenger/P-03a-commute-step1.html
- legacy-birge-mockups/passenger/P-03b-commute-step2.html
- legacy-birge-mockups/passenger/P-03c-commute-step3.html
- legacy-birge-mockups/passenger/P-03d-commute-step4.html
- legacy-birge-mockups/passenger/P-03e-commute-step5.html

Save improved versions under:
- improved/passenger/P-03-onboarding.html
- improved/passenger/P-03a-route-origin.html
- improved/passenger/P-03b-route-destination.html
- improved/passenger/P-03c-route-schedule.html
- improved/passenger/P-03d-route-review.html
- improved/passenger/P-03e-monthly-plan.html

Do not edit legacy files directly.

Primary goal:
- Make BIRGE’s subscription commute model clear before any ride-day screens.

Secondary goals:
- More native iOS feel.
- Cleaner hierarchy.
- Stronger trust/safety cues.
- Russian/Kazakh-market-friendly copy.
- Realistic Almaty / Kazakhstan context.
- Easy future SwiftUI implementation.

Keep the BIRGE product logic intact:
- passenger declares recurring monthly routes
- each route becomes an independent corridor
- MVP corridor is hub-to-hub: one pickup node, one dropoff node
- user chooses weekdays and time windows
- user can add multiple routes such as Home -> Work, Work -> Home, Work -> Gym, Gym -> Home
- user sees reliability / match confidence and monthly savings before subscription selection
- subscription selection comes after route plan review
- on-demand taxi remains fallback only

After each improved screen, briefly explain:
- what changed
- why it improves the product
- any iOS/SwiftUI implementation notes
```

---

## Recommended Work Split

Planning chat:

```text
strategy, priorities, product logic, roadmap, business/tech decisions
```

Open Design chat:

```text
mockups, visual design, design system, HTML prototypes, screen improvement
```

iOS implementation chat:

```text
SwiftUI/TCA implementation of approved designs
```

This separation keeps the planning chat clean and prevents design iteration history from overwhelming product/engineering planning.

---

## Completion Definition For A Design Pass

A pass is complete when:
- improved HTML files exist in `improved/passenger/` or `improved/driver/`
- legacy files remain unchanged
- the improved screens render correctly in Open Design
- the flow feels coherent across connected states
- no text overlaps
- buttons and controls are tappable
- bottom sheets and maps have clear hierarchy
- the result is ready to discuss before SwiftUI implementation

After completion, summarize:
- files changed
- key UX changes
- remaining questions
- recommended next screen or flow
