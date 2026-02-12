
# Naming & Style Guide

## Canonical Names
- **HCNA‑3** — Hierarchical Composite Node Architecture (3‑System Model)
- **2S1C** — Two Systems, One Computer (motif)

## Roles & Codes
- `M` — Moderator (control plane)
- `W` — Worker (data plane)
- `C` — Composite (client‑facing logical node)
- `S` — Storage (future)
- `A` — Accelerator (future)
- `G` — Gateway (future)
- `O` — Observer (future)
- `X` — Auditor (future)
- `R` — Orchestrator (future)

## Node Naming Pattern
```
{ARCH}-{ROLE}{INDEX}[--{ENV}][--{REG}][--v{MAJOR}.{MINOR}]
```
Examples:
```
HCNA-M1--prod--usw1--v1.0
HCNA-W1--lab--usw1--v1.0
HCNA-C1--prod--global--v1.0
```

## Network & Endpoints
- C1: `https://hcna-c1.prod.local`
- M1: `https://hcna-m1.prod.local:8443`
- W1: `https://hcna-w1.prod.local:9443`

## Documentation Style
- First use: full term + acronym.
- Headings: Title Case for docs, sentence case for sections.
- Colors: M `#1565C0`, W `#2E7D32`, C `#6A1B9A`, Control `#0277BD`, Data `#388E3C`.
