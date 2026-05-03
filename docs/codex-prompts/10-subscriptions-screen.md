# Codex Prompt 10 — Subscriptions Screen

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Мокапы: `P-17-subscriptions.html`, `P-18-subscription-detail.html`

**Требование: выполни промпты 01–09 сначала.**

---

## Задача A — SubscriptionPlan модель

Создай `BIRGEPassenger/Models/SubscriptionPlan.swift`:

```swift
import Foundation

struct SubscriptionPlan: Identifiable, Equatable {
    let id: String
    let name: String
    let priceMonthly: Int
    let priceYearly: Int       // со скидкой
    let savingsPercent: Int    // % экономии годового
    let features: [String]     // описание фич
    let isPopular: Bool
    let sfSymbol: String       // иконка плана

    var monthlyIfYearly: Int { priceYearly / 12 }

    static let plans: [SubscriptionPlan] = [
        .init(
            id: "basic",
            name: "Базовый",
            priceMonthly: 1990,
            priceYearly: 19900,
            savingsPercent: 17,
            features: [
                "До 20 поездок в месяц",
                "AI-подбор коридоров",
                "Приоритет при поиске",
                "Стандартная поддержка"
            ],
            isPopular: false,
            sfSymbol: "star.circle.fill"
        ),
        .init(
            id: "pro",
            name: "Pro",
            priceMonthly: 3490,
            priceYearly: 34900,
            savingsPercent: 17,
            features: [
                "Неограниченные поездки",
                "AI-подбор коридоров",
                "Высший приоритет поиска",
                "Бесплатная отмена 3×/мес",
                "Поддержка 24/7"
            ],
            isPopular: true,
            sfSymbol: "crown.fill"
        ),
        .init(
            id: "family",
            name: "Семейный",
            priceMonthly: 5990,
            priceYearly: 59900,
            savingsPercent: 16,
            features: [
                "До 4 участников",
                "Неограниченные поездки",
                "Общий AI-профиль маршрутов",
                "Семейная статистика",
                "Приоритетная поддержка"
            ],
            isPopular: false,
            sfSymbol: "person.3.fill"
        )
    ]
}
```

---

## Задача B — SubscriptionsFeature

Создай `BIRGEPassenger/Features/Subscriptions/SubscriptionsFeature.swift`:

```swift
@Reducer
struct SubscriptionsFeature {
    @ObservableState
    struct State: Equatable {
        var plans: [SubscriptionPlan] = SubscriptionPlan.plans
        var selectedPlan: SubscriptionPlan? = nil
        var billingPeriod: BillingPeriod = .monthly
        var activePlanId: String? = nil   // текущая подписка (nil = нет)
        var isPurchasing: Bool = false

        enum BillingPeriod: Equatable {
            case monthly
            case yearly
        }
    }

    enum Action: Sendable {
        case planSelected(SubscriptionPlan)
        case billingPeriodChanged(State.BillingPeriod)
        case subscribeTapped
        case purchaseCompleted
        case delegate(Delegate)

        enum Delegate: Sendable {
            case subscribed
        }
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .planSelected(let plan):
                state.selectedPlan = plan
                return .none
            case .billingPeriodChanged(let period):
                state.billingPeriod = period
                return .none
            case .subscribeTapped:
                state.isPurchasing = true
                return .run { send in
                    try await Task.sleep(for: .seconds(1.5))
                    await send(.purchaseCompleted)
                }
            case .purchaseCompleted:
                state.isPurchasing = false
                state.activePlanId = state.selectedPlan?.id
                return .send(.delegate(.subscribed))
            case .delegate:
                return .none
            }
        }
    }
}
```

---

## Задача C — SubscriptionsView

Создай `BIRGEPassenger/Features/Subscriptions/SubscriptionsView.swift`:

