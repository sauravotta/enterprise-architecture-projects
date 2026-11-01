# ShopSphere C4 Model (Mermaid)

C4 model ([c4model.com](https://c4model.com/)) of **ShopSphere**, an e-commerce platform that extends the Kafka order-saga demo (`eventdriven_microservices`: order, payment, inventory, notification services on Kafka). All diagrams are Mermaid and render natively on GitHub and in VS Code (Markdown preview with a Mermaid extension).

Elements labelled **(Planned)** are target-state and not part of the current demo.

| Level | Diagram | File |
|---|---|---|
| Landscape | System landscape | [11-system-landscape.md](mermaid/11-system-landscape.md) |
| 1 Context | System context | [01-system-context.md](mermaid/01-system-context.md) |
| 2 Container | Containers | [02-containers.md](mermaid/02-containers.md) |
| 3 Component | Order Service | [03-components-order-service.md](mermaid/03-components-order-service.md) |
| 3 Component | Payment Service | [04-components-payment-service.md](mermaid/04-components-payment-service.md) |
| 3 Component | Inventory (and Notification) | [05-components-inventory-service.md](mermaid/05-components-inventory-service.md) |
| Dynamic | Saga success | [06-dynamic-saga-success.md](mermaid/06-dynamic-saga-success.md) |
| Dynamic | Payment failure | [07-dynamic-payment-failure.md](mermaid/07-dynamic-payment-failure.md) |
| Dynamic | Inventory rejected, refund, DLT | [08-dynamic-inventory-rejected-refund.md](mermaid/08-dynamic-inventory-rejected-refund.md) |
| Deployment | Production | [09-deployment-production.md](mermaid/09-deployment-production.md) |
| Deployment | Local docker compose | [10-deployment-local-docker-compose.md](mermaid/10-deployment-local-docker-compose.md) |

Supporting docs: [event catalogue](docs/event-catalogue.md), [ADRs](docs/adr.md).

## Notes
- Mermaid's C4 support is experimental: layout is automatic, so tune `$c4ShapeInRow` in `UpdateLayoutConfig` if a diagram looks crowded.
- Level 4 (code) is intentionally omitted, as recommended by C4 for most systems.
- The dynamic views also include a sequence diagram, which shows ordering more clearly than C4Dynamic.
