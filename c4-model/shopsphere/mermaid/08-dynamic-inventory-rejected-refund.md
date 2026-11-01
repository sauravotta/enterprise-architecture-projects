# Dynamic: Inventory rejected, refund (compensation)

Payment succeeded but stock is unavailable (e.g. `LAPTOP-1`). Payment Service listens to `inventory.rejected` and refunds.

```mermaid
C4Dynamic
    title Order saga: inventory rejected, payment refunded

    Container(order, "Order Service", "Spring Boot", "")
    ContainerQueue(kafka, "Apache Kafka", "KRaft", "")
    Container(payment, "Payment Service", "Spring Boot", "")
    Container(inventory, "Inventory Service", "Spring Boot", "")
    Container(notification, "Notification Service", "Spring Boot", "")
    System_Ext(pgw, "Payment Gateway", "")

    Rel(order, kafka, "1. Publishes orders.created", "Kafka")
    Rel(kafka, payment, "2. Delivers orders.created", "Kafka")
    Rel(payment, kafka, "3. Publishes payments.completed", "Kafka")
    Rel(kafka, inventory, "4. Delivers payments.completed", "Kafka")
    Rel(inventory, kafka, "5. Publishes inventory.rejected", "Kafka")
    Rel(kafka, payment, "6. Delivers inventory.rejected", "Kafka")
    Rel(payment, pgw, "7. Refunds payment", "HTTPS")
    Rel(kafka, order, "8. Delivers inventory.rejected: CANCELLED", "Kafka")
    Rel(kafka, notification, "9. Delivers outcome", "Kafka")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

```mermaid
sequenceDiagram
    autonumber
    participant O as Order Service
    participant K as Kafka
    participant P as Payment Service
    participant I as Inventory Service
    participant N as Notification Service
    O->>K: orders.created
    K->>P: orders.created
    P->>K: payments.completed
    K->>I: payments.completed
    I->>K: inventory.rejected
    par compensation and status
        K->>P: inventory.rejected (refund)
    and
        K->>O: inventory.rejected (CANCELLED)
    and
        K->>N: inventory.rejected (notify)
    end
```

## Failure handling (dead-letter)

```mermaid
flowchart LR
    T[Source topic] --> C[Consumer]
    C -- success --> OK[Commit offset]
    C -- exception --> R{Retry up to 3x}
    R -- recovered --> OK
    R -- exhausted --> D[topic.DLT]
    D --> OPS[SRE inspects in Control Center]
    B[Bad payload] --> E[ErrorHandlingDeserializer] --> D
```
