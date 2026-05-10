# Open Design Prompt — Passenger UX Cleanup

Use this prompt in Open Design for a focused cleanup pass on the BIRGE passenger mockups.

Source project:

```text
/Users/arsenabduhalyk/Projects/DesignLab/open-design/.od/projects/birge-legacy-mockups
```

Input files:

```text
improved/passenger/
```

Do not overwrite the existing files directly.

Create cleaned versions in:

```text
improved/passenger-clean/
```

---

You are improving BIRGE passenger app mockups.

BIRGE is a transportation-as-a-subscription platform for recurring Almaty commuters.
The existing passenger mockups cover many scenarios, but they currently feel too AI-generated:

- too much text
- too many blocks
- too many actions per screen
- unclear next tap
- too much business logic visible in the UI

Your job is not to add more screens.
Your job is to simplify the core passenger flow so it feels like a real native iOS app.

## Hard UX Rules

Every screen must answer within 3 seconds:

```text
Where am I?
What is happening?
What do I tap next?
```

Per-screen constraints:

- exactly one primary CTA
- max two secondary actions
- max four card-like blocks on normal screens
- max 40-70 visible words on normal screens
- no long explainer paragraphs
- no product-pitch sections
- no investor/business language
- no visible "fallback" label in UI copy
- no full-screen AI explanations unless absolutely necessary
- use native iOS hierarchy: forms, lists, concise cards, bottom sheets

Reduce or avoid these phrases in visible UI:

```text
плотность маршрута
активные коридоры
fallback
оценка основана на
AI заметил
планирование следующих поездок
по сравнению с ежедневными поездками вне подписки
```

Prefer simple passenger language:

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

## Cleanup Scope

Clean only the P0 passenger core flow first:

- `P-00-splash.html`
- `P-01-phone.html`
- `P-02-otp.html`
- `P-03a-route-origin.html`
- `P-03b-route-destination.html`
- `P-03c-route-schedule.html`
- `P-03d-route-review.html`
- `P-03e-monthly-plan.html`
- `subscription-selection.html`
- `subscription-kaspi.html`
- `subscription-success.html`
- `P-04-home.html`

Create:

- `improved/passenger-clean/index.html`
- `improved/passenger-clean/birge-passenger-clean.css`
- cleaned versions of the screens above

Keep links working in this order:

```text
P-00 -> P-01 -> P-02 -> P-03a -> P-03b -> P-03c -> P-03d -> P-03e -> subscription-selection -> subscription-kaspi -> subscription-success -> P-04
```

## Screen-Specific Direction

### Splash

Keep it calm and minimal.

Do not explain the whole product.

CTA:

```text
Начать
```

### Phone / OTP

Use standard native iOS auth flow.
No marketing copy.

### Route Origin / Destination / Schedule

Each screen should collect one thing.

Do not show savings, AI, route density, or subscription logic yet.

### Route Review

Show a compact route summary:

- from
- to
- days
- preferred time
- pickup/dropoff note

CTA:

```text
Продолжить
```

### Monthly Plan

Show:

- route is ready
- next step is choosing subscription
- one simple reliability/status line if needed

Remove:

- savings card
- AI insight card
- long explanation about how matching works

CTA:

```text
Выбрать подписку
```

### Subscription Selection

Show three plans, but simplify each plan:

1. Solo Corridor — one route
2. Multi Corridor — several regular routes
3. Flex Pack — irregular schedule

Each plan card:

- name
- one-line purpose
- price placeholder
- max two bullets

Recommended plan: Multi Corridor.

Primary CTA:

```text
Продолжить с Multi Corridor
```

Move full plan comparison out of this screen.

### Kaspi Payment

Make it clear:

- selected plan
- monthly charge placeholder
- payment method
- primary CTA to Kaspi

No long terms paragraph.

### Success

Show:

- subscription active
- first route is being prepared
- what user can do now

Primary CTA:

```text
Открыть план на сегодня
```

### Home

This is the most important cleanup.

Home should answer:

```text
What is my next commute and what do I do now?
```

Keep:

- today's date
- next commute route
- pickup time
- pickup point
- driver/status
- one primary CTA

Remove from first viewport:

- AI traffic note
- history link
- fallback taxi link
- long co-passenger explanation
- route management explanation

Primary CTA:

```text
Открыть поездку
```

Secondary action:

```text
Изменить маршрут
```

## Visual Direction

Keep BIRGE passenger blue accent `#037AFB`, but reduce decorative density.

Use:

- white/slate surfaces
- compact iOS typography
- clear spacing
- fewer nested cards
- calmer status chips
- native bottom CTA where appropriate

Do not create a marketing landing page.
Do not add external images.
Do not add emoji.

## Output

At the end, report:

- files created under `improved/passenger-clean/`
- what was simplified
- what was intentionally removed
- any remaining screens that still need cleanup

