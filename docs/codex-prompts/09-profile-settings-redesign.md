# Codex Prompt 09 — Profile Redesign + Settings

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Файлы:
- `BIRGEPassenger/Features/Profile/ProfileView.swift`
- `BIRGEPassenger/Features/Profile/ProfileFeature.swift`

Мокапы: `P-19-profile.html`, `P-20-edit-profile.html`, `P-21-settings.html`, `P-22-privacy-settings.html`

**Требование: выполни промпты 01–08 сначала.**

---

## Анализ текущего ProfileView

Текущий ProfileView использует `List` с `.insetGrouped` стилем. Нужен полный редизайн:
- Убрать List, использовать ScrollView + VStack
- Добавить Liquid Glass карточки
- Большой аватар секции header
- Статы (рейтинг, поездки, сэкономлено)
- Секции настроек с иконками в стиле iOS Settings

---

## Задача A — ProfileView полный редизайн

Перепиши `ProfileView.swift` полностью.

### Структура:
```swift
struct ProfileView: View {
    let store: StoreOf<ProfileFeature>

    var body: some View {
        ScrollView(showsIndicators: false) {
            VStack(spacing: BIRGELayout.s) {
                // 1. HEADER — аватар + имя
                profileHeader

                // 2. STATS — 3 карточки
                statsRow

                // 3. СЕКЦИИ НАСТРОЕК
                settingsSection(title: "Аккаунт", items: accountItems)
                settingsSection(title: "Приложение", items: appItems)
                settingsSection(title: "Поддержка", items: supportItems)

                // 4. LOGOUT BUTTON
                BIRGEDestructiveButton(title: "Выйти из аккаунта") {
                    store.send(.logoutTapped)
                }
                .padding(.horizontal, BIRGELayout.m)
                .padding(.bottom, BIRGELayout.xl)
            }
            .padding(.top, BIRGELayout.s)
        }
        .background(BIRGEColors.surfaceGrouped)
        .navigationTitle("Профиль")
        .navigationBarTitleDisplayMode(.inline)
        .onAppear { store.send(.onAppear) }
    }
}
```

### profileHeader
```swift
private var profileHeader: some View {
    VStack(spacing: BIRGELayout.s) {
        // Аватар
        ZStack {
            Circle()
                .fill(BIRGEColors.brandPrimary.opacity(0.15))
                .frame(width: 90, height: 90)
            Text(store.name.prefix(1))
                .font(.system(size: 36, weight: .bold, design: .rounded))
                .foregroundStyle(BIRGEColors.brandPrimary)

            // Edit button overlay
            Circle()
                .fill(BIRGEColors.brandPrimary)
                .frame(width: 28, height: 28)
                .overlay(
                    Image(systemName: "pencil")
                        .font(.system(size: 12, weight: .semibold))
                        .foregroundStyle(.white)
                )
                .offset(x: 30, y: 30)
        }

        VStack(spacing: BIRGELayout.xxxs) {
            Text(store.name)
                .font(BIRGEFonts.title)
            Text(store.phone)
                .font(BIRGEFonts.subtext)
                .foregroundStyle(BIRGEColors.textSecondary)
        }
    }
    .frame(maxWidth: .infinity)
    .padding(.vertical, BIRGELayout.m)
    .birgeGlassCard()
    .padding(.horizontal, BIRGELayout.m)
}
```

