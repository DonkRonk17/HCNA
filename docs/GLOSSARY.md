# HCNA — Terminology Glossary

**Comprehensive reference for HCNA architecture terms, acronyms, and concepts.**

**Last Updated**: 2026-02-11

---

## Core Architecture Terms

### HCNA
**Hierarchical Composite Node Architecture** — The overall distributed computing architecture where specialized physical computers federate hierarchically to instantiate emergent composite systems. The physical embodiment of multi-agent systems.

### HCNA-N
**HCNA with N Components** — Specific implementations of the HCNA architecture. The number N represents total component count:
- **HCNA-3**: 3 components (M1 + W1 + C1)
- **HCNA-4**: 4 components (e.g., M1 + W1 + W2 + C1)
- **HCNA-5**: 5 components (e.g., M1 + W1 + W2 + S1 + C1)
- And so on...

### 2S1C
**Two Systems, One Computer, Three Entities** — The core philosophy of HCNA-3:
- **2 Systems**: M1 (Moderator) and W1 (Worker) physical computers
- **1 Computer**: C1 (Composite) virtual unified system
- **3 Entities**: Total component count in HCNA-3

### Compositional Computing
Computing paradigm where specialized nodes **compose hierarchically** to instantiate emergent systems, rather than coordinating as peers or replicas. The whole becomes genuinely greater than the sum of its parts.

---

## Node Roles

### M1 (Moderator Node)
**Control Plane Supervisor** — Physical computer #1 that orchestrates tasks, enforces policies, schedules workloads, and monitors health. The strategic brain that decides *what* to do and *when*.

