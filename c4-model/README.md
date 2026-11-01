# C4 Models

A collection of [C4 model](https://c4model.com/) architecture diagrams (Mermaid). Each system lives in its own sub-folder with its own README, `mermaid/` diagrams and `docs/`.

| System | Description | Folder |
|---|---|---|
| ShopSphere | Event-driven e-commerce platform extending the Kafka order-saga demo | [shopsphere/](shopsphere/README.md) |

## Adding a new C4 model

1. Create `c4-model/<system-name>/` with `README.md`, `mermaid/` and `docs/`.
2. Number the diagrams by level (context, containers, components, dynamic, deployment).
3. Add a row to the table above.
