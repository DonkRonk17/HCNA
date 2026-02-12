# HCNA — Frequently Asked Questions

**Last Updated**: 2026-02-11

---

## General Questions

### What is HCNA?

**HCNA (Hierarchical Composite Node Architecture)** is a distributed computing architecture where specialized physical computers federate hierarchically to instantiate emergent composite systems. Think of it as the **hardware embodiment of multi-agent AI systems** — one node per specialized role, working together to create something greater than the sum of its parts.

### What does the number mean (HCNA-3, HCNA-4, etc.)?

The number represents the **total component count**, not a version number:
- **HCNA-3**: 3 components (M1 + W1 + C1) — the first implementation
- **HCNA-4**: 4 components (e.g., M1 + W1 + W2 + C1)
- **HCNA-5**: 5 components (e.g., M1 + W1 + W2 + S1 + C1)
- **HCNA-N**: Arbitrary component counts as workload demands

### How is HCNA different from traditional distributed systems?

Traditional distributed systems treat nodes as **peers** or **replicas**. HCNA treats nodes as **specialized roles** that compose hierarchically:
- **M1 doesn't "manage" W1** like a manager manages workers
- **M1 and W1 collaborate to BE C1** — true emergent composition
- **C1 isn't load balancing** — it's the unified system that M1+W1 instantiate

### Is HCNA theoretical or real?

**Real.** A physical 2-laptop HCNA-3 prototype is currently in development by Randell Logan Smith (Metaphy LLC), demonstrating the architecture with working QEGG and DRGFC workloads.

---

## Technical Questions

### What is the "2S1C" principle?

**2S1C** = **Two Systems, One Computer, Three Entities**

- **System 1 (M1)**: Physical Moderator computer
- **System 2 (W1)**: Physical Worker computer
- **Computer 1 (C1)**: Virtual Composite system (emergent entity)

Two physical systems compose into one logical computer, yielding three total entities.

### What are the three planes?

1. **Control Plane (M1)**: Policy, scheduling, orchestration, health monitoring
2. **Data Plane (W1)**: Workload execution, resource management, result generation
3. **Composite Plane (C1)**: Unified external API, coherent system identity

### What is an "intent message"?

An **intent** is a structured JSON envelope that M1 sends to W1 to request work execution:

```json
{
  "msg_id": "ulid-01HZX...",
  "ts": "2026-02-11T18:00:00Z",
  "src": "HCNA-M1--prod--usw1--v1.0",
  "dst": "HCNA-W1--prod--usw1--v1.0",
  "intent": "execute.task",
  "schema": "hcna.intent.v1",
  "payload": {"task_id": "job-123", "task_type": "qegg.generate"}
}
```

Intents are M1's way of asking W1 to do something (generate QEGG model, compress file, etc.).

### What programming languages does HCNA use?

HCNA is **language-agnostic**. The architecture defines:
- Communication protocols (JSON intents, REST APIs, WebSockets)
- Node roles and responsibilities
- Security requirements (mTLS, encryption)

**Implementation can use any language**: Python (reference implementation), Go, Rust, Java, etc.

### Can I run HCNA on virtual machines or containers?

