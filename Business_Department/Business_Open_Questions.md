---
status: living
version: v1.3
last_updated: 2026-05-09
source: Context/Universal_AI_Handoff_BIRGE.md v5.0 + Business Session 7 + Driver Utilization Risk + AI Legal Risk Assessment
---

# BIRGE Business Open Questions

This file tracks business, product, market, legal, and operations unknowns.

Do not treat these as solved until there is evidence.

## Critical

| ID | Question | Why It Matters | Status |
|---|---|---|---|
| BQ-001 | Will Almaty drivers accept fixed subscription instead of commission? | Core driver monetization | Unknown |
| BQ-002 | What passenger density makes one corridor viable? | Marketplace liquidity | Working assumption: 2-3 pax/run |
| BQ-003 | Will passengers accept fixed time + ~100m walk to pickup node? | Core behavior change | Unknown |
| BQ-004 | Legal status of paid shared rides in Kazakhstan | May block launch or require license | Unknown; lawyer needed |
| BQ-005 | Can Arsen open ИП/ТОО and get Kaspi merchant access? | Payments and contracts | Not started |
| BQ-006 | Is UvU Shuttle a direct competitor on first corridor? | Positioning and pricing | Not checked |
| BQ-023 | Does standard ОГПО cover paid BIRGE shared corridor rides? | Insurance may block first real corridor | Unknown; insurer/lawyer needed |
| BQ-024 | Is Kazakhstan data residency/localization required for BIRGE production data? | Backend hosting/legal compliance | Unknown; privacy/legal confirmation needed |
| BQ-025 | Is ИП sufficient for paid pilot or should BIRGE start with ТОО due to liability? | Legal entity and liability | Unknown; lawyer/tax consultant needed |

## High Priority

| ID | Question | Status |
|---|---|---|
| BQ-007 | What exactly is passenger pricing for MVP Solo Corridor, Multi Corridor, and Flex Pack? | Needed for UI |
| BQ-008 | What happens when a scheduled corridor has only 1 passenger? | Needed for UI/backend |
| BQ-009 | How should BIRGE communicate “not taxi, monthly commute plan”? | Needed for onboarding |
| BQ-010 | What are first 5 driver interview questions? | Needed for validation |
| BQ-011 | What are first 10 passenger interview questions? | Needed for validation |
| BQ-012 | What is the exact first pilot corridor operating model? | Needed before real launch |
| BQ-013 | Should on-demand appear on passenger home screen at all? | Needed for UI direction |
| BQ-014 | What are upgrade/downgrade paths between Solo Corridor, Multi Corridor, and Flex Pack? | Needed for MVP subscription UX |
| BQ-015 | How should Flex Pack balance, expiry, refunds, and unused rides work? | Needed for MVP Flex Pack |

## Recently Resolved / Working Decisions

These are accepted as current business truth, but can be revisited after real interviews or legal review.

| ID | Decision | Status |
|---|---|---|
| BD-001 | Minimum driver vehicle year for MVP mockups/business logic | 2000 or newer |
| BD-002 | Driver subscription activation timing | After first active corridor |
| BD-003 | Driver verification failure handling | Repeat submission / correction flow |
| BD-004 | Passenger late cancellation / no-show policy | Consumes ride/slot and protects driver; exact compensation accounting TBD |
| BD-005 | Driver exclusivity | Not required; parallel Yandex/InDrive work allowed |
| BD-006 | Driver MVP positioning | Peak-hour income layer, not full-day replacement |
| BD-007 | Peak Starter plan | Confirmed as fourth driver plan at 9,900 KZT/month |

## New Driver-Side Open Questions

| ID | Question | Why It Matters | Status |
|---|---|---|---|
| BQ-016 | Should 2000+ vehicle rule be tightened after MVP supply is secured? | Passenger trust and comfort | Later review |
| BQ-017 | What exact driver compensation accounting happens after late cancellation/no-show? | Driver trust, payments, subscription balance | Needs pricing/payment design |
| BQ-018 | What happens if a driver's first corridor is delayed after verification? | Churn prevention | Working rule: no billing until active corridor |
| BQ-019 | What are exact rejection reasons and allowed retry limits for KYC/documents? | Compliance and support workload | Needs ops/legal design |
| BQ-020 | How exactly does Peak Starter differ from Starter beyond price? | Pricing clarity and mockup copy | Needs final product definition |
| BQ-021 | When should the app offer upgrade to Professional? | Avoid overselling before enough active corridors | Working idea: after 2+ active corridors |
| BQ-022 | How to prevent BIRGE corridor conflicts with Yandex/InDrive jobs? | Passenger reliability | Needs launch ops/product policy |

## Legal / Compliance Blockers From AI Risk Assessment

Source: [[Business_Department/Legal_Compliance_Risk_Assessment]]

Do not treat these as confirmed legal advice. Treat them as priority questions for a licensed Kazakhstan lawyer.

| ID | Blocker | Why It Matters | Status |
|---|---|---|---|
| LB-001 | Register ИП or ТОО | Required before official payments, contracts, Kaspi merchant | Not started |
| LB-002 | Transport lawyer consultation | Need classification: aggregator vs carrier/operator | Not done |
| LB-003 | Insurance solution | Standard ОГПО may not cover paid passenger carriage | Unknown |
| LB-004 | Public offer + refund/no-show policy | Required before passenger subscription payments | Not drafted |
| LB-005 | Privacy policy + personal data consents/localization review | Required before collecting sensitive data/location/documents | Not done |

## Data Integrity Warning

The old 52-respondent survey used in academic/application contexts was self-filled and must not be presented as real market validation.

Allowed use:

```text
synthetic methodology demo
```

Not allowed:

```text
investor proof
market validation
real user research
```

Next required evidence:

```text
30-50 real Almaty commuter survey responses
10 real passenger interviews
5 real driver interviews
UvU Shuttle product test
transport lawyer consultation
```
