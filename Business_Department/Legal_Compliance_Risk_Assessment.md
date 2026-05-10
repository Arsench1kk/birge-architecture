---
status: active
version: v1.0
last_updated: 2026-05-09
source: AI legal/compliance chat output based on Universal_AI_Handoff_BIRGE.md v5.0
disclaimer: AI-generated risk map, not legal advice
---

# BIRGE Legal And Compliance Risk Assessment

This document summarizes an AI-generated legal risk assessment for BIRGE in Kazakhstan.

Important:

```text
This is not legal advice.
Use this as a preparation document for a licensed Kazakhstan transport lawyer, tax consultant, data/privacy consultant, and payment provider consultation.
```

## Executive Summary

BIRGE has serious legal/compliance blockers before real passengers, real drivers, and real payments.

The biggest risks:

1. Paid recurring shared corridors may be classified as commercial passenger transportation, not simple carpooling.
2. BIRGE may be treated as more than a passive aggregator because it pre-plans routes, schedules, drivers, and passengers.
3. Standard driver ОГПО may not cover paid passenger transportation in a personal vehicle.
4. Passenger subscription, no-show rules, and refunds require a public offer and explicit acceptance before payment.
5. Kazakhstan personal data rules may require local data residency and formal privacy/compliance steps.

## Confidence Labels

Use these labels:

| Label | Meaning |
|---|---|
| Confirmed legal fact | Based on publicly known legal requirements, still verify with lawyer |
| Likely interpretation | Plausible legal reading, needs licensed lawyer confirmation |
| Unknown | Requires lawyer/regulator/payment provider confirmation |

## 1. Shared Ride Legality

Likely interpretation:

```text
A private driver carrying multiple paying passengers on a fixed recurring route may be treated as commercial passenger transportation.
```

Risk:

```text
High.
```

Why:
- BIRGE has fixed recurring corridors.
- Passengers pay for service.
- Driver carries multiple paying passengers.
- Hub-to-hub corridor with schedule may look like route transport / marshrutka-style operation.

Open legal questions:
- Is there a legal carpooling exception in Kazakhstan that can apply to BIRGE?
- Can BIRGE structure the service as aggregator/marketplace without becoming carrier/operator?
- Does subscription payment change classification? AI legal assessment says likely no.

## 2. Platform Classification

Possible classifications:

| Classification | Risk |
|---|---|
| Aggregator | Lower burden, but BIRGE may be doing too much planning for this to be clean |
| Marketplace / IT platform | Medium burden, but payment/routing/scheduling still create risk |
| Dispatcher / transport operator | Higher burden |
| Carrier | Highest burden |

Likely issue:

```text
BIRGE does more than a passive marketplace: it plans corridors, schedules, matching, and payments.
```

This may make the regulator view BIRGE as a transport operator or dispatcher.

## 3. Insurance Risk

Potential blocker:

```text
Standard ОГПО may not cover paid passenger transportation in a personal vehicle.
```

This is a serious legal, financial, and reputational risk.

Before first real corridor:
- confirm whether standard ОГПО covers BIRGE rides
- find commercial passenger insurance if needed
- define who must hold insurance: driver, BIRGE, or both
- reflect this in driver onboarding and contracts

## 4. Legal Entity

AI legal assessment recommendation:

| Stage | Recommended entity |
|---|---|
| Small pilot | ИП may be fast and enough to start administrative/payment setup |
| Investor-ready / scale | ТОО is likely required |

Important:

```text
No real payments, Kaspi merchant contract, driver contracts, or official revenue without ИП/ТОО.
```

Open questions:
- Is ИП enough for actual pilot with paid shared passenger rides?
- Should BIRGE skip ИП and create ТОО earlier due to liability/transport risk?
- Which tax regime applies best?

## 5. Kaspi And Payments

Known requirements likely include:
- ИП/ТОО and БИН
- bank account
- merchant agreement
- business activity description
- public offer/refund terms

Payment blockers:
- no Kaspi merchant without legal entity
- no real passenger subscriptions without public offer and refund/no-show policy
- recurring payment support must be confirmed directly with Kaspi or alternative processor

Alternative processors to investigate:
- Halyk / Homebank
- Forte
- PayBox / Wooppay
- CloudPayments KZ

## 6. Driver Relationship

Current BIRGE position:

