# Level 2: Containers

Split in two for readability. Elements marked **(Planned)** are target-state and not in the current Kafka demo.

## 2a. Core containers (current demo plus entry points)

```mermaid
C4Container
    title Container diagram for ShopSphere: core

    System_Boundary(shop, "ShopSphere") {
        Container(spa, "Web Storefront", "React SPA", "Customer shopping UI")
        Container(gw, "API Gateway (Planned)", "Spring Cloud Gateway", "Routing, JWT validation, rate limiting")
        Container(order, "Order Service", "Java 17, Spring Boot :8080", "Accepts orders (202 PENDING), tracks status from events")
        Container(payment, "Payment Service", "Java 17, Spring Boot", "Charges customers; refunds on inventory.rejected")
        Container(inventory, "Inventory Service", "Java 17, Spring Boot", "Reserves stock")
        Container(notification, "Notification Service", "Java 17, Spring Boot", "Notifies customers of final outcomes")
        ContainerQueue(kafka, "Apache Kafka", "Confluent 7.7, KRaft", "orders.created, payments.*, inventory.*, *.DLT")
        Container(cc, "Control Center", "Confluent, :9021", "Topics, messages, consumer lag")
    }

    Person(customer, "Customer", "Shops online")
    Person(sre, "SRE", "Operates the platform")
    System_Ext(pay, "Payment Gateway", "Charges and refunds")
    System_Ext(notify, "Email / SMS Provider", "Message delivery")

    Rel(customer, spa, "Uses", "HTTPS")
    Rel(spa, gw, "API calls", "HTTPS/JSON")
    Rel(gw, order, "POST /orders, GET /orders/{id}", "HTTPS/JSON")
    Rel(order, kafka, "Publishes orders.created; consumes outcomes", "Kafka/JSON")
    Rel(kafka, payment, "orders.created, inventory.rejected", "Kafka/JSON")
    Rel(payment, kafka, "payments.completed / failed", "Kafka/JSON")
    Rel(kafka, inventory, "payments.completed", "Kafka/JSON")
    Rel(inventory, kafka, "inventory.reserved / rejected", "Kafka/JSON")
    Rel(kafka, notification, "Final outcome events", "Kafka/JSON")
    Rel(payment, pay, "Charges and refunds", "HTTPS/JSON")
    Rel(notification, notify, "Sends messages", "HTTPS/JSON")
    Rel(sre, cc, "Inspects topics and lag", "HTTPS")
    Rel(cc, kafka, "Monitors", "Kafka protocol")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

The shared `common` Maven module (event records, topic names, Kafka config with topic creation, 3x retry, dead-letter topics, `ErrorHandlingDeserializer`) is compiled into the four services and omitted here.

## 2b. Planned expansion (target state)

```mermaid
C4Container
    title Container diagram for ShopSphere: planned expansion

    System_Ext(carrier, "Shipping Carrier API", "Shipments")
    System_Ext(idp, "Identity Provider", "OIDC")
    Person(staff, "Merchant / Support", "Back-office user")

    System_Boundary(shop, "ShopSphere") {
        Container(bo, "Back-office Portal", "React SPA", "Merchant and support tooling")
        Container(gw, "API Gateway", "Spring Cloud Gateway", "Routing, JWT validation")
        Container(order, "Order Service", "Spring Boot", "Existing")
        Container(payment, "Payment Service", "Spring Boot", "Existing")
        Container(inventory, "Inventory Service", "Spring Boot", "Existing")
        Container(catalog, "Catalog Service", "Spring Boot", "Product catalogue and search")
        Container(shipping, "Shipping Service", "Spring Boot", "Ships orders once stock is reserved")
        ContainerQueue(kafka, "Apache Kafka", "KRaft", "Existing")
        Container(registry, "Schema Registry", "Confluent", "Event schema governance")
        Container(obs, "Observability", "Prometheus, Grafana, OTel", "Metrics, traces, dashboards")
        ContainerDb(orderdb, "Orders DB", "PostgreSQL", "Order state")
        ContainerDb(paydb, "Payments DB", "PostgreSQL", "Payment records")
        ContainerDb(invdb, "Inventory DB", "PostgreSQL", "Stock levels")
        ContainerDb(search, "Search Index", "Elasticsearch", "Catalogue search")
    }

    Rel(staff, bo, "Uses", "HTTPS")
    Rel(bo, gw, "API calls", "HTTPS/JSON")
    Rel(gw, idp, "Validates tokens", "OIDC/JWKS")
    Rel(gw, catalog, "Product queries", "HTTPS/JSON")
    Rel(order, orderdb, "Reads/writes", "JDBC")
    Rel(payment, paydb, "Reads/writes", "JDBC")
    Rel(inventory, invdb, "Reads/writes", "JDBC")
    Rel(catalog, search, "Indexes, queries", "HTTPS")
    Rel(kafka, shipping, "inventory.reserved", "Kafka/JSON")
    Rel(shipping, carrier, "Creates shipments", "HTTPS/JSON")
    Rel(kafka, catalog, "Inventory changes", "Kafka/JSON")
    Rel(kafka, registry, "Validates schemas", "HTTPS")
    Rel(order, obs, "Emits telemetry", "OTLP")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```
