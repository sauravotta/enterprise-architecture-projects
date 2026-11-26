# Software Architecture Principles

Architecture is the set of important decisions that shape a system. Good decisions start with business needs and balance quality, delivery speed, operational effort, and cost. No design is best for every workload: make trade-offs explicit and revisit them as requirements change.

## Tie principles to business value

TOGAF recommends documenting each architecture principle with a **statement**, its **rationale** (why it matters to the enterprise), and its **implications** (what it requires and may cost). Each principle below makes those parts explicit. Assign a business owner and an architecture steward to the principle set, and tailor targets to the workload.

| Principle | Business value | Possible measure |
|---|---|---|
| Requirements | Invest in outcomes customers and the business need. | Goal attainment, customer satisfaction |
| Modularity and change | Deliver improvements sooner; reduce change risk. | Lead time for change, change failure rate |
| Reliability | Protect revenue, trust, and essential services. | Availability, recovery time, failed transactions |
| Performance | Reduce user friction and support more completed work. | Response time, throughput, abandonment |
| Scalability | Handle growth without paying for permanent peak capacity. | Cost per transaction, capacity headroom |
| Data ownership and consistency | Improve decision quality and avoid conflicting records. | Data quality, reconciliation effort |
| Security and privacy | Reduce breach, fraud, legal, and reputational risk. | Security incidents, remediation time |
| Deployment and operations | Release value safely and reduce service disruption. | Deployment frequency, incident duration |
| Cost and sustainability | Improve return on technology spend and resource use. | Cost per business outcome, resource utilization |
| Decision and exception governance | Make trade-offs accountable and prevent unmanaged risk. | Decision lead time, overdue exceptions |

Choose measures that reflect the workload and business priorities; these examples are not universal targets. For each major decision, record the baseline, target, evidence source, accountable owner, and review point. Agree in advance what result triggers corrective action.

## 1. Start with requirements

**Statement:** Base architecture decisions on agreed business outcomes and measurable quality requirements.

**Implications:**
- Identify users, key workflows, data sensitivity, expected traffic, and external dependencies.
- Turn quality needs into measurable targets: response time, availability, recovery time, and budget.
- Prioritize requirements by business impact. Higher availability or faster response can increase cost and complexity.

**Rationale / business value:** Focuses investment on outcomes that matter instead of unneeded technical sophistication.

*Example:* Specify that checkout should respond within 300 ms at expected peak load and recover within 30 minutes after a regional outage.

## 2. Modularity and change

**Statement:** Keep responsibilities cohesive and expose stable, well-defined interfaces.

**Implications:**
- Split responsibilities into cohesive modules with clear interfaces; avoid dependencies on another component's internal details.
- Keep the design as simple as requirements allow. More services add network calls, deployment work, and failure modes.
- Version APIs and plan data changes so components can evolve without requiring a coordinated release.

**Rationale / business value:** Makes features easier to deliver and lowers the cost and risk of future changes.

*Example:* An order module calls a documented inventory API instead of reading inventory tables directly.

## 3. Reliability and recovery

**Statement:** Design services to detect failures, limit their impact, and recover within business-defined targets.

**Implications:**
- Assume machines, networks, deployments, and external services can fail, sometimes independently.
- Avoid single points of failure with appropriate redundancy, health checks, and tested backups.
- Use timeouts, bounded retries with backoff, and circuit breakers; retries without limits can worsen overload.
- Make retried operations idempotent, using an idempotency key or equivalent where needed, to prevent duplicate orders or charges.
- Define recovery point and recovery time targets, then test failover and restore procedures.

**Rationale / business value:** Limits lost sales and customer impact when components or dependencies fail.

*Example:* If a payment provider is unavailable, stop retrying after a short limit and leave the order in a retryable state.

## 4. Performance

**Statement:** Meet performance targets for representative workloads without wasting resources.

**Implications:**
- Measure latency, throughput, and resource saturation under representative data and workload.
- Find the actual bottleneck before optimizing; growing queues often indicate slow processing, contention, or limited capacity.
- Reduce unnecessary network and disk access; tune queries and algorithms before adding hardware.
- Cache frequently read, slowly changing data when stale results are acceptable. Set expiry and invalidation rules.
- Improve concurrency carefully; shared mutable state and coarse locks can limit throughput or cause deadlocks.

**Rationale / business value:** Improves customer experience and productivity while helping avoid unnecessary infrastructure spend.

*Example:* Cache product descriptions, but read current inventory from its source before confirming checkout.

## 5. Scalability

**Statement:** Distribute work and capacity so the system can grow or shrink with demand.

**Implications:**
- Scale vertically (larger machines) for simplicity; scale horizontally (more instances) when demand or availability requires it.
- Prefer stateless request handlers so any healthy instance can serve a request.
- Use load balancing, asynchronous queues, and partitioning to distribute work and avoid one constrained component.
- Choose partition keys that spread demand; monitor for uneven load and hot partitions.
- Replication can improve read capacity and availability, but adds consistency and synchronization trade-offs.

