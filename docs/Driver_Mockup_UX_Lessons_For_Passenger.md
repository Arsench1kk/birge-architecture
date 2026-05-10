# Driver Mockup UX Lessons For Future Passenger Cleanup

Last updated: 2026-05-10

Purpose: capture the UX issues discovered while polishing the improved driver mockups so they can later be applied carefully to the existing 78 passenger screens without deleting product coverage.

## Context

Passenger mockups are currently frozen as a broad product coverage prototype. Do not rewrite or reduce the passenger set by default.

Future passenger cleanup should preserve all 78 screens, but make each screen lighter, clearer, and less AI-generated.

## Lessons From Driver Mockups

### 1. Bottom Tab Bar Must Be A Real App Shell

Issue found in driver Phase 2:
- tab bar existed visually but was not attached to the bottom safe-area zone
- it sat too high because it was part of the flex layout before the home indicator

Fix pattern:
- on main tab screens, make tab bar absolute inside `.screen`
- keep home indicator at the true bottom
- add bottom padding to scrollable content so it does not hide behind the tab bar

Passenger implication:
- passenger tab bar should be audited the same way
- tab bar should not feel like another card or floating block

### 2. Header Subtitles Should Earn Their Place

Issue found:
- many headers had generic subtitles like “Плановые коридоры”, “Профиль машины”, “Сегодня · 07:45”
- these made screens feel generated and overexplained

Rule:
- keep subtitle only if it adds useful operational context
- remove subtitles that repeat the title or repeat content in the main card

Passenger implication:
- audit every passenger `<h1> + subtitle` pair
- remove filler descriptions and keep only useful state/timing/payment information

### 3. Stop Using Cards For Everything

Issue found:
- Open Design tends to wrap every idea in a card
- too many cards make screens look like AI-generated dashboards

Better patterns:
- map surface + bottom sheet
- native list rows with dividers
- settings list rows
- plain sections with whitespace
- compact fact rows
- inline chips only where they are real controls/statuses

Passenger implication:
- preserve 78 screens, but reduce card density screen-by-screen
- especially audit Home, Routes, Subscription, AI suggestion, Support, and Billing screens

### 4. Bottom Notes Usually Add Noise

Issue found:
- onboarding screens had bottom notes explaining obvious things
- these repeated passenger density problems

Rule:
- remove standalone bottom notes unless legally or operationally critical
- move critical info into field-level helper text or header subtitle

Passenger implication:
- remove extra explanatory notes under CTA areas
- avoid defensive product copy below primary actions

### 5. One Screen, One Job

Issue found:
- when a screen mixes route status, education, subscription logic, and support, next tap becomes unclear

Rule:
- each screen answers in 3 seconds:
  - Where am I?
  - What is happening?
  - What do I tap next?

Passenger implication:
- future cleanup should not delete scenarios, but should simplify each screen's job

### 6. Avoid User-Facing Internal Language

Issue found:
- terms like density, platform logic, planning model, and internal product explanations make UI feel business-facing

Rule:
- use operational user language
- prefer “Сегодня”, “Маршруты”, “Посадка”, “Водитель”, “Оплата”, “Изменить”, “Продолжить”

Passenger implication:
- remove visible words like “fallback”, “плотность маршрута”, “AI заметил”, and long reasoning explanations from primary surfaces

## Future Passenger Cleanup Strategy

Do not create `passenger-clean` again unless explicitly requested.

Recommended future approach:
1. Keep `improved/passenger/` and all 78 screens.
2. Work in small batches of 3-5 screens.
3. Do not change product coverage or navigation unless broken.
4. Apply one cleanup theme at a time:
   - header subtitle cleanup
   - card density reduction
   - bottom note removal
   - tab bar/safe-area correction
   - CTA clarity
5. Review after each batch before continuing.

## Open Design Prompt Principles

Future passenger prompts should say:
- preserve existing files and screen count
- do not create a separate passenger-clean folder
- do not redesign the product flow
- reduce visible explanation, not product capability
- replace unnecessary cards with native lists/rows/sections
- keep one primary CTA per screen
- keep all links working



## Driver Checkpoint Applied — 2026-05-09

Driver mockups now provide a cleaner reference for passenger cleanup:
- 47 driver screens, `D-00` through `D-46`, are checkpointed in Open Design.
- The best reusable lessons are native list rows, row icons instead of text actions, circular avatars, compact headers, bottom tab discipline, and targeted service-zone onboarding around home/base start points.
- Passenger cleanup should not rewrite `improved/passenger/`; use a separate copy such as `improved/passenger-polished/`.

## Passenger Polished Checkpoint Applied — 2026-05-10

The driver lessons were applied to a separate passenger working copy:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger-polished
```

Result:
- all 78 passenger screens were preserved
- broad product coverage was not reduced to MVP-only screens
- screens were improved in batches with targeted QA after each batch
- the original `improved/passenger/` was not the final polish target

Most important new lesson:

```text
“Remove cards” is not a sufficient Open Design instruction.
A good prompt must name the replacement composition.
```

Successful replacement compositions:
- route canvas / mini map route preview
- subscription pass and ticket/receipt surfaces
- account ledger with thin dividers
- timeline/status rail
- iOS wheel time picker
- horizontal payment carousel
- icon action rail
- map/node canvas with native selector
- live activity-like ride surfaces
- inline insight text instead of benefit/tradeoff tables

Prompts that worked best were concrete and visual:
- “Use a subscription pass + account ledger”
- “Use a route canvas with two nodes and a compact schedule strip”
- “Use an iOS-style wheel picker like Clock/Alarm”
- “Use a horizontal payment picker with Apple Pay, saved card, Kaspi, and card option”

Future SwiftUI mapping should preserve these patterns instead of translating every HTML surface into generic cards/lists.
