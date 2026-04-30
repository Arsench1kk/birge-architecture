---
last_updated: 2026-04-22
---

# GRDB Recipes — BIRGE Reference

> Готовые паттерны GRDB для копирования.

## Setup (DatabaseManager)

```swift
import GRDB

final class DatabaseManager {
    static let shared = DatabaseManager()
    let dbQueue: DatabaseQueue
    
    private init() {
        let path = try! FileManager.default
            .url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: true)
            .appendingPathComponent("birge.sqlite")
            .path
        
        var config = Configuration()
        config.journalMode = .wal  // ОБЯЗАТЕЛЬНО для BIRGE
        
        dbQueue = try! DatabaseQueue(path: path, configuration: config)
        try! runMigrations()
    }
    
    private func runMigrations() throws {
        var migrator = DatabaseMigrator()
        
        migrator.registerMigration("v1_location_records") { db in
            try db.create(table: "location_records") { t in
                t.autoIncrementedPrimaryKey("id")
                t.column("ride_id", .text).notNull()
                t.column("latitude", .double).notNull()
                t.column("longitude", .double).notNull()
                t.column("timestamp", .double).notNull()
                t.column("accuracy", .double)
                t.column("synced", .boolean).notNull().defaults(to: false)
            }
            try db.create(indexOn: "location_records", columns: ["ride_id", "synced"])
        }
        
        try migrator.migrate(dbQueue)
    }
}
```

## Model (Record)

```swift
struct LocationRecord: Codable, FetchableRecord, PersistableRecord {
    var id: Int64?
    let rideID: String
    let latitude: Double
    let longitude: Double
    let timestamp: Double
    let accuracy: Double?
    var synced: Bool = false
    
    static let databaseTableName = "location_records"
    
    enum Columns {
        static let rideID = Column("ride_id")
        static let synced = Column("synced")
        static let timestamp = Column("timestamp")
    }
}
```

## Repository

```swift
actor LocationRepository {
    private let dbQueue: DatabaseQueue
    
    init(dbQueue: DatabaseQueue = DatabaseManager.shared.dbQueue) {
        self.dbQueue = dbQueue
    }
    
    // Вставка одной записи
    func insert(_ record: LocationRecord) async throws {
        try await dbQueue.write { db in
            var r = record
            try r.insert(db)
        }
    }
    
    // Bulk вставка (100-200 записей)
    func insertBatch(_ records: [LocationRecord]) async throws {
        try await dbQueue.write { db in
            for var record in records {
                try record.insert(db)
            }
        }
    }
    
    // Получить несинхронизированные
    func fetchUnsynced(rideID: String) async throws -> [LocationRecord] {
        try await dbQueue.read { db in
            try LocationRecord
                .filter(LocationRecord.Columns.rideID == rideID)
                .filter(LocationRecord.Columns.synced == false)
                .order(LocationRecord.Columns.timestamp)
                .fetchAll(db)
        }
    }
    
    // Отметить как синхронизированные
    func markSynced(_ ids: [Int64]) async throws {
        try await dbQueue.write { db in
            try LocationRecord
                .filter(ids.contains(Column("id")))
                .updateAll(db, Column("synced").set(to: true))
        }
    }
}
```

## In-Memory DB для тестов

```swift
// В тестах всегда in-memory — быстро, изолировано
let testDB = try! DatabaseQueue()  // in-memory

// Или через DatabaseManager тест конфигурацию:
extension DatabaseManager {
    static var test: DatabaseManager {
        // Создать с in-memory queue
    }
}
```
