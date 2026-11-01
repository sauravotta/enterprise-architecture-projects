# Architecture Decision Records

## ADR-1: Choreography saga instead of orchestration
- **Status:** Accepted
- **Context:** Order placement spans payment and inventory, each with its own data.
- **Decision:** Services react to events; no central orchestrator.
- **Consequences:** Low coupling, easy to add consumers (notification, shipping). End-to-end flow is implicit, so it is documented in the dynamic views.

## ADR-2: Kafka as the event backbone
- **Status:** Accepted
- **Decision:** Confluent Kafka (KRaft), JSON events keyed by `orderId`, `acks=all`, idempotent producers, retry x3 then DLT.
- **Consequences:** Per-order ordering and replay. Operational cost of a cluster; Schema Registry planned for contract governance.

## ADR-3: Compensation through events
- **Status:** Accepted
- **Decision:** `inventory.rejected` triggers a refund in Payment Service.
- **Consequences:** Eventual consistency; refunds must be idempotent.

## ADR-4: Database per service (planned)
- **Status:** Proposed
- **Decision:** Replace in-memory state with a PostgreSQL schema per service.
- **Consequences:** Independent scaling and ownership; no cross-service joins.

## ADR-5: Transactional outbox (planned)
- **Status:** Proposed
- **Decision:** Write the order and the `orders.created` event in one DB transaction, relayed to Kafka.
- **Consequences:** Removes dual-write risk once Orders DB exists.
