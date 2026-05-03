# Codex Prompt 07 — Corridor List + Corridor Detail

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Мокапы: `P-06a-corridor-list.html`, `P-06b-corridor-detail.html`

**Требование: выполни промпты 01–06 сначала.**

---

## Задача A — CorridorListFeature + CorridorListView

Создай два файла:
- `BIRGEPassenger/Features/Corridor/CorridorListFeature.swift`
- `BIRGEPassenger/Features/Corridor/CorridorListView.swift`

### CorridorListFeature

```swift
// BIRGEPassenger/Features/Corridor/CorridorListFeature.swift
import ComposableArchitecture

@Reducer
struct CorridorListFeature {
    @ObservableState
    struct State: Equatable {
        var corridors: [CorridorOption] = CorridorOption.mockData
        var selectedFilter: Filter = .all
        var aiSummary: String = "AI нашёл 5 коридоров по вашим маршрутам"

        enum Filter: String, CaseIterable, Equatable {
            case all = "Все"
            case morning = "Утро 07–09"
            case evening = "Вечер 17–20"
            case nearby = "Рядом со мной"
            case affordable = "до 1000₸"
        }

        var filteredCorridors: [CorridorOption] {
            switch selectedFilter {
            case .all: return corridors
            case .morning: return corridors   // filter by time — stub
            case .evening: return corridors
            case .nearby: return corridors
            case .affordable: return corridors.filter { $0.price <= 1000 }
            }
        }
    }

    @CasePathable
    enum Action: Sendable {
        case filterSelected(State.Filter)
        case corridorTapped(CorridorOption)
        case delegate(Delegate)

        enum Delegate: Sendable {
            case corridorSelected(CorridorOption)
            case back
        }
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .filterSelected(let filter):
                state.selectedFilter = filter
                return .none
            case .corridorTapped(let corridor):
                return .send(.delegate(.corridorSelected(corridor)))
            case .delegate:
                return .none
            }
        }
    }
}
```

### CorridorListView

