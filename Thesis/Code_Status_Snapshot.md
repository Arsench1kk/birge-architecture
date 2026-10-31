---
created: 2026-10-18
status: черновик
---

# Срез кода для диплома

## Состояние на 2026-10-31

- Passenger и Driver собраны для arm64 Simulator на Xcode 27.2 с TCA 1.26.1 и Sharing 2.8.2; запуск и тесты клиента не подтверждены. [проверено запуском 2026-10-31] [[Status_2026-10-31]]
- Backend: 12 тестов, 0 failures; matching-lab: 31 тест, 0 failures. [проверено запуском 2026-10-31] [[Status_2026-10-31]]
- Отметки об ошибке iOS-сборки 18 октября ниже являются историческими и заменены более поздним результатом. [проверено запуском 2026-10-31]

Таблица ниже сохраняет выводы прежнего статического аудита; его отметка о не запущенных проверках относилась к моменту того аудита. Текущие результаты сборки и тестов добавлены ниже. Состояние реализации из статического аудита помечено [из аудита кода] [не проверено запуском], если отдельная проверка ниже его не подтвердила.

| Область | Состояние по аудиту | Дипломный разрыв |
|---|---|---|
| Backend | Модели `RecurringRoute`, `CommutePlan` и CRUD найдены | Нет подтверждённого генератора расписания/рейсов |
| Matching | Оценка аудита ~0–5%; ручное принятие запросов и debug endpoint | Нет пакетного constrained matching |
| Эксперименты | Генератор спроса, симулятор, метрики и baseline не найдены | Нужно создать стенд |
| iOS | UI преимущественно подключён к mock clients | Нужен тонкий реальный путь к API либо демо через симулятор |
| Проверки (на момент прежнего аудита) | Найдены 171 тест; запуск тогда не состоялся из-за Xcode license prompt | Текущие результаты см. ниже |
| Инфраструктура | Compose без PostGIS; актуальный запуск сервисов не подтверждён | Простые расстояния позволяют не блокироваться на geo stack |

Проблемы из переданного аудита: OTP пишется в лог; проверка владельца route IDs в commute plan отсутствует; отмена возможна в `inProgress`; Kaspi signature bypass если секрет не задан; сосуществуют разные тарифные модели; в compose нет PostGIS. [из аудита кода] [не проверено запуском]

Документация обещает Redis GEO, OSRM, PostGIS/H3 и архитектурный matching, но аудит не подтвердил matching recurring trips или работающий эксперимент. [из заметок: Architecture/Geo_Matching_and_AI.md] [из аудита кода] [не проверено запуском]

Оценки трудоёмкости ниже не включать как факт; заполнить после плана руководителя. Примерная оценка в днях допустима только как гипотеза с диапазоном после декомпозиции. [рекомендация консультанта]


## Результаты проверки сборки и тестов (2026-10-18)

| Проверка | Статус | Команда / результат |
|---|---|---|
| Xcode license / toolchain | РАБОТАЕТ | `xcodebuild -version` → Xcode 27.1, Build 27A9275; команда завершилась успешно, лицензионный prompt отсутствовал. [проверено запуском 2026-10-18] |
| Backend tests | РАБОТАЕТ | `swift test --scratch-path /private/tmp/birge-build/backend` из `BIRGEPassenger/birge-vapor` → 11 тестов, 0 failures; exit 0. Компиляция выдала warnings в `configure.swift` и `AuthService.swift`, тесты завершились успешно. [проверено запуском 2026-10-18] |
| Passenger build | НЕ СОБИРАЕТСЯ | `xcodebuild build -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'generic/platform=iOS Simulator' -derivedDataPath /private/tmp/birge-build/ios-passenger -clonedSourcePackagesDirPath /private/tmp/birge-build/ios-source-packages -quiet` → exit 65. Первая ошибка в зависимости `swift-composable-architecture` 1.25.5: `NavigationStack+Observation.swift:149:17: cannot form key path to main actor-isolated subscript`. [проверено запуском 2026-10-18] |
| Driver build | НЕ СОБИРАЕТСЯ | `xcodebuild build -project BIRGEPassenger.xcodeproj -scheme BIRGEDrive -destination 'generic/platform=iOS Simulator' -derivedDataPath /private/tmp/birge-build/ios-driver -clonedSourcePackagesDirPath /private/tmp/birge-build/ios-source-packages -quiet` → exit 65, та же первая ошибка в `swift-composable-architecture/NavigationStack+Observation.swift:149:17`. [проверено запуском 2026-10-18] |
| Passenger tests | НЕ ПРОВЕРЕНО (нет доступного iPhone Simulator) | Попытка `xcodebuild test -project BIRGEPassenger.xcodeproj -scheme BIRGEPassenger -destination 'platform=iOS Simulator,name=iPhone 17 Pro' -derivedDataPath /private/tmp/birge-build/ios-passenger -clonedSourcePackagesDirPath /private/tmp/birge-build/ios-source-packages -quiet` → exit 70: no available device matched destination. CoreSimulator 1051.55.0 старее требуемой версии 1174.9.2. Тесты не запускались. [проверено запуском 2026-10-18] |
| Driver tests | НЕ ПРОВЕРЕНО (в проекте не найден test target) | `xcodebuild -list -project BIRGEPassenger.xcodeproj` перечисляет `BIRGEPassengerTests`, но не `BIRGEDriveTests`; схемы `BIRGEPassenger`, `BIRGEDrive`, `BIRGECore`. Отдельный driver test target не найден. [проверено запуском 2026-10-18] |
| Compose configuration | РАБОТАЕТ | `docker compose config -q` из `BIRGEPassenger/birge-vapor` → exit 0. [проверено запуском 2026-10-18] |
| PostgreSQL / Redis services | НЕ ПРОВЕРЕНО (Docker daemon недоступен) | `docker compose ps` завершилась ошибкой: не удалось подключиться к `unix:///Users/arsenabduhalyk/.docker/run/docker.sock`; сервисы не запускались. [проверено запуском 2026-10-18] |

Passenger `build-for-testing` также завершился ошибкой до запуска тестов при линковке package product `Sharingdynamic-product`; runtime-тестирование не проводилось. [проверено запуском 2026-10-18]
Артефакты и DerivedData направлялись в `/private/tmp/birge-build`; исходный код проекта не изменялся. [проверено запуском 2026-10-18]

Вывод на 18 октября (заменён более поздним результатом): backend собирается и проходит тесты; iOS блокируется зависимостью TCA 1.25.5 при Xcode 27.1 и несовместимым CoreSimulator; вероятная причина — окружение/версии, исправление не выполнялось. [проверено запуском 2026-10-18]
