# Codex Prompt 01 — Liquid Glass Design System

## Контекст
Проект: `/Users/arsenabduhalyk/Projects/BIRGE/BIRGEApp/BIRGEPassenger`
Целевой файл: `BIRGECore/Sources/BIRGECore/DesignSystem/BIRGEModifiers.swift`

Мокапы используют glassmorphism (backdrop-filter: blur + полупрозрачный фон + тонкая белая рамка).
Codex должен добавить нативные Liquid Glass компоненты в дизайн-систему.

---

## Задача

Отредактируй файл `BIRGECore/Sources/BIRGECore/DesignSystem/BIRGEModifiers.swift`.
Добавь новые компоненты **в конец файла**, не удаляй существующие.

### 1. BIRGEGlassModifier — базовый стеклянный modifier

```swift
/// Универсальный Liquid Glass modifier.
/// На iOS 26+ использует нативный .glassEffect().
/// На iOS 15–25 — .ultraThinMaterial с border.
public struct BIRGEGlassModifier: ViewModifier {
    public enum Variant {
        case card       // карточки, листы — radius 20
        case sheet      // нижний лист — radius 28 top only
        case pill       // pills/chips — radius 100
        case button     // кнопки — radius 16
    }

    let variant: Variant
    let tint: Color

    public init(variant: Variant = .card, tint: Color = .clear) {
        self.variant = variant
        self.tint = tint
    }

    public func body(content: Content) -> some View {
        if #available(iOS 26, *) {
            content
                .glassEffect(
                    .regular
                        .tinted(tint == .clear ? BIRGEColors.brandPrimary.opacity(0.05) : tint)
                )
                .clipShape(clipShape)
        } else {
            content
                .background(
                    clipShape
                        .fill(.ultraThinMaterial)
                        .overlay(
                            clipShape
                                .stroke(Color.white.opacity(0.18), lineWidth: 1)
                        )
                )
        }
    }

    @ViewBuilder
    private var clipShape: some InsettableShape {
        switch variant {
        case .card:   RoundedRectangle(cornerRadius: BIRGELayout.radiusL)
        case .sheet:  UnevenRoundedRectangle(
                          topLeadingRadius: BIRGELayout.radiusL,
                          topTrailingRadius: BIRGELayout.radiusL
                      )
        case .pill:   Capsule()
        case .button: RoundedRectangle(cornerRadius: BIRGELayout.radiusM)
        }
    }
}

public extension View {
    /// Применить Liquid Glass стиль.
    func liquidGlass(
        _ variant: BIRGEGlassModifier.Variant = .card,
        tint: Color = .clear
    ) -> some View {
        modifier(BIRGEGlassModifier(variant: variant, tint: tint))
    }
}
```

### 2. BIRGEGlassSheet — нижний лист

```swift
/// Стандартный нижний стеклянный лист BIRGE.
/// Заменяет паттерн `.background(BIRGEColors.background.clipShape(...))`.
public struct BIRGEGlassSheet<Content: View>: View {
    let content: Content

    public init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    public var body: some View {
        VStack(spacing: 0) {
            BIRGESheetHandle()
                .padding(.top, BIRGELayout.xxs)
            content
        }
        .liquidGlass(.sheet)
        .shadow(color: BIRGEColors.brandPrimary.opacity(0.08), radius: 24, y: -8)
    }
}
```

### 3. BIRGEGlassCard — карточка

```swift
/// Стеклянная карточка — замена `.birgeCard()`.
public struct BIRGEGlassCardModifier: ViewModifier {
    public func body(content: Content) -> some View {
        content
            .liquidGlass(.card)
            .shadow(
                color: BIRGEColors.brandPrimary.opacity(0.06),
                radius: 16,
                x: 0, y: 8
            )
    }
}

public extension View {
    func birgeGlassCard() -> some View {
        modifier(BIRGEGlassCardModifier())
    }
}
```

### 4. BIRGEAIPill — AI notification pill

```swift
/// Pill с AI-иконкой и текстом. Аналог .ai-pill из мокапа.
public struct BIRGEAIPill: View {
    let text: String

    public init(_ text: String) {
        self.text = text
    }

    public var body: some View {
        HStack(spacing: BIRGELayout.xxxs) {
            Image(systemName: "sparkles")
                .font(BIRGEFonts.captionBold)
                .foregroundStyle(BIRGEColors.brandPrimary)
            Text(text)
                .font(BIRGEFonts.captionBold)
                .foregroundStyle(BIRGEColors.brandPrimary)
        }
        .padding(.horizontal, BIRGELayout.s)
        .padding(.vertical, BIRGELayout.xxs)
        .liquidGlass(.pill, tint: BIRGEColors.brandPrimary.opacity(0.06))
        .shadow(color: BIRGEColors.brandPrimary.opacity(0.18), radius: 8, y: 4)
    }
}
```

### 5. BIRGEMatchBadge — AI процент совпадения

```swift
/// Бейдж совпадения AI (98%, 87% и т.д.)
public struct BIRGEMatchBadge: View {
    let percent: Int

    public init(_ percent: Int) {
        self.percent = percent
    }

    public var body: some View {
        HStack(spacing: BIRGELayout.xxxs) {
            Image(systemName: "sparkles")
                .font(.system(size: 10, weight: .semibold))
            Text("\(percent)% совпадение")
                .font(.system(size: 12, weight: .semibold))
        }
        .foregroundStyle(BIRGEColors.brandPrimary)
        .padding(.horizontal, BIRGELayout.xs)
        .padding(.vertical, BIRGELayout.xxxs)
        .background(BIRGEColors.brandPrimary.opacity(0.1))
        .clipShape(Capsule())
        .overlay(Capsule().stroke(BIRGEColors.brandPrimary.opacity(0.22), lineWidth: 1))
    }
}
```

---

## Дополнительно — обнови BIRGELayout

В файле `BIRGECore/Sources/BIRGECore/DesignSystem/BIRGELayout.swift` добавь:

```swift
// MARK: - Map overlay
public static let mapSearchBarTop: CGFloat = 59   // отступ от top safe area
public static let mapSearchBarHeight: CGFloat = 52
```

---

## Проверка после изменений

Убедись что проект компилируется:
```
xcodebuild -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build 2>&1 | tail -20
```

Ожидаемый результат: `BUILD SUCCEEDED`
