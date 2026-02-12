
# Architecture

## Principles
- Hierarchical control, federated execution, composite emergence.
- Loose coupling with strong contracts.
- Fail‑soft operation and graceful degradation.
- Observability‑first design.
- Reproducible deployments.

## System Context (ASCII)
```
+-------------------------- External Clients --------------------------+
            |                 |                     |
            v                 v                     v
                    [ C1: Composite Node API ]
                               |
              +----------------+----------------+
              |                                 |
   [M1: Moderator Node] <---- Control Bus ----> [W1: Worker Node]
              |                                 |
       Telemetry/Policy                   Execution/Resources
```

## Operational Modes
- **Nominal:** M1 governs; W1 executes; C1 serves clients via VIP/DNS.
- **Degraded:** W1 continues with cached policies; C1 may expose reduced service.
- **Recovery:** M1 reconciles and resumes full orchestration.

## Non‑Functional Requirements
- Availability ≥ 99.5% (single site).
- P95 dispatch latency < 50 ms on local network.
- mTLS between nodes; signed intents from M1; least privilege on C1 interfaces.
- Unified logs, metrics, traces with W3C TraceContext.

## Risks & Mitigations
- **M1 bottleneck:** Async queues, backpressure, limited W1 autonomy.
- **Split‑brain C1:** Leaderless reads, leased writes, quorum flags.
- **Clock skew:** Logical/hybrid clocks; monotonic IDs.
