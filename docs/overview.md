
# Overview — HCNA‑3 (Hierarchical Composite Node Architecture)

**HCNA‑3** is a distributed computing architecture in which two autonomous physical computers — a **Moderator Node (M1)** and a **Worker Node (W1)** — federate hierarchically to instantiate a **third, composite system (C1)** that operates as a single logical computer.

HCNA‑3 separates responsibilities into three planes:
1. **Control Plane (M1‑centric):** Policy, scheduling, intent issuance, arbitration, and health regulation.
2. **Data Plane (W1‑centric):** Execution of labor‑intensive tasks, resource management, and result production.
3. **Composite Plane (C1):** The emergent virtual node exporting a unified interface (VIP/DNS/API) while M1 and W1 remain independently addressable.

Key properties: hierarchical federation, composite emergence, active–active cooperation, autonomy + cohesion, observability‑first, and extensibility.