### statsRow (3 карточки)
```swift
private var statsRow: some View {
    HStack(spacing: BIRGELayout.xs) {
        statCard(
            symbol: "star.fill",
            symbolColor: .yellow,
            value: String(format: "%.1f", store.rating),
            label: "Рейтинг"
        )
        statCard(
            symbol: "car.fill",
            symbolColor: BIRGEColors.brandPrimary,
            value: "\(store.totalRides)",
            label: "Поездок"
        )
        statCard(
            symbol: "leaf.fill",
            symbolColor: BIRGEColors.success,
            value: "12 кг",
            label: "CO₂ экономия"
        )
    }
    .padding(.horizontal, BIRGELayout.m)
}

private func statCard(symbol: String, symbolColor: Color, value: String, label: String) -> some View {
    VStack(spacing: BIRGELayout.xxs) {
        Image(systemName: symbol)
            .font(BIRGEFonts.sectionTitle)
            .foregroundStyle(symbolColor)
        Text(value)
            .font(BIRGEFonts.sectionTitle)
        Text(label)
            .font(BIRGEFonts.caption)
            .foregroundStyle(BIRGEColors.textSecondary)
    }
    .frame(maxWidth: .infinity)
    .padding(.vertical, BIRGELayout.s)
    .birgeGlassCard()
}
```

### settingsSection + settingsItem
```swift
struct SettingsItem: Identifiable {
    let id: UUID = UUID()
    let symbol: String
    let symbolColor: Color
    let title: String
    let subtitle: String?
    let badge: String?
    let action: () -> Void

    init(symbol: String, symbolColor: Color, title: String,
         subtitle: String? = nil, badge: String? = nil, action: @escaping () -> Void = {}) {
        self.symbol = symbol
        self.symbolColor = symbolColor
        self.title = title
        self.subtitle = subtitle
        self.badge = badge
        self.action = action
    }
}

private func settingsSection(title: String, items: [SettingsItem]) -> some View {
    VStack(alignment: .leading, spacing: BIRGELayout.xxs) {
        Text(title)
            .font(BIRGEFonts.captionBold)
            .foregroundStyle(BIRGEColors.textSecondary)
            .padding(.horizontal, BIRGELayout.m)
            .padding(.bottom, BIRGELayout.xxxs)

        VStack(spacing: 1) {
            ForEach(items) { item in
                settingsRow(item)
            }
        }
        .birgeGlassCard()
        .padding(.horizontal, BIRGELayout.m)
    }
}

private func settingsRow(_ item: SettingsItem) -> some View {
    Button(action: item.action) {
        HStack(spacing: BIRGELayout.s) {
            // SF Symbol иконка с цветным фоном
            ZStack {
                RoundedRectangle(cornerRadius: BIRGELayout.radiusXS)
                    .fill(item.symbolColor)
                    .frame(width: 32, height: 32)
                Image(systemName: item.symbol)
                    .font(.system(size: 15, weight: .semibold))
                    .foregroundStyle(.white)
            }

            VStack(alignment: .leading, spacing: 2) {
                Text(item.title)
                    .font(BIRGEFonts.body)
                    .foregroundStyle(BIRGEColors.textPrimary)
                if let subtitle = item.subtitle {
                    Text(subtitle)
                        .font(BIRGEFonts.caption)
                        .foregroundStyle(BIRGEColors.textSecondary)
                }
            }

            Spacer()

            if let badge = item.badge {
                Text(badge)
                    .font(.system(size: 12, weight: .semibold))
                    .foregroundStyle(.white)
                    .padding(.horizontal, BIRGELayout.xxs)
                    .padding(.vertical, 2)
                    .background(BIRGEColors.brandPrimary)
                    .clipShape(Capsule())
            }

            Image(systemName: "chevron.right")
                .font(BIRGEFonts.caption)
                .foregroundStyle(BIRGEColors.textTertiary)
        }
        .padding(.horizontal, BIRGELayout.s)
        .padding(.vertical, BIRGELayout.xs)
    }
    .buttonStyle(.plain)
}
```