```swift
// BIRGEPassenger/Features/Corridor/CorridorListView.swift
@ViewAction(for: CorridorListFeature.self)
struct CorridorListView: View {
    @Bindable var store: StoreOf<CorridorListFeature>

    var body: some View {
        ZStack(alignment: .top) {
            // BACKGROUND
            BIRGEColors.background
                .ignoresSafeArea()

            // CONTENT
            VStack(spacing: 0) {
                // FILTER BAR
                filterBar
                    .padding(.top, BIRGELayout.xxs)

                // AI SUMMARY PILL
                BIRGEAIPill(store.aiSummary)
                    .padding(.top, BIRGELayout.xxs)
                    .padding(.horizontal, BIRGELayout.m)
                    .frame(maxWidth: .infinity, alignment: .leading)

                // LIST
                ScrollView(showsIndicators: false) {
                    LazyVStack(spacing: BIRGELayout.xs) {
                        ForEach(store.filteredCorridors) { corridor in
                            corridorCard(corridor)
                        }

                        // Footer
                        VStack(spacing: BIRGELayout.xxs) {
                            Text("Это все коридоры по вашим маршрутам")
                                .font(BIRGEFonts.caption)
                                .foregroundStyle(BIRGEColors.textSecondary)
                            Button("Добавить новый маршрут") { }
                                .font(BIRGEFonts.captionBold)
                                .foregroundStyle(BIRGEColors.brandPrimary)
                        }
                        .padding(.vertical, BIRGELayout.m)
                    }
                    .padding(.horizontal, BIRGELayout.m)
                    .padding(.top, BIRGELayout.xs)
                }
            }
        }
        .navigationTitle("Коридоры")
        .navigationBarTitleDisplayMode(.inline)
        .navigationBarBackButtonHidden(false)
    }

    // MARK: - Filter Bar

    private var filterBar: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: BIRGELayout.xxs) {
                ForEach(CorridorListFeature.State.Filter.allCases, id: \.self) { filter in
                    filterPill(filter)
                }
            }
            .padding(.horizontal, BIRGELayout.m)
        }
    }

    private func filterPill(_ filter: CorridorListFeature.State.Filter) -> some View {
        let isActive = store.selectedFilter == filter
        Button {
            withAnimation(.spring(response: 0.3)) {
                send(.filterSelected(filter))
            }
        } label: {
            Text(filter.rawValue)
                .font(BIRGEFonts.captionBold)
                .foregroundStyle(isActive ? .white : BIRGEColors.brandPrimary)
                .padding(.horizontal, BIRGELayout.s)
                .padding(.vertical, BIRGELayout.xxs)
                .background(isActive ? BIRGEColors.brandPrimary : BIRGEColors.brandPrimary.opacity(0.1))
                .clipShape(Capsule())
        }
        .buttonStyle(.plain)
    }

    // MARK: - Corridor Card

    private func corridorCard(_ corridor: CorridorOption) -> some View {
        Button { send(.corridorTapped(corridor)) } label: {
            VStack(alignment: .leading, spacing: BIRGELayout.xs) {
                // TOP ROW: AI badge + seats badge
                HStack {
                    BIRGEMatchBadge(corridor.matchPercent)
                    Spacer()
                    seatsBadge(corridor)
                }

                // ROUTE
                VStack(alignment: .leading, spacing: BIRGELayout.xxxs) {
                    // Откуда
                    HStack(spacing: BIRGELayout.xs) {
                        Circle()
                            .fill(BIRGEColors.brandPrimary)
                            .frame(width: 8, height: 8)
                        Text(corridor.name.components(separatedBy: " → ").first ?? "")
                            .font(BIRGEFonts.sectionTitle)
                    }

                    // Dashed line
                    Rectangle()
                        .fill(BIRGEColors.brandPrimary.opacity(0.3))
                        .frame(width: 1.5, height: 16)
                        .padding(.leading, 3)

                    // Куда
                    HStack(spacing: BIRGELayout.xs) {
                        Circle()
                            .fill(BIRGEColors.brandPrimary)
                            .frame(width: 8, height: 8)
                        Text(corridor.name.components(separatedBy: " → ").last ?? "")
                            .font(BIRGEFonts.sectionTitle)
                    }
                }

                // STATS ROW
                HStack(spacing: BIRGELayout.s) {
                    Label(corridor.departure, systemImage: "clock.fill")
                        .font(BIRGEFonts.caption)
                        .foregroundStyle(BIRGEColors.textSecondary)

                    Label("\(corridor.seatsLeft)/\(corridor.seatsTotal)", systemImage: "person.2.fill")
                        .font(BIRGEFonts.captionBold)
                        .foregroundStyle(BIRGEColors.brandPrimary)

                    Label("\(corridor.price)₸", systemImage: "tengesign.circle.fill")
                        .font(BIRGEFonts.captionBold)
                        .foregroundStyle(BIRGEColors.textPrimary)
                }

                Divider()

                // PASSENGER AVATARS + CTA
                HStack {
                    // Avatar stack
                    HStack(spacing: -BIRGELayout.xxs + 2) {
                        ForEach(corridor.passengerInitials.prefix(3), id: \.self) { initial in
                            Text(initial)
                                .font(.system(size: 11, weight: .bold))
                                .foregroundStyle(.white)
                                .frame(width: 30, height: 30)
                                .background(BIRGEColors.brandPrimary.opacity(0.8))
                                .clipShape(Circle())
                                .overlay(Circle().stroke(.white, lineWidth: 2))
                        }
                    }

                    if corridor.seatsLeft > 0 {
                        Text("+\(corridor.seatsLeft) свободно")
                            .font(BIRGEFonts.caption)
                            .foregroundStyle(BIRGEColors.textSecondary)
                    }

                    Spacer()

                    Label("Подробнее", systemImage: "chevron.right")
                        .font(BIRGEFonts.captionBold)
                        .foregroundStyle(BIRGEColors.brandPrimary)
                }
            }
            .padding(BIRGELayout.m)
            .birgeGlassCard()
        }
        .buttonStyle(.plain)
    }

    private func seatsBadge(_ corridor: CorridorOption) -> some View {
        let hasSeats = corridor.seatsLeft > 0
        return Text(hasSeats ? "\(corridor.seatsLeft) место" : "Занято")
            .font(BIRGEFonts.captionBold)
            .foregroundStyle(hasSeats ? Color(hex: "065F46") : BIRGEColors.textSecondary)
            .padding(.horizontal, BIRGELayout.xs)
            .padding(.vertical, BIRGELayout.xxxs)
            .background(hasSeats ? Color(hex: "D1FAE5") : BIRGEColors.surfacePrimary)
            .clipShape(Capsule())
    }
}
```

---

## Задача B — CorridorDetailFeature + CorridorDetailView

Создай файлы:
- `BIRGEPassenger/Features/Corridor/CorridorDetailFeature.swift`
- `BIRGEPassenger/Features/Corridor/CorridorDetailView.swift`

Мокап: `P-06b-corridor-detail.html` — детальная карточка коридора с картой, участниками, кнопкой Join.

```swift
// CorridorDetailFeature.swift
@Reducer
struct CorridorDetailFeature {
    @ObservableState
    struct State: Equatable {
        let corridor: CorridorOption
        var isJoining: Bool = false
    }

    enum Action: Sendable {
        case joinTapped
        case delegate(Delegate)

        enum Delegate: Sendable {
            case joined
            case back
        }
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .joinTapped:
                state.isJoining = true
                return .run { send in
                    try await Task.sleep(for: .seconds(1.0))
                    await send(.delegate(.joined))
                }
            case .delegate:
                return .none
            }
        }
    }
}
```