**Color Code**: Blue (#1565C0)
**Analogy**: Like CLIO in Team Brain (orchestrator, coordinator)

### W1 (Worker Node)
**Data Plane Executor** — Physical computer #2 that executes compute-intensive workloads, processes data, generates results, and reports telemetry. The muscle that does the *actual work*.

**Color Code**: Green (#2E7D32)
**Analogy**: Like FORGE in Team Brain (tool builder, task executor)

### C1 (Composite Node)
**Unified External Interface** — Virtual emergent entity that clients interact with. C1 **is** the system M1 and W1 collectively instantiate. Not load balancing — true emergence.

**Color Code**: Purple (#6A1B9A)
**Analogy**: Like BCH in Team Brain (unified API for all agents)

### S1 (Storage Node)
**Persistent Data Layer** — Node providing replicated volumes, distributed storage, and data persistence across the HCNA system.

**Role Letter**: S
**Used In**: HCNA-4+ (e.g., M1 + W1 + S1 + C1)

### A1 (Accelerator Node)
**Specialized Compute** — Node with GPU/NPU/TPU hardware for offloading compute-heavy tasks (AI inference, QEGG rendering, video encoding).

**Role Letter**: A
**Used In**: HCNA-5+ (e.g., M1 + W1 + S1 + A1 + C1)

### G1 (Gateway Node)
**Ingress/Egress Control** — Node handling traffic control, mTLS termination, rate limiting, and API gateway functions.

**Role Letter**: G
**Used In**: HCNA-6+

### O1 (Observer Node)
**Observability** — Node dedicated to metrics collection, log aggregation, tracing, and dashboard visualization (Prometheus, Grafana, Loki).

**Role Letter**: O
**Used In**: HCNA-6+

### X1 (Auditor Node)
**Compliance & Security Monitoring** — Node for audit logging, compliance validation, security event monitoring, and forensics.

**Role Letter**: X
**Used In**: HCNA-7+

### R1 (Orchestrator Node)
**Multi-Cluster Coordinator** — HMSS-level orchestrator that coordinates multiple HCNA-N units into federated networks.

**Role Letter**: R
**Used In**: HMSS (Hierarchical Multi-System Supervisor)

---

## The Three Planes

### Control Plane
**M1's Domain** — Handles policy enforcement, task scheduling, arbitration, health monitoring, and intent propagation. The "management layer" where decisions are made.

**Key Flows**: M1 → W1 (intents), W1 → M1 (telemetry)

### Data Plane
**W1's Domain** — Handles workload execution, resource management, result generation, and data processing. The "execution layer" where work gets done.

**Key Flows**: W1 → C1 (results), Client → W1 (input data)

### Composite Plane
**C1's Domain** — Presents unified external API, maintains coherent system identity, routes client requests, and aggregates results. The "interface layer" clients interact with.

**Key Flows**: Client → C1 (requests), C1 → Client (responses)

---

## Communication Protocols

### Intent Message
Structured JSON envelope M1 sends to W1 requesting work execution. Contains:
- **msg_id**: Unique message identifier (ULID format)
- **ts**: Timestamp (ISO 8601)
- **src**: Source node ID (e.g., `HCNA-M1--prod--usw1--v1.0`)
- **dst**: Destination node ID (e.g., `HCNA-W1--prod--usw1--v1.0`)
- **intent**: Intent type (e.g., `execute.task`, `qegg.generate`, `drgfc.compress`)
- **schema**: Message schema version (e.g., `hcna.intent.v1`)
- **payload**: Intent-specific data (task details, parameters)

### Telemetry Report
Structured JSON message W1 sends to M1 reporting task status, health, and resource usage. Contains:
- **msg_id**, **ts**, **src**, **dst**: Same as intent messages
- **intent**: Always `telemetry.report`
- **schema**: `hcna.telemetry.v1`
- **payload**: Task status, duration, resource usage, errors

### Intent Protocol
The overall communication framework for M1↔W1 messages using JSON envelopes, ULID message IDs, timestamp tracking, and schema versioning.

### mTLS (Mutual TLS)
**Transport-level security** — Both M1 and W1 authenticate each other using X.509 certificates. Ensures only authorized nodes can communicate.

### BPCS Encryption
**Message-level security** — Intent and telemetry payloads encrypted with Base-Prime Cryptographic System (Metaphy LLC proprietary). Provides defense-in-depth: even if mTLS is compromised, messages remain encrypted.

---

## Metaphy LLC Technologies

### QEGG
**Quantum Entangled Geometric Grid** — Proprietary Metaphy LLC technology that generates dodecahedral geometric models using Blender-based hierarchical recursion.

**Integration**: M1 orchestrates QEGG tasks, W1 executes Blender scripts
**Status**: ✅ Working (v4.0.3)
**Use Case**: Generate perfect dodecahedral models for visualization, 3D printing, simulation

### DRGFC
**Dodecahedral Recursive Geometric Fractal Compression** — Proprietary Metaphy LLC compression algorithm using novel fractal techniques.

**Integration**: M1 assigns compression tasks, W1 runs DRGFC algorithms
**Status**: ✅ Working (v16.1 Python, Rust port operational)
**Performance**: Competitive compression ratios, faster decompression
**Use Case**: Compress QEGG models, arbitrary data files

### DRGFC-3D
**3D Variant of DRGFC** — Hierarchical compression specifically optimized for 3D geometric data.

**Status**: ✅ Working (v1.3)
**Use Case**: Compress 3D models, mesh data, point clouds

### BPCS
**Base-Prime Cryptographic System** — Proprietary Metaphy LLC encryption using prime-based mathematics.

**Integration**: Secures M1↔W1 control plane communication
**Status**: ✅ Production (deployed in BCH)
**Use Case**: Encrypt intent messages, telemetry reports

### BCH
**Beacon Command Hub** — Production AI communication system demonstrating the C1 composite pattern with AI agents as "nodes."

**Status**: ✅ Production (desktop/mobile apps operational)
**Use Case**: Reference implementation of HCNA principles with virtual systems

### LWIS
**Light-Wave Information Systems** — Future photonic communication protocol replacing TCP/IP with laser-based links for sub-100 microsecond latency.

**Status**: 🔬 Research phase
**Target**: 2027-2028 prototype
**Use Case**: Ultra-low-latency M1↔W1 communication for real-time control loops

### SPTS
**Simplified Ternary Coding System** — Future ternary (0/1/2) encoding for control messages, reducing bandwidth by ~5x vs. JSON.

**Status**: 🔬 Theoretical
**Target**: 2028-2029 prototype
**Use Case**: Compact intent message representation

### HMSS
**Hierarchical Multi-System Supervisor** — HCNA at scale, orchestrating 10s or 100s of HCNA-N units into federated networks.

**Role**: R1 (Orchestrator) sits above multiple C1 composite nodes
**Status**: 📋 Planned
**Target**: 2029-2030 prototype

### QUAD
**Quantum Universal Adaptive Dynamics** — 100-year vision for planetary-scale networks of HMSS clusters running QEGG→DRGFC→LWIS workloads.

**Scale**: 1,000,000+ HCNA units (2M computers) globally by 2125
**Status**: 🎯 Vision
**Use Case**: Type III civilization computing infrastructure

---

## Naming Conventions

### Node ID Format
```
{ARCH}-{ROLE}{INDEX}[--{ENV}][--{REG}][--v{MAJOR}.{MINOR}]
```

**Examples**:
- `HCNA-M1--prod--usw1--v1.0` — Production Moderator, US-West-1, version 1.0
- `HCNA-W1--lab--usw1--v1.0` — Lab Worker, US-West-1, version 1.0
- `HCNA-C1--prod--global--v1.0` — Production Composite, global VIP, version 1.0

### Environment Codes
- **prod**: Production
- **stage**: Staging/pre-production
- **lab**: Laboratory/testing
- **dev**: Development

### Region Codes
- **usw1**: US-West-1
- **use1**: US-East-1
- **euc1**: Europe-Central-1
- **aps1**: Asia-Pacific-South-1
- **global**: Multi-region VIP

---

## Deployment Terms

### VIP (Virtual IP)
IP address or DNS name (e.g., `hcna-c1.local`, `192.168.1.100`) that clients connect to for the C1 Composite API. VIP can point to:
- M1 directly (simplest, for prototypes)
- Load balancer (production, for HA)
- API gateway (with rate limiting, mTLS termination)

### Health Check
Periodic ping/probe from M1 to W1 (and vice versa) to verify node is alive and responsive. Typical interval: 10-30 seconds.

**Failure threshold**: 3 consecutive failed health checks → node marked unhealthy

### Task Queue
M1's internal queue of pending tasks waiting for W1 availability. Tasks are:
- **Queued**: Waiting for scheduling
- **Dispatched**: Sent to W1 for execution
- **Completed**: Results returned from W1
- **Failed**: Error occurred, may be retried

### Observability Stack
Tools for monitoring HCNA system health:
- **Prometheus**: Metrics collection (CPU, RAM, task counts, latency)
- **Grafana**: Dashboard visualization
- **Loki**: Log aggregation (centralized logs from M1, W1, C1)
- **OpenTelemetry**: Distributed tracing (track intents across nodes)

---

## AI/Multi-Agent Analogies

### Team Brain
Metaphy LLC's multi-agent AI system where specialized AI agents (CLIO, FORGE, IRIS, ATLAS, etc.) collaborate on complex tasks. **HCNA is the hardware version of Team Brain.**

| Team Brain Concept | HCNA Equivalent |
|--------------------|-----------------|
| **CLIO** (Git/Trophy Master) | **M1** (Moderator) — Orchestrates |
| **FORGE** (Tool Builder) | **W1** (Worker) — Executes specialized work |
| **BCH** (Unified API) | **C1** (Composite) — Single client interface |
| **SynapseLink** (Agent communication) | **Intent Protocol** (Node communication) |
| **Agent Roles** (specialization) | **Node Roles** (M, W, S, A, G, O...) |

### Agent Specialization
In multi-agent systems, each agent has a specialized role (CLIO = Git, FORGE = Tools, IRIS = Desktop). **HCNA mirrors this at the hardware level**: M1 = orchestration, W1 = execution, S1 = storage, etc.

---

## License & Commercial Terms

### MIT License
Open source license for HCNA architecture documentation. Allows commercial use, modification, and distribution. See [`LICENSE`](../LICENSE).

### Proprietary Technology
Metaphy LLC technologies (QEGG, DRGFC, BPCS, LWIS, SPTS) that require separate commercial licensing. Not included in MIT-licensed HCNA repo.

### Commercial Licensing
Paid license required to use proprietary Metaphy LLC technologies in production. Options:
- **Research License**: Academic/non-commercial
- **Commercial License**: Production deployments
- **OEM License**: Embed in products
- **Enterprise License**: Unlimited deployment

**Contact**: logan@metaphysicsandcomputing.com

### CLA (Contributor License Agreement)
Legal agreement required to contribute integration code for proprietary technologies (QEGG, DRGFC, BPCS).

---

## Common Acronyms

### API
**Application Programming Interface** — C1 exposes REST API for clients to submit tasks and retrieve results.

### DNS
**Domain Name System** — Maps hostnames like `hcna-c1.local` to IP addresses like `192.168.1.100`.

### HA
**High Availability** — System design ensuring uptime even with node failures. Requires multi-moderator (M1, M2, M3) and multi-worker (W1, W2, W3) deployments.

### JSON
**JavaScript Object Notation** — Data format for intent messages, telemetry reports, and API requests/responses.

### ULID
**Universally Unique Lexicographically Sortable Identifier** — 128-bit ID format for message IDs. Like UUID but sortable by timestamp.

**Example**: `01HZX3Z9T2A3B5C8D2E4F7G9H1`

### mTLS
**Mutual Transport Layer Security** — Both client and server authenticate each other using X.509 certificates. Used for M1↔W1 communication.

### REST
**Representational State Transfer** — Architectural style for C1 API. Uses HTTP methods (GET, POST, PUT, DELETE) and JSON payloads.

### WebSocket
Protocol for real-time bidirectional communication. Used in BCH (WebSocket API for AI agents). Could be used for M1↔W1 communication as alternative to REST.

---

## Metaphy LLC

### Metaphy LLC
Company founded by Randell Logan Smith developing HCNA, QEGG, DRGFC, BPCS, and the broader QUAD vision.

**Website**: [MetaphysicsandComputing.com](https://metaphysicsandcomputing.com)
**Email**: logan@metaphysicsandcomputing.com
**Mission**: Build Type III civilization computing infrastructure

### Randell Logan Smith
Founder and architect of Metaphy LLC, creator of HCNA architecture and all associated technologies (QEGG, DRGFC, BPCS, HMSS, QUAD).

**Background**: Healthcare AI, privacy & ethics focus
**Vision**: "If I dream it, it's probable that we can build it!"

### Type III Civilization
Kardashev scale concept: civilization capable of harnessing energy and resources at planetary scale. **QUAD's vision** is to provide computing infrastructure for a Type III civilization.

---

## Document Conventions

### ✅ Status Icons
- **✅ Working**: Technology built and operational
- **🔬 Research**: Active research phase
- **📋 Planned**: Design phase, not yet prototyped
- **🎯 Vision**: Long-term goal (10-100 year horizon)

### Color Codes
- **M1 (Moderator)**: Blue (#1565C0) — Thoughtful, strategic
- **W1 (Worker)**: Green (#2E7D32) — Productive, resourceful
- **C1 (Composite)**: Purple (#6A1B9A) — Emergent, unified

### Font Conventions
- **`Code`**: Filenames, commands, code snippets
- **[Links]()**: References to other documentation
- ***Italic***: Emphasis, new terms on first use
- **Bold**: Strong emphasis, key concepts

---

**Document Version**: 1.0.0
**Last Updated**: 2026-02-11
**Maintained By**: CLIO (Team Brain) on behalf of Randell Logan Smith / Metaphy LLC