### Данные секций
```swift
private var accountItems: [SettingsItem] {
    [
        SettingsItem(symbol: "person.fill", symbolColor: BIRGEColors.brandPrimary,
                     title: "Редактировать профиль") { },
        SettingsItem(symbol: "lock.fill", symbolColor: .gray,
                     title: "Конфиденциальность",
                     subtitle: "Управление данными") { },
        SettingsItem(symbol: "bell.fill", symbolColor: .orange,
                     title: "Уведомления", badge: "3") { }
    ]
}

private var appItems: [SettingsItem] {
    [
        SettingsItem(symbol: "globe", symbolColor: .blue,
                     title: "Язык", subtitle: "Русский") { },
        SettingsItem(symbol: "moon.fill", symbolColor: .indigo,
                     title: "Тема", subtitle: "Системная") { },
        SettingsItem(symbol: "brain", symbolColor: .purple,
                     title: "Как работает AI",
                     subtitle: "Как подбираются маршруты") { }
    ]
}

private var supportItems: [SettingsItem] {
    [
        SettingsItem(symbol: "questionmark.circle.fill", symbolColor: .gray,
                     title: "Помощь") { },
        SettingsItem(symbol: "doc.text.fill", symbolColor: .gray,
                     title: "Правовые документы") { },
        SettingsItem(symbol: "star.fill", symbolColor: .yellow,
                     title: "Оценить приложение") { }
    ]
}
```

---

## Задача B — AI Explanation Screen (P-23)

Создай простой `BIRGEPassenger/Features/Profile/AIExplanationView.swift`:

```swift
struct AIExplanationView: View {
    var body: some View {
        ScrollView(showsIndicators: false) {
            VStack(alignment: .leading, spacing: BIRGELayout.l) {
                // Hero
                VStack(spacing: BIRGELayout.s) {
                    Image(systemName: "brain")
                        .font(.system(size: 60))
                        .foregroundStyle(BIRGEColors.brandPrimary)
                    Text("Как работает AI BIRGE")
                        .font(BIRGEFonts.title)
                        .multilineTextAlignment(.center)
                    Text("Наш алгоритм обрабатывает тысячи маршрутов ежедневно")
                        .font(BIRGEFonts.body)
                        .foregroundStyle(BIRGEColors.textSecondary)
                        .multilineTextAlignment(.center)
                }
                .frame(maxWidth: .infinity)
                .padding(.top, BIRGELayout.xl)

                // Steps
                aiStep(number: "01", symbol: "location.fill",
                       title: "Анализ ваших маршрутов",
                       body: "AI изучает ваши регулярные поездки и предсказывает потребности")
                aiStep(number: "02", symbol: "person.2.fill",
                       title: "Подбор попутчиков",
                       body: "Алгоритм сопоставляет маршруты тысяч пассажиров для максимального совпадения")
                aiStep(number: "03", symbol: "chart.line.uptrend.xyaxis",
                       title: "Оптимизация цены",
                       body: "Динамическое ценообразование снижает стоимость поездки до 52%")
                aiStep(number: "04", symbol: "lock.shield.fill",
                       title: "Приватность данных",
                       body: "Ваши данные обрабатываются локально. Мы не продаём личную информацию")
            }
            .padding(.horizontal, BIRGELayout.m)
            .padding(.bottom, BIRGELayout.xl)
        }
        .background(BIRGEColors.background)
        .navigationTitle("AI BIRGE")
        .navigationBarTitleDisplayMode(.inline)
    }

    private func aiStep(number: String, symbol: String, title: String, body: String) -> some View {
        HStack(alignment: .top, spacing: BIRGELayout.s) {
            // Number + Icon
            VStack(spacing: BIRGELayout.xxxs) {
                Text(number)
                    .font(.system(size: 11, weight: .bold, design: .monospaced))
                    .foregroundStyle(BIRGEColors.brandPrimary)
                Image(systemName: symbol)
                    .font(BIRGEFonts.sectionTitle)
                    .foregroundStyle(BIRGEColors.brandPrimary)
            }
            .frame(width: 44)

            VStack(alignment: .leading, spacing: BIRGELayout.xxxs) {
                Text(title)
                    .font(BIRGEFonts.sectionTitle)
                Text(body)
                    .font(BIRGEFonts.body)
                    .foregroundStyle(BIRGEColors.textSecondary)
                    .fixedSize(horizontal: false, vertical: true)
            }
        }
        .padding(BIRGELayout.s)
        .birgeGlassCard()
    }
}
```

---

## Проверка

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```