**Rationale / business value:** Supports growth and demand spikes without requiring a costly redesign or permanent peak-sized systems.

*Example:* Add API instances behind a load balancer and partition order records by customer or region.

## 6. Data ownership and consistency

**Statement:** Give data clear ownership and choose consistency rules to match each business process.

**Implications:**
- Assign an authoritative owner for each important data set; other components access it through agreed interfaces or events.
- Decide where immediate consistency is essential and where delayed updates are acceptable; communicate the user and business impact of stale data.
- Define transaction boundaries, data-quality rules, and compatible schema-change procedures.
- Reconcile replicated or exchanged data and make ownership and retention responsibilities clear.

**Rationale / business value:** Trusted, timely data supports better decisions and avoids costly disputes between systems.

*Example:* An order service owns order status; an analytics feed can update asynchronously, but checkout checks authoritative inventory.

## 7. Security and privacy

**Statement:** Protect identities, systems, and data according to business risk and applicable obligations.

**Implications:**
- Protect confidentiality, integrity, and availability throughout design, development, deployment, and operation.
- Authenticate identities and authorize each action; apply least privilege and use managed identity where practical.
- Encrypt sensitive data in transit and at rest; manage keys and secrets outside application code.
- Validate untrusted input and use parameterized queries to prevent injection. Protect state-changing browser requests from CSRF with anti-forgery tokens, SameSite cookies, or origin checks.
- Threat-model trust boundaries and manage third-party dependencies, including security updates.
- Minimize sensitive data collection and retention. Audit important actions without logging credentials or tokens.

**Rationale / business value:** Protects customer trust and reduces the financial, legal, and operational impact of security incidents.

*Example:* Require TLS, use parameterized database queries, and give the reporting service read-only access.

## 8. Deployment and operations

**Statement:** Make systems repeatable to deploy, observable in operation, and recoverable from change or failure.

**Implications:**
- Automate builds, infrastructure provisioning, configuration, tests, and deployment so environments are repeatable.
- Make changes observable with useful logs, metrics, traces, health checks, and alerts.
- Deploy gradually and keep a rollback path. Select rolling, canary, or blue-green releases based on risk and capacity.
- Practice incident response and data recovery; automation should be safe to repeat and should report failures clearly.
- Prefer managed services when their reliability and reduced maintenance justify their cost and constraints.

**Rationale / business value:** Gets useful changes to customers sooner while reducing avoidable outages and support effort.

*Example:* Release to 5% of traffic, compare errors and latency with the existing version, then continue or roll back.

## 9. Cost and sustainability

**Statement:** Use resources responsibly while meeting required business outcomes and service levels.

**Implications:**
- Estimate the full lifecycle cost, including compute, storage, network traffic, licensing, support, and operations.
- Match capacity to actual demand; use autoscaling where useful and remove idle resources.
- Review data retention, redundant resources, and workload efficiency against business and environmental goals.
- Avoid optimizing cost in ways that undermine required security, reliability, or performance.

**Rationale / business value:** Directs budget to higher-value work and reduces waste and environmental impact.

*Example:* Scale a batch worker to zero when its queue is empty, while retaining enough capacity to meet processing deadlines.

## 10. Decision and exception governance

**Statement:** Resolve conflicts according to business priorities, and make significant decisions and exceptions accountable.

**Implications:**
- Record important decisions, alternatives, business impact, risks, and the accountable decision owner in an architecture decision record.
- Name who can approve a deviation from a principle. Document its rationale, risk acceptance, compensating controls, and expiry or review trigger.
- Reassess exceptions when requirements, risks, or systems change; escalate unresolved conflicts to the designated business and architecture decision-makers.

**Rationale / business value:** Makes trade-offs transparent, speeds future decisions, and prevents unreviewed exceptions from accumulating.

*Example:* Accept a temporary single-region deployment to meet a launch date, with an approved risk owner and a dated plan to add regional recovery.

## Architecture review checklist

- Are key requirements and measurable service targets documented?
- Are important failure modes, security boundaries, and data flows understood?
- Can components be tested, deployed, monitored, and changed independently?
- Have performance and recovery been tested under realistic conditions?
- Is each principle's business owner, target, evidence source, and review point clear?
- Are decisions and exceptions recorded with an accountable approver and review date?
- Are the costs and trade-offs acceptable to the business?

## References

- [Microsoft Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/)
- [Azure application design principles](https://learn.microsoft.com/en-us/azure/architecture/guide/design-principles/)
- [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [Google Cloud Well-Architected Framework](https://docs.cloud.google.com/architecture/framework)
- [The Open Group TOGAF Standard](https://www.opengroup.org/togaf) — architecture principles are framed with a statement, rationale, and implications.
