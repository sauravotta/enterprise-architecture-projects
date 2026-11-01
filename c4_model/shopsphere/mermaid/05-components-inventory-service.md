# Level 3: Components, Inventory Service

```mermaid
C4Component
    title Component diagram for Inventory Service

    ContainerQueue(kafka, "Apache Kafka", "KRaft", "Event backbone")
    ContainerDb(db, "Inventory DB (Planned)", "PostgreSQL", "Stock levels")

    Container_Boundary(inv, "Inventory Service") {
        Component(lis, "Inventory Listener", "@KafkaListener", "Consumes payments.completed; emits inventory.reserved / inventory.rejected")
        Component(mgr, "Stock Reservation Manager", "Spring Bean", "Checks and decrements stock atomically (BOOK-1=10, PHONE-1=2, LAPTOP-1=0 in the demo)")
        Component(repo, "Stock Repository", "In-memory map; Spring Data JPA (Planned)", "Stock levels")
    }

    Rel(kafka, lis, "payments.completed", "Kafka/JSON")
    Rel(lis, mgr, "Reserve stock")
    Rel(mgr, repo, "Reads/writes")
    Rel(lis, kafka, "inventory.reserved / inventory.rejected", "Kafka/JSON")
    Rel(repo, db, "Reads/writes", "JDBC")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Notification Service (summary)

Single `@KafkaListener` component consuming final outcome events (`inventory.reserved`, `payments.failed`, `inventory.rejected`) and logging a simulated email. Planned: template renderer and a provider adapter calling the Email/SMS provider.