```swift
// CorridorDetailView.swift
@ViewAction(for: CorridorDetailFeature.self)
struct CorridorDetailView: View {
    @Bindable var store: StoreOf<CorridorDetailFeature>
    @State private var position: MapCameraPosition = .automatic

    var body: some View {
        ZStack(alignment: .top) {
            // Map (top half)
            Map(position: $position)
                .mapStyle(.standard(elevation: .realistic))
                .mapControls { }
                .frame(height: 300)
                .ignoresSafeArea(edges: .top)

            // Content sheet
            VStack(alignment: .leading, spacing: 0) {
                Spacer().frame(height: 260) // перекрывает карту

                BIRGEGlassSheet {
                    VStack(alignment: .leading, spacing: BIRGELayout.s) {
                        // Header
                        HStack {
                            BIRGEMatchBadge(store.corridor.matchPercent)
                            Spacer()
                            seatsBadge
                        }

                        Text(store.corridor.name)
                            .font(BIRGEFonts.title)

                        // Departure time
                        Label(store.corridor.departure, systemImage: "clock.fill")
                            .font(BIRGEFonts.body)
                            .foregroundStyle(BIRGEColors.textSecondary)

                        Divider()

                        // Stats
                        HStack(spacing: BIRGELayout.xl) {
                            statItem(value: "\(store.corridor.price)₸", label: "Стоимость", symbol: "tengesign.circle.fill")
                            statItem(value: "\(store.corridor.seatsLeft)/\(store.corridor.seatsTotal)", label: "Мест", symbol: "person.2.fill")
                            statItem(value: "~35 мин", label: "В пути", symbol: "timer")
                        }

                        Divider()

                        // Passengers section
                        Text("Участники")
                            .font(BIRGEFonts.sectionTitle)

                        HStack(spacing: BIRGELayout.xs) {
                            ForEach(store.corridor.passengerInitials, id: \.self) { initial in
                                VStack(spacing: BIRGELayout.xxxs) {
                                    Text(initial)
                                        .font(BIRGEFonts.bodyMedium)
                                        .foregroundStyle(.white)
                                        .frame(width: 44, height: 44)
                                        .background(BIRGEColors.brandPrimary)
                                        .clipShape(Circle())
                                    Text("Пассажир")
                                        .font(BIRGEFonts.caption)
                                        .foregroundStyle(BIRGEColors.textSecondary)
                                }
                            }
                        }

                        // CTA Button
                        BIRGEPrimaryButton(
                            title: store.isJoining ? "Присоединяемся..." : "Присоединиться · \(store.corridor.price)₸",
                            isLoading: store.isJoining
                        ) {
                            send(.joinTapped)
                        }
                        .disabled(store.isJoining)
                        .padding(.top, BIRGELayout.xxs)
                    }
                    .padding(.horizontal, BIRGELayout.m)
                    .padding(.bottom, BIRGELayout.m)
                }
            }
        }
        .navigationBarTitleDisplayMode(.inline)
        .navigationTitle("Маршрут")
    }

    private func statItem(value: String, label: String, symbol: String) -> some View {
        VStack(spacing: BIRGELayout.xxxs) {
            Image(systemName: symbol)
                .font(BIRGEFonts.sectionTitle)
                .foregroundStyle(BIRGEColors.brandPrimary)
            Text(value)
                .font(BIRGEFonts.bodyMedium)
            Text(label)
                .font(BIRGEFonts.caption)
                .foregroundStyle(BIRGEColors.textSecondary)
        }
    }

    private var seatsBadge: some View {
        let hasSeats = store.corridor.seatsLeft > 0
        return Text(hasSeats ? "Места есть" : "Занято")
            .font(BIRGEFonts.captionBold)
            .foregroundStyle(hasSeats ? Color(hex: "065F46") : BIRGEColors.textSecondary)
            .padding(.horizontal, BIRGELayout.xs)
            .padding(.vertical, BIRGELayout.xxxs)
            .background(hasSeats ? Color(hex: "D1FAE5") : BIRGEColors.surfacePrimary)
            .clipShape(Capsule())
    }
}
```

---

## Задача C — Встроить в PassengerAppFeature

В `PassengerAppFeature.Path` удали `case corridor(CorridorFeature)` и добавь:
```swift
case corridorList(CorridorListFeature)
case corridorDetail(CorridorDetailFeature)
```

Обнови `.home(.delegate(.openCorridorList))`:
```swift
case .home(.delegate(.openCorridorList)):
    state.path.append(.corridorList(CorridorListFeature.State()))
    return .none
```

Добавь переход CorridorList → CorridorDetail:
```swift
case .path(.element(_, action: .corridorList(.delegate(.corridorSelected(let corridor))))):
    state.path.append(.corridorDetail(CorridorDetailFeature.State(corridor: corridor)))
    return .none
```

Обнови `PassengerAppView.swift` — добавь destinationView для новых case.

---

## Проверка

```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -30
```