```swift
@ViewAction(for: SubscriptionsFeature.self)
struct SubscriptionsView: View {
    @Bindable var store: StoreOf<SubscriptionsFeature>
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        ScrollView(showsIndicators: false) {
            VStack(spacing: BIRGELayout.l) {
                // HERO HEADER
                heroHeader

                // BILLING TOGGLE
                billingToggle

                // PLAN CARDS
                planCards

                // FEATURE COMPARISON (если план выбран)
                if let plan = store.selectedPlan {
                    featureList(plan)
                }

                // CTA BUTTON
                if store.selectedPlan != nil {
                    BIRGEPrimaryButton(
                        title: store.isPurchasing ? "Оформляем..." : ctaTitle,
                        isLoading: store.isPurchasing
                    ) {
                        send(.subscribeTapped)
                    }
                    .padding(.horizontal, BIRGELayout.m)
                    .disabled(store.isPurchasing)
                }

                // LEGAL NOTE
                Text("Подписка автоматически продлевается. Отменить можно в любое время.")
                    .font(BIRGEFonts.caption)
                    .foregroundStyle(BIRGEColors.textSecondary)
                    .multilineTextAlignment(.center)
                    .padding(.horizontal, BIRGELayout.m)
                    .padding(.bottom, BIRGELayout.xl)
            }
            .padding(.top, BIRGELayout.s)
        }
        .background(BIRGEColors.background)
        .navigationTitle("Подписки")
        .navigationBarTitleDisplayMode(.inline)
    }

    // MARK: - Hero Header

    private var heroHeader: some View {
        VStack(spacing: BIRGELayout.xs) {
            ZStack {
                Circle()
                    .fill(BIRGEColors.brandPrimary.opacity(0.1))
                    .frame(width: 80, height: 80)
                Image(systemName: "crown.fill")
                    .font(.system(size: 36))
                    .foregroundStyle(BIRGEColors.brandPrimary)
            }
            Text("BIRGE Premium")
                .font(BIRGEFonts.title)
            Text("Неограниченные поездки, AI-маршруты и приоритетный поиск")
                .font(BIRGEFonts.body)
                .foregroundStyle(BIRGEColors.textSecondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal, BIRGELayout.xl)
        }
    }

    // MARK: - Billing Toggle

    private var billingToggle: some View {
        HStack(spacing: 0) {
            billingPill("Ежемесячно", period: .monthly)
            billingPill("Ежегодно", period: .yearly)
        }
        .padding(BIRGELayout.xxxs)
        .liquidGlass(.pill)
        .padding(.horizontal, BIRGELayout.m)
    }

    private func billingPill(_ title: String, period: SubscriptionsFeature.State.BillingPeriod) -> some View {
        let isActive = store.billingPeriod == period
        return Button {
            withAnimation(.spring(response: 0.3)) {
                send(.billingPeriodChanged(period))
            }
        } label: {
            VStack(spacing: 2) {
                Text(title)
                    .font(BIRGEFonts.captionBold)
                    .foregroundStyle(isActive ? .white : BIRGEColors.textSecondary)
                if period == .yearly {
                    Text("−17%")
                        .font(.system(size: 9, weight: .bold))
                        .foregroundStyle(isActive ? .white.opacity(0.8) : BIRGEColors.success)
                }
            }
            .frame(maxWidth: .infinity)
            .padding(.vertical, BIRGELayout.xxs)
            .background(isActive ? BIRGEColors.brandPrimary : Color.clear)
            .clipShape(Capsule())
        }
        .buttonStyle(.plain)
    }

    // MARK: - Plan Cards

    private var planCards: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: BIRGELayout.xs) {
                ForEach(store.plans) { plan in
                    planCard(plan)
                }
            }
            .padding(.horizontal, BIRGELayout.m)
        }
    }

    private func planCard(_ plan: SubscriptionPlan) -> some View {
        let isSelected = store.selectedPlan?.id == plan.id
        let isActive = store.activePlanId == plan.id

        return Button {
            withAnimation(.spring(response: 0.3)) {
                send(.planSelected(plan))
            }
        } label: {
            VStack(alignment: .leading, spacing: BIRGELayout.xs) {
                // Popular badge
                if plan.isPopular {
                    Text("Популярный")
                        .font(.system(size: 11, weight: .bold))
                        .foregroundStyle(.white)
                        .padding(.horizontal, BIRGELayout.xxs)
                        .padding(.vertical, 3)
                        .background(BIRGEColors.brandPrimary)
                        .clipShape(Capsule())
                }

                // Icon + Name
                Image(systemName: plan.sfSymbol)
                    .font(.system(size: 32))
                    .foregroundStyle(isSelected ? BIRGEColors.brandPrimary : BIRGEColors.textSecondary)

                Text(plan.name)
                    .font(BIRGEFonts.sectionTitle)
                    .foregroundStyle(BIRGEColors.textPrimary)

                // Price
                VStack(alignment: .leading, spacing: 2) {
                    let price = store.billingPeriod == .monthly
                        ? plan.priceMonthly
                        : plan.monthlyIfYearly
                    Text("\(price)₸")
                        .font(BIRGEFonts.heroNumber)
                        .foregroundStyle(isSelected ? BIRGEColors.brandPrimary : BIRGEColors.textPrimary)
                    Text("/ месяц")
                        .font(BIRGEFonts.caption)
                        .foregroundStyle(BIRGEColors.textSecondary)
                }

                if isActive {
                    Label("Активна", systemImage: "checkmark.circle.fill")
                        .font(BIRGEFonts.captionBold)
                        .foregroundStyle(BIRGEColors.success)
                }
            }
            .padding(BIRGELayout.m)
            .frame(width: 160, alignment: .leading)
            .liquidGlass(
                .card,
                tint: isSelected ? BIRGEColors.brandPrimary.opacity(0.1) : .clear
            )
            .overlay(
                RoundedRectangle(cornerRadius: BIRGELayout.radiusL)
                    .stroke(
                        isSelected ? BIRGEColors.brandPrimary : Color.clear,
                        lineWidth: 1.5
                    )
            )
        }
        .buttonStyle(.plain)
    }

    // MARK: - Feature List

    private func featureList(_ plan: SubscriptionPlan) -> some View {
        VStack(alignment: .leading, spacing: BIRGELayout.xs) {
            Text("Что включено в \(plan.name):")
                .font(BIRGEFonts.sectionTitle)

            ForEach(plan.features, id: \.self) { feature in
                HStack(spacing: BIRGELayout.xs) {
                    Image(systemName: "checkmark.circle.fill")
                        .foregroundStyle(BIRGEColors.success)
                        .font(BIRGEFonts.body)
                    Text(feature)
                        .font(BIRGEFonts.body)
                }
            }
        }
        .padding(BIRGELayout.m)
        .birgeGlassCard()
        .padding(.horizontal, BIRGELayout.m)
        .transition(.opacity.combined(with: .scale(scale: 0.98)))
    }

    // MARK: - Helpers

    private var ctaTitle: String {
        guard let plan = store.selectedPlan else { return "Выбрать план" }
        let price = store.billingPeriod == .monthly ? plan.priceMonthly : plan.priceYearly
        let period = store.billingPeriod == .monthly ? "/мес" : "/год"
        return "Подписаться за \(price)₸\(period)"
    }
}
```

---

## Задача D — Встроить в PassengerAppFeature

Добавь в `Path`:
```swift
case subscriptions(SubscriptionsFeature)
```

Обработай делегат из Home:
```swift
case .home(.delegate(.openSubscription)):
    state.path.append(.subscriptions(SubscriptionsFeature.State()))
    return .none
```

---

## Проверка

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```