Yes! HCNA works on:
- **Physical hardware**: Laptops, desktops, servers (like Logan's 2-laptop prototype)
- **Virtual machines**: VMware, VirtualBox, KVM, Hyper-V
- **Containers**: Docker, Kubernetes, Podman
- **Cloud instances**: AWS EC2, Google Compute Engine, Azure VMs

The key is **network connectivity** between M1 and W1 nodes.

---

## Architecture Questions

### What roles can nodes have?

Current defined roles (more can be added):

- **M (Moderator)**: Orchestration, scheduling, policy enforcement
- **W (Worker)**: Task execution, compute-intensive workloads
- **C (Composite)**: Unified external interface (emergent entity)
- **S (Storage)**: Persistent data layer, replicated volumes
- **A (Accelerator)**: GPU/NPU/TPU offload for specialized compute
- **G (Gateway)**: Ingress/egress traffic control, mTLS termination
- **O (Observer)**: Metrics, logging, tracing, observability
- **X (Auditor)**: Compliance monitoring, security auditing
- **R (Orchestrator)**: Multi-cluster coordinator (HMSS-level)

### Can I have multiple Workers (W1, W2, W3...)?

**Yes!** That's HCNA-4+. Example:
- **HCNA-4**: M1 + W1 + W2 + C1
  - M1 distributes tasks across W1 and W2 in parallel
  - Both workers report telemetry back to M1
  - C1 aggregates results from both workers

### What happens if M1 fails?

**Single M1 (HCNA-3)**: System loses orchestration, tasks fail (no redundancy yet)

**Multi-Moderator (future HCNA-N with M1, M2, M3)**:
- Use Raft/etcd consensus for leader election
- Standby moderators take over on M1 failure
- Requires distributed state management

### What happens if W1 fails?

**Single W1 (HCNA-3)**: Tasks queued on M1 wait until W1 recovers

**Multi-Worker (HCNA-4+)**: M1 reschedules failed tasks to W2, W3, etc.

### How does C1 "exist" if it's virtual?

**C1 is implemented as**:
- **VIP (Virtual IP)**: DNS name (`hcna-c1.local`) pointing to M1 or load balancer
- **API Gateway**: Reverse proxy (NGINX, HAProxy, Traefik) routing client requests
- **Unified Interface**: Single API endpoint clients connect to

Clients don't know M1/W1 exist — they only interact with C1.

---

## Deployment Questions

### What hardware do I need to deploy HCNA-3?

**Minimum (prototype)**:
- **M1**: 2-core CPU, 2GB RAM, 20GB disk (orchestration is lightweight)
- **W1**: 4-core CPU, 8GB RAM, 50GB disk (execution needs horsepower)
- **Network**: LAN connectivity between M1 and W1 (Ethernet or WiFi)

**Recommended (production)**:
- **M1**: 4-core CPU, 8GB RAM, 100GB SSD
- **W1**: 8+ core CPU, 16GB+ RAM, 500GB+ SSD (for QEGG/DRGFC workloads)
- **Network**: 1 Gbps Ethernet, low latency (<5ms)

### Can I deploy HCNA in the cloud?

**Yes!** Example AWS deployment:
- **M1**: t3.medium EC2 instance (2 vCPU, 4GB RAM) in us-west-2a
- **W1**: c5.2xlarge EC2 instance (8 vCPU, 16GB RAM) in us-west-2a
- **C1**: Application Load Balancer (ALB) with VIP
- **Network**: VPC with private subnet, security groups allowing M1↔W1 traffic

### How long does deployment take?

**First-time deployment** (following [`DEPLOYMENT_GUIDE.md`](DEPLOYMENT_GUIDE.md)):
- **Phase 1-4** (setup): 2-3 hours
- **Phase 5-6** (testing): 1 hour
- **Total**: ~4 hours for prototype

**Subsequent deployments** (automated with Ansible/Terraform): <30 minutes

### Do I need Kubernetes?

**No, HCNA is simpler**:
- **Kubernetes**: Orchestrates many containers across many nodes (complex, overkill for HCNA-3)
- **HCNA**: Orchestrates specialized nodes with clear roles (M1, W1, C1)

**However**, you CAN run HCNA nodes inside Kubernetes pods if desired. HCNA is orthogonal to Kubernetes.

---

## Integration Questions

### What is QEGG?

**QEGG (Quantum Entangled Geometric Grid)** is a Metaphy LLC proprietary technology that generates dodecahedral geometric models using Blender-based hierarchical recursion.

**Integration with HCNA**: M1 assigns QEGG generation tasks to W1, which runs Blender to create models.

**Status**: ✅ Working (v4.0.3 models built and tested)

### What is DRGFC?

**DRGFC (Dodecahedral Recursive Geometric Fractal Compression)** is a novel compression algorithm developed by Metaphy LLC using fractal techniques.

**Integration with HCNA**: M1 assigns compression tasks to W1, which compresses/decompresses files.

**Status**: ✅ Working (v16.1 Python, v1.3 3D variant, Rust port operational)

**Performance**: Competitive with ZSTD, faster decompression due to fractal approach.

### What is BPCS?

**BPCS (Base-Prime Cryptographic System)** is a prime-based encryption system developed by Metaphy LLC.

**Integration with HCNA**: Secures all M1↔W1 control plane communication with message-level encryption (in addition to mTLS transport security).

**Status**: ✅ Production (deployed in BCH)

### Can I use QEGG/DRGFC/BPCS without licensing?

**No**. QEGG, DRGFC/DRGFC-3D, BPCS, and other Metaphy LLC technologies are **proprietary**. This repository documents the **HCNA architecture** (open source, MIT), but **integration code** requires separate commercial licensing.

**Contact**: logan@metaphysicsandcomputing.com

### What is BCH?

**BCH (Beacon Command Hub)** is a production AI communication system developed by Metaphy LLC that demonstrates the **C1 composite pattern** with AI agents.

**Architecture parallels**:
- **C1 = BCH WebSocket API** (unified interface)
- **M1 = BCH backend** (orchestrates message routing)
- **W1 = AI agents** (CLIO, FORGE, IRIS, etc. execute tasks)

BCH proves the HCNA concept works in production with virtual systems (AI agents), paving the way for physical systems (hardware nodes).

---

## Licensing & Commercial Questions

### Is HCNA open source?

**Yes, the architecture is MIT licensed**. You can:
- Use HCNA architecture in your own projects
- Build your own M1/W1/C1 implementations
- Contribute documentation improvements
- Create educational materials

**Proprietary components** (QEGG, DRGFC, BPCS) require separate licensing.

### Can I use HCNA commercially?

**Yes!** MIT license allows commercial use of the architecture. However:
- **Open source HCNA**: Free for any use (commercial or non-commercial)
- **Proprietary integrations** (QEGG, DRGFC, BPCS): Require commercial licensing from Metaphy LLC

### How do I get a commercial license for QEGG/DRGFC/BPCS?

**Contact**: logan@metaphysicsandcomputing.com

**Licensing options**:
- **Research License**: Academic/non-commercial use
- **Commercial License**: Production deployments
- **OEM License**: Embed in products
- **Enterprise License**: Unlimited deployment within organization

### Can I contribute to HCNA?

**Yes!** Contributions to **open source documentation** are welcome:
- Documentation improvements
- Example configurations
- Deployment guides
- Bug reports and feature requests

See [`CONTRIBUTING.md`](../CONTRIBUTING.md) for guidelines.

**Integration code contributions** (proprietary QEGG/DRGFC/BPCS) require Metaphy LLC Contributor License Agreement (CLA).

---

## Future Vision Questions

### What is HMSS?

**HMSS (Hierarchical Multi-System Supervisor)** is **HCNA at scale** — orchestrating 10s or 100s of HCNA-N units into federated networks.

**Example**: 50 HCNA-3 units (100 physical computers) → 50 C1 composite nodes → 1 HMSS orchestrator (R1 role) → global API

**Status**: 📋 Planned, expect prototype 2029-2030

### What is QUAD?

**QUAD (Quantum Universal Adaptive Dynamics)** is the **100-year vision** — planetary-scale networks of HMSS clusters running QEGG→DRGFC→LWIS workloads for Type III civilization computing.

**Scale**: 1,000,000+ HCNA units (2 million computers) in global network by 2125

**Status**: 🎯 Vision, 100-year roadmap (2025→2125)

### What is LWIS?

**LWIS (Light-Wave Information Systems)** will replace TCP/IP networking between M1 and W1 with **photonic (laser-based) communication** for sub-100 microsecond latency.

**Status**: 🔬 Research phase, expect prototype 2027-2028

### What is SPTS?

**SPTS (Simplified Ternary Coding System)** encodes control plane messages using **0/1/2 ternary logic** instead of binary, reducing bandwidth by ~5x.

**Status**: 🔬 Theoretical, expect prototype 2028-2029

---

## Community & Support

### Where can I get help?

- **GitHub Issues**: Report bugs, ask questions
- **GitHub Discussions**: Community forum (coming soon)
- **Email**: logan@metaphysicsandcomputing.com
- **Website**: [MetaphysicsandComputing.com](https://metaphysicsandcomputing.com)

### How can I stay updated?

- **Star the GitHub repo**: Watch for releases and updates
- **Twitter/X**: Follow [@metaphyllc](https://twitter.com/metaphyllc) (if available)
- **Newsletter**: Sign up at MetaphysicsandComputing.com

### Who created HCNA?

**Randell Logan Smith**, Founder of [Metaphy LLC](https://metaphysicsandcomputing.com)

**Collaborators**:
- **Microsoft Copilot**: Naming conventions, initial documentation
- **Claude (CLIO)**: Enhanced documentation, integration notes, HTML design

---

## Troubleshooting

### M1 and W1 can't communicate

**Check**:
1. Network connectivity: `ping hcna-w1.local` from M1
2. Firewall rules: M1 port 8443, W1 port 9443 open
3. TLS certificates: Verify CN matches hostname
4. Logs: `journalctl -u hcna-m1 -n 50` and `journalctl -u hcna-w1 -n 50`

### Tasks stuck in "queued" status

**Check**:
1. W1 service running: `systemctl status hcna-w1`
2. M1 scheduler active: `journalctl -u hcna-m1 | grep scheduler`
3. Resource limits: Is W1 out of CPU/RAM?

### High latency (>1 second per task)

**Check**:
1. Network latency: `ping hcna-w1.local` should be <5ms on LAN
2. W1 CPU usage: `htop` — is it maxed out?
3. M1 queue size: If queue is large, add more workers (HCNA-4)

See full troubleshooting guide in [`DEPLOYMENT_GUIDE.md`](DEPLOYMENT_GUIDE.md#troubleshooting).

---

**Document Version**: 1.0.0
**Last Updated**: 2026-02-11
**Maintained By**: CLIO (Team Brain) on behalf of Randell Logan Smith / Metaphy LLC
