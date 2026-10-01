# Day 1 — Scenario Analysis

## PayScale Financial Technologies: HTTPE Redesign

PayScale Financial Technologies is preparing for a major increase in transaction
traffic during its Diwali festive-season campaign. The current platform is designed
for approximately 1,200 transactions per second (TPS), while the target system must
sustain 12,000+ TPS and tolerate bursts up to 18,000 TPS. At the same time, the
architecture must maintain low latency, high availability, transaction correctness,
and regulatory compliance.

The current architecture has several bottlenecks that prevent it from scaling
linearly. The PostgreSQL primary experiences connection-pool exhaustion at around
1,800 TPS, with query queues exceeding 500 requests. This creates a critical
database bottleneck. The single-node RabbitMQ deployment also develops message
acknowledgement backlog, resulting in consumer lag of more than 30 seconds at
2,000 TPS. At the application layer, thread-pool saturation causes approximately
40% request timeouts under heavy load. Redis performance also degrades as its cache
hit ratio falls from 92% to 61%, while memory evictions increase. Finally,
row-level contention on the accounts table produces a reported 15% deadlock rate,
making concurrent balance updates a major correctness and performance concern.

The redesign therefore needs to remove individual scaling limits and distribute
work across multiple components. The database should move from a single primary
toward a distributed PostgreSQL-based architecture with sharding, replication,
partitioning and connection pooling. Messaging should move to a multi-broker
Apache Kafka cluster so transaction events can be processed in parallel using
partitions and consumer groups. The application layer should use horizontally
scalable instances behind a load balancer and API gateway rather than depending
on a fixed thread pool.

The cache layer should be redesigned as a Redis Cluster with appropriate key
distribution, memory limits and cache invalidation rules. For account-balance
updates, optimistic concurrency control should be used so concurrent transactions
can detect version conflicts instead of creating incorrect balances. The system
also requires idempotency, Saga orchestration, fault tolerance and compensating
actions to protect financial transactions during retries or component failures.

The redesign must also consider the operational constraints of the scenario.
The infrastructure budget has a ceiling of $45,000 per month, existing team
expertise is concentrated in Java/Kotlin and PostgreSQL, and the project specifies
that data must remain within India. Therefore, technology choices must balance
throughput, latency, reliability, operational complexity, cost and team familiarity.

The objective is not simply to increase capacity. The final architecture must
provide predictable performance, strong transaction correctness, fault isolation
and measurable recovery behavior while scaling from the existing 1,200 TPS
environment toward the 12,000+ TPS target.

## Top Five Bottlenecks

| Bottleneck | Current Problem | Preliminary Solution |
|---|---|---|
| PostgreSQL Primary | Connection exhaustion at ~1,800 TPS; query queue >500 | Distributed/sharded database, replicas, PgBouncer |
| RabbitMQ | Consumer acknowledgement backlog; >30s lag at 2,000 TPS | Multi-broker Apache Kafka with partitions and consumer groups |
| Application Layer | Thread-pool saturation; ~40% request timeout | Horizontal scaling behind load balancer + bounded concurrency |
| Redis | Hit ratio falls 92% → 61%; memory evictions | Redis Cluster + memory policies + cache invalidation strategy |
| Account DB Locks | ~15% deadlock rate during concurrent balance updates | OCC/version checking + controlled retry strategy |