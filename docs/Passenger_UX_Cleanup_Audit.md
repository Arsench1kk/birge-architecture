# Passenger UX Cleanup Audit

Last updated: 2026-05-09

Source inspected:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups/improved/passenger
```

## Verdict

The UI/UX designer's feedback is valid.

The passenger mockups are useful as a **product coverage draft**, but they are not ready as polished UI/UX screens.

Current state:

```text
Good for: mapping product scenarios, edge cases, subscription logic, support states.
Not good for: final app UX, visual confidence, click clarity, native iOS simplicity.
```

The main issue is not that AI generated the screens. The main issue is that the screens behave like AI output:

- too many blocks
- too many explanatory paragraphs
- too many parallel actions
- too much business logic visible to the user
- weak hierarchy between primary and secondary actions
- operational screens look like product documentation

## Evidence From Files

There are about 80 passenger HTML screens.

Text/action density examples:

| Screen | Approx words | Links | Card-like elements | UX risk |
|---|---:|---:|---:|---|
| `P-05-search.html` | 186 | 15 | 9 | Too many routes/actions on one screen |
| `P-17-subscriptions.html` | 152 | 14 | 13 | Subscription management looks like admin dashboard |
| `subscription-selection.html` | 148 | 3 | 22 | Plan choice is visually overloaded |
| `P-04-home.html` | 135 | 9 | 12 | Home has too many competing actions |
| `P-03e-monthly-plan.html` | 132 | 2 | 8 | Plan confirmation over-explains before CTA |
| `P-23-ai-assistant.html` | 140 | 3 | 13 | AI assistant feels like explanatory feature demo |
| `P-55-ai-repeated-route-detected.html` | 106 | 3 | 8 | AI insight screen over-explains privacy/reasoning |

This confirms the designer's points:

```text
"непонятно что и куда кликать"
"много текста"
"много блоков"
```

## Core UX Problems

### 1. Home Screen Tries To Do Too Much

Example: `P-04-home.html`.

The screen includes:
- subscription chip
- date/greeting
- next corridor
- 3 ETA tiles
- driver assignment
- co-passenger assignment
- traffic advice
- primary CTA
- evening corridor
- route management link
- history link
- fallback taxi link
- tab bar

Problem:

```text
The user cannot instantly know what matters now.
```

Home should answer one question:

```text
What is my next commute and what do I do now?
```

### 2. Route Management Screen Is Too Dense

Example: `P-05-search.html`.

The screen mixes:
- subscription status
- AI suggestion
- saved routes
- route management explanation
- savings explanation
- add route CTA
- history
- fallback taxi
- tab bar

Problem:

```text
It reads like a product pitch, not a route list.
```

### 3. Subscription Screens Show Too Much At Once

Examples:
- `subscription-selection.html`
- `P-17-subscriptions.html`
- `P-18-subscription-detail.html`

The plan selection screen has three large cards with repeated feature lists.
The management screen shows current plan, route coverage, plan options, and five actions.

Problem:

```text
The user sees comparison, management, education, and billing all at once.
```

### 4. AI Screens Feel Like Feature Demos

Examples:
- `P-55-ai-repeated-route-detected.html`
- `P-56-ai-pickup-node-suggestion.html`
- `P-58-ai-subscription-tier-recommendation.html`
- `P-61-ai-route-density-explanation.html`

Problem:

```text
AI reasoning is too visible. It sounds like the product is explaining itself to an investor, not helping a passenger.
```

Most AI states should be compact suggestions inside existing screens, not separate full screens.

### 5. Too Many Secondary Actions Look Equally Important

Common pattern:

```text
Primary CTA
Secondary CTA
Fallback link
History link
Support link
Details link
Tab bar item
```

Problem:

```text
The blue button is not enough if the rest of the screen competes with it.
```

### 6. UI Copy Exposes Internal Business Logic

Examples of user-facing phrases that should be reduced:

- "плотность маршрута"
- "коридорная подписка"
- "Multi Corridor покрывает до 4 активных коридоров"
- "fallback"
- "оценка основана на плотности маршрута"
- "по сравнению с ежедневными поездками вне подписки"

These are useful business concepts, but too heavy for first-pass app UX.

## P0 Cleanup Target

Do not clean all 80 screens first.

Create a polished **P0 passenger core flow** of about 10-12 screens:

1. `P-00-splash.html`
2. `P-01-phone.html`
3. `P-02-otp.html`
4. `P-03a-route-origin.html`
5. `P-03b-route-destination.html`
6. `P-03c-route-schedule.html`
7. `P-03d-route-review.html`
8. `P-03e-monthly-plan.html`
9. `subscription-selection.html`
10. `subscription-kaspi.html`
11. `subscription-success.html`
12. `P-04-home.html`

Then clean secondary screens:

1. `P-05-search.html`
2. `P-17-subscriptions.html`
3. `P-18-subscription-detail.html`
4. `P-09-driver-en-route.html`
5. `P-10-driver-arrived.html`
6. `P-11-ride-in-progress.html`
7. `P-12-ride-complete.html`

All other screens should stay as scenario coverage until the core flow feels real.

## Cleanup Rules

Use these as hard constraints for Open Design and future UI review.

### Per-Screen Budget

| Item | Rule |
|---|---|
| Primary CTA | exactly 1 |
| Secondary actions | max 2 |
| Card-like blocks | max 4 on normal screens |
| Text | max 40-70 visible words on normal screens |
| Explainer paragraphs | avoid; move to "Подробнее" |
| Tab bar | only on post-subscription main app screens |
| AI explanation | compact suggestion only, not full essay |

### Hierarchy Rule

Every screen must answer in 3 seconds:

```text
Where am I?
What is happening?
What should I tap next?
```

If the answer is not obvious, the screen fails.

### Copy Rule

Prefer:

```text
Сегодня
Маршруты
Подписка
Посадка
Водитель
Оплата
Изменить
Продолжить
```

Avoid or reduce:

```text
плотность
fallback
активные коридоры
оценка основана на
AI заметил
планирование следующих поездок
по сравнению с ежедневными поездками вне подписки
```

### Structure Rule

Do not make screens explain the entire BIRGE model.

Each screen needs one job:

```text
select route
confirm schedule
choose plan
pay
see today's commute
manage routes
handle ride-day state
```

## Specific Fix Directions

### `P-04-home.html`

Reduce to:
- greeting/date
- one next commute card
- pickup time / pickup point / driver state
- one CTA: "Открыть поездку"
- one small secondary: "Изменить маршрут"

Remove from first viewport:
- AI traffic note
- history link
- fallback taxi link
- evening corridor detail if it crowds the screen

### `P-05-search.html`

Rename conceptually:

```text
Мои маршруты
```

Reduce to:
- list of saved routes
- status chip per route
- one CTA: "Добавить маршрут"

Move to secondary screens:
- savings explanation
- AI suggestion
- fallback taxi
- history

### `subscription-selection.html`

Reduce plan cards:
- plan name
- one-line purpose
- price placeholder
- 2 key bullets max

Move full comparison to separate screen or bottom sheet.

Keep one primary CTA:

```text
Продолжить с Multi Corridor
```

### `P-17-subscriptions.html`

Split into:
- Current plan summary
- Covered routes
- Billing next payment

Move plan comparison and actions to detail screens.

### AI Screens

Do not show 7 separate AI screens as primary UX.

Keep AI as:
- small recommendation row
- one bottom-sheet explanation when tapped
- no long "why this helps" paragraphs in main flow

## Driver Mockup Lesson

Before starting driver mockups, apply this rule:

```text
Do not let Open Design create explanatory AI-looking screens.
Make every driver screen compact, task-first, and CTA-obvious.
```

Driver prompt must explicitly include:

- one primary CTA per screen
- max 2 secondary actions
- no long educational copy
- no product-pitch blocks
- operational native iOS forms/lists over decorative cards
- screen must be understandable in 3 seconds

## Recommended Next Step

Decision update after review:

```text
Do not rewrite passenger mockups now.
Keep them as a visually polished product coverage prototype.
Use this audit as a lesson for driver mockups and as optional future cleanup reference.
```

Active next step:

1. Start driver onboarding mockups.
2. Use passenger mockups as visual-quality reference only.
3. Apply this audit's density constraints to driver screens so they stay beautiful but calmer.