```text
Driver is independent contractor / platform subscriber, not employee.
```

Helpful facts:
- driver pays BIRGE subscription
- BIRGE takes 0% per-ride commission
- driver can work on Yandex/InDrive
- no exclusivity

Risky facts:
- BIRGE plans routes and schedules
- BIRGE matches drivers/passengers
- BIRGE may control access to corridors

Contract direction:

```text
Use civil-law service/partner agreement, not employment contract.
Explicitly state independent contractor status, own vehicle, own tax obligations, and non-exclusivity.
```

Needs lawyer confirmation:
- whether BIRGE becomes tax agent under driver agreements
- whether platform controls create employment risk

## 7. Passenger Subscription And No-Show Policy

Working no-show policy:

| Passenger action | Consequence |
|---|---|
| Cancels more than 2 hours before ride | Free cancellation, slot returns |
| Cancels 30 min to 2 hours before ride | First time warning/free; repeated cases may consume slot |
| Cancels less than 30 minutes before ride | One ride/slot is consumed |
| No-show | One ride/slot is consumed + warning |
| 3 no-shows in one month | Temporary subscription suspension/review |

Legal/compliance requirement:

```text
These rules must be explicitly accepted before payment.
```

Minimum needed before charging:
- public offer
- refund policy
- no-show/cancellation policy
- checkbox acceptance before payment
- clear subscription cancellation terms

## 8. Driver Verification

Documents to verify:
- driver's license
- vehicle registration
- ОГПО
- technical inspection if applicable
- ID / identity verification
- insurance status

Internal quality rule:

```text
Vehicle year 2000+ is a BIRGE quality rule, not necessarily a legal rule.
```

Potential added checks:
- criminal record / background check
- medical requirements if legally required for commercial passenger carriage

## 9. Data Protection And Privacy

High-risk data:
- phone
- name
- ID documents
- driver documents
- live location
- route history
- payment/subscription events

Likely obligations:
- privacy policy in Russian
- explicit consent for personal data processing
- separate clear consent for location tracking
- do not store payment card data directly
- data retention/deletion policy
- Kazakhstan data residency/localization requirements must be confirmed
- possible registration/notification as personal data operator

Potential blocker:

```text
If production backend stores Kazakhstan citizens' personal data outside Kazakhstan, this may violate data localization requirements.
```

Needs legal/privacy confirmation.

## Top Legal Blockers Before First Real Corridor

### 1. Legal Entity

Need ИП or ТОО before:
- Kaspi merchant
- official revenue
- driver contracts
- passenger subscription payments

### 2. Insurance

Need confirmation that paid BIRGE corridor rides are covered.

If standard ОГПО does not cover paid passenger transportation, BIRGE must solve insurance before first real passenger ride.

### 3. Transport Lawyer Consultation + Public Offer / Privacy

Need:
- licensed Kazakhstan transport lawyer opinion
- platform classification strategy
- driver agreement template
- passenger public offer
- no-show/refund policy
- privacy policy and data processing consents

## Blocker / Important / Future Action List

### Blocker

| Action | Why |
|---|---|
| Register ИП or ТОО | Needed for payments/contracts |
| Transport lawyer consultation | Determines aggregator vs carrier/operator risk |
| Insurance solution | Standard ОГПО may not cover paid rides |
| Public offer + refund/no-show policy | Required before passenger payments |
| Privacy policy + data consent/localization review | Required before collecting sensitive data |

### Important Within 3 Months

| Action |
|---|
| Start Kaspi merchant process |
| Prepare driver civil-law agreement |
| Confirm tax regime |
| Confirm data hosting location |
| Confirm personal data operator obligations |

### Future Before Scaling

| Action |
|---|
| Convert/structure as ТОО before investment |
| Full compliance audit |
| Explore regulatory sandbox |
| Formal driver background-check policy |
| IP assignment / NDA for contributors |

## Impact On Product And Mockups

Open Design / mockup work can continue, but must not imply legal launch readiness.

Avoid copy like:

```text
Работайте уже сегодня
Вы готовы перевозить пассажиров
Начните поездки сразу после проверки
```

Prefer:

```text
После проверки мы уведомим, когда ваш первый коридор будет готов.
Перед запуском поездок BIRGE проверяет документы, страховку и правила работы.
```

Driver onboarding should include calm verification/compliance language, but not legal scare text.

