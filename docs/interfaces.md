
# Interfaces

## Message Envelope (JSON)
```json
{
  "msg_id": "ulid-01HZX3Z9T2A3...",
  "ts": "2026-02-11T17:05:00Z",
  "src": "HCNA-M1--prod--usw1--v1.0",
  "dst": "HCNA-W1--prod--usw1--v1.0",
  "intent": "execute.task",
  "schema": "hcna.intent.v1",
  "payload": {
    "task_id": "job-12345",
    "priority": "high",
    "resources": {"cpu": 4, "mem_gb": 8},
    "artifact_uri": "hcna://bundle/abc123"
  },
  "trace": {"trace_id": "00-...-...-01"}
}
```

## Endpoints
- **C1 API:** `https://hcna-c1.prod.local`
- **M1:** `https://hcna-m1.prod.local:8443`
- **W1:** `https://hcna-w1.prod.local:9443`
