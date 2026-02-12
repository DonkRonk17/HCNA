# HCNA-3 Deployment Guide

**Guide Purpose**: Step-by-step instructions for deploying your first HCNA-3 (2S1C) system using two physical computers.

**Audience**: System administrators, DevOps engineers, and enthusiasts building prototype HCNA-3 systems.

**Scope**: This guide covers the **open source architecture** deployment. Integration with proprietary Metaphy LLC technologies (QEGG, DRGFC, BPCS) requires separate licensing and will be covered in commercial documentation.

---

## Overview

By the end of this guide, you will have:
- ✅ Two computers configured as **M1 (Moderator)** and **W1 (Worker)**
- ✅ Control plane communication (M1 → W1 intents, W1 → M1 telemetry)
- ✅ Composite node **C1** exposing unified API to clients
- ✅ Basic observability (logs, metrics, health checks)
- ✅ Sample workload (echo/ping tasks) proving the 2S1C concept

**Estimated Time**: 4-6 hours for first deployment

**Prerequisites**:
- 2 computers (laptops, desktops, or VMs) on same local network
- Linux (Ubuntu 22.04+ recommended) or compatible OS
- Network connectivity between computers
- Basic familiarity with Linux CLI, Python, and networking

---

## Architecture Recap

```
Client (Your Phone/Laptop)
   ↓
   └→ C1 (Composite Node API) ← VIP/DNS: hcna-c1.local
        ↑
        ├─ Control Plane ─→ M1 (Moderator) ← Physical Computer #1
        │                     ↓
        │                     └─ Intent → W1 (Worker) ← Physical Computer #2
        │                                    ↓
        └───── Data Plane ──────────────────┘
                (Results flow back)
```

**Key Concepts**:
- **M1**: Schedules tasks, enforces policies, monitors health
- **W1**: Executes tasks, reports telemetry, delivers results
- **C1**: Virtual entity (VIP or load balancer) clients connect to
- **Control Plane**: M1↔W1 communication (intents, telemetry)
- **Data Plane**: W1→C1 results delivery

---

## Phase 1: Hardware Preparation

### 1.1 Select Your Computers

**Moderator (M1) Requirements**:
- **CPU**: 2+ cores (lightweight orchestration)
- **RAM**: 2GB minimum, 4GB recommended
- **Disk**: 20GB minimum
- **Network**: Ethernet or WiFi (low latency preferred)
- **Role**: Orchestration brain, not compute-heavy

**Worker (W1) Requirements**:
- **CPU**: 4+ cores (compute-intensive workloads)
- **RAM**: 8GB minimum, 16GB+ recommended for QEGG/DRGFC
- **Disk**: 50GB+ (stores input files, outputs, intermediate data)
- **Network**: Ethernet or WiFi (high bandwidth for result uploads)
- **Role**: Execution engine, needs horsepower

**Example Setup**:
- **M1**: Old laptop (2015 MacBook Air, 2-core i5, 4GB RAM) running Ubuntu 22.04
- **W1**: Gaming laptop (2022 Legion, 8-core i7, 16GB RAM, NVIDIA GPU) running Ubuntu 22.04

### 1.2 Operating System Installation

**Recommended OS**: Ubuntu 22.04 LTS (Server or Desktop)

**Installation Steps** (both M1 and W1):
1. Download Ubuntu 22.04 ISO from ubuntu.com
2. Create bootable USB with Rufus (Windows) or `dd` (Linux/Mac)
3. Boot from USB, follow installer
4. **Hostname Convention**:
   - M1: `hcna-m1-prod-home-v1`
   - W1: `hcna-w1-prod-home-v1`
5. Create user: `hcna-admin` (consistent across both nodes)
6. Enable SSH during installation (or install after: `sudo apt install openssh-server`)

### 1.3 Network Configuration

**Static IP Assignment** (recommended for stability):

**On M1**:
```bash
# Edit netplan config
sudo nano /etc/netplan/01-netcfg.yaml

# Example config:
network:
  version: 2
  ethernets:
    eth0:  # Or your interface name (check with: ip a)
      dhcp4: no
      addresses:
        - 192.168.1.100/24  # M1 static IP
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# Apply config
sudo netplan apply
```

**On W1**:
```bash
# Same process, different IP:
addresses:
  - 192.168.1.101/24  # W1 static IP
```

**DNS Configuration** (optional but recommended):

Add to `/etc/hosts` on **both** M1 and W1:
```
192.168.1.100  hcna-m1.local  hcna-m1
192.168.1.101  hcna-w1.local  hcna-w1
192.168.1.100  hcna-c1.local  hcna-c1  # C1 VIP points to M1 initially
```

**Test Connectivity**:
```bash
# From M1, ping W1:
ping hcna-w1.local

# From W1, ping M1:
ping hcna-m1.local

# Should see: 64 bytes from... time=<1ms (if on same LAN)
```

---

## Phase 2: Software Stack Installation

### 2.1 Base Dependencies (Both M1 and W1)

```bash
# Update package list
sudo apt update && sudo apt upgrade -y

# Install base tools
sudo apt install -y \
  python3 python3-pip python3-venv \
  git curl wget nano vim \
  build-essential \
  net-tools iptables \
  htop tmux

# Verify Python version (need 3.9+)
python3 --version
```

### 2.2 Install Docker (Optional, for Containerized Deployment)

If you prefer containerized M1/W1 services:

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group (logout/login required after)
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose-plugin

# Test Docker
docker --version
docker compose version
```

### 2.3 HCNA-3 Core Software (M1 and W1)

**Clone this repository** (on both M1 and W1):

```bash
cd ~
git clone https://github.com/metaphyllc/hcna-3.git
cd hcna-3

# Create Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies (when available)
pip install -r requirements.txt
```

**Note**: This repository contains **architecture documentation only**. For a reference implementation, see the [BCH (Beacon Command Hub)](https://github.com/metaphyllc/bch) project, which demonstrates the C1 composite pattern with AI agents.

For now, we'll build a **minimal prototype** using Python scripts provided in `/samples/` directory.

---

## Phase 3: Moderator (M1) Setup

### 3.1 M1 Service Configuration

**Directory Structure** (on M1):
```
/opt/hcna-m1/
├── config.yaml          # M1 configuration
├── moderator.py         # Main M1 service
├── intent_queue.py      # Intent message queue
├── telemetry_collector.py  # W1 telemetry receiver
└── logs/
    └── moderator.log    # M1 logs
```

**Create Config** (`/opt/hcna-m1/config.yaml`):
```yaml
moderator:
  node_id: "HCNA-M1--prod--home--v1.0"
  listen_address: "0.0.0.0"  # Listen on all interfaces
  control_port: 8443          # M1 control API
  worker_nodes:
    - node_id: "HCNA-W1--prod--home--v1.0"
      address: "hcna-w1.local"
      port: 9443
  scheduling:
    max_concurrent_tasks: 10
    task_timeout_seconds: 300
  security:
    enable_mtls: true
    cert_path: "/opt/hcna-m1/certs/m1.crt"
    key_path: "/opt/hcna-m1/certs/m1.key"
    ca_path: "/opt/hcna-m1/certs/ca.crt"
```

### 3.2 Generate TLS Certificates (For mTLS)

**On M1** (generate CA and certificates for M1/W1):

```bash
# Install OpenSSL (should be installed already)
sudo apt install openssl

# Create certs directory
sudo mkdir -p /opt/hcna-m1/certs
cd /opt/hcna-m1/certs

# Generate CA key and certificate (self-signed for testing)
openssl req -x509 -newkey rsa:4096 -keyout ca.key -out ca.crt -days 365 -nodes \
  -subj "/C=US/ST=State/L=City/O=HCNA-3/OU=CA/CN=hcna-ca.local"

# Generate M1 key and CSR
openssl req -newkey rsa:4096 -keyout m1.key -out m1.csr -nodes \
  -subj "/C=US/ST=State/L=City/O=HCNA-3/OU=M1/CN=hcna-m1.local"

# Sign M1 certificate with CA
openssl x509 -req -in m1.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out m1.crt -days 365

# Generate W1 key and CSR
openssl req -newkey rsa:4096 -keyout w1.key -out w1.csr -nodes \
  -subj "/C=US/ST=State/L=City/O=HCNA-3/OU=W1/CN=hcna-w1.local"

# Sign W1 certificate with CA
openssl x509 -req -in w1.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out w1.crt -days 365

# Set permissions
sudo chmod 600 *.key
sudo chmod 644 *.crt

# Copy W1 certs to W1 node (via SCP)
scp w1.crt w1.key ca.crt hcna-admin@hcna-w1.local:/tmp/
```

### 3.3 Start M1 Service

**Systemd Service** (`/etc/systemd/system/hcna-m1.service`):

```ini
[Unit]
Description=HCNA-3 Moderator (M1) Service
After=network.target

[Service]
Type=simple
User=hcna-admin
WorkingDirectory=/opt/hcna-m1
ExecStart=/usr/bin/python3 /opt/hcna-m1/moderator.py
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

**Enable and Start**:
```bash
sudo systemctl daemon-reload
sudo systemctl enable hcna-m1
sudo systemctl start hcna-m1
sudo systemctl status hcna-m1  # Should show "active (running)"
```

**View Logs**:
```bash
journalctl -u hcna-m1 -f  # Follow logs in real-time
```

---

## Phase 4: Worker (W1) Setup

### 4.1 W1 Service Configuration

**Directory Structure** (on W1):
```
/opt/hcna-w1/
├── config.yaml          # W1 configuration
├── worker.py            # Main W1 service
├── task_executor.py     # Task execution engine
├── telemetry_reporter.py  # Send telemetry to M1
└── logs/
    └── worker.log       # W1 logs
```

**Create Config** (`/opt/hcna-w1/config.yaml`):
```yaml
worker:
  node_id: "HCNA-W1--prod--home--v1.0"
  listen_address: "0.0.0.0"
  worker_port: 9443
  moderator_node:
    node_id: "HCNA-M1--prod--home--v1.0"
    address: "hcna-m1.local"
    port: 8443
  resources:
    cpu_cores: 8          # Max cores to use
    memory_gb: 12         # Max RAM to allocate
    disk_path: "/data"    # Working directory for tasks
  security:
    enable_mtls: true
    cert_path: "/opt/hcna-w1/certs/w1.crt"
    key_path: "/opt/hcna-w1/certs/w1.key"
    ca_path: "/opt/hcna-w1/certs/ca.crt"
```

### 4.2 Install W1 Certificates

```bash
# Move certs from /tmp/ (where M1 SCP'd them) to proper location
sudo mkdir -p /opt/hcna-w1/certs
sudo mv /tmp/{w1.crt,w1.key,ca.crt} /opt/hcna-w1/certs/
sudo chmod 600 /opt/hcna-w1/certs/*.key
sudo chmod 644 /opt/hcna-w1/certs/*.crt
```

### 4.3 Start W1 Service

**Systemd Service** (`/etc/systemd/system/hcna-w1.service`):

```ini
[Unit]
Description=HCNA-3 Worker (W1) Service
After=network.target

[Service]
Type=simple
User=hcna-admin
WorkingDirectory=/opt/hcna-w1
ExecStart=/usr/bin/python3 /opt/hcna-w1/worker.py
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

**Enable and Start**:
```bash
sudo systemctl daemon-reload
sudo systemctl enable hcna-w1
sudo systemctl start hcna-w1
sudo systemctl status hcna-w1  # Should show "active (running)"
```

**Verify M1↔W1 Communication**:

Check M1 logs:
```bash
journalctl -u hcna-m1 -n 50  # Last 50 lines
# Should see: "W1 node HCNA-W1--prod--home--v1.0 registered successfully"
```

Check W1 logs:
```bash
journalctl -u hcna-w1 -n 50
# Should see: "Connected to M1 HCNA-M1--prod--home--v1.0 at hcna-m1.local:8443"
```

---

## Phase 5: Composite Node (C1) Setup

### 5.1 C1 API Gateway (On M1 or Separate Host)

**Option A: C1 on M1** (simplest for prototypes)
- C1 API runs on M1, forwards client requests to M1 control plane
- Clients connect to `hcna-c1.local` (which resolves to M1's IP)

**Option B: C1 on Separate Host** (production-ready)
- C1 runs on dedicated server or load balancer
- C1 proxies requests to M1, fetches results from W1

**For Prototype, Use Option A**:

**Directory Structure** (on M1):
```
/opt/hcna-c1/
├── config.yaml
├── composite_api.py     # C1 REST API server
└── logs/
    └── composite.log
```

**Create Config** (`/opt/hcna-c1/config.yaml`):
```yaml
composite:
  node_id: "HCNA-C1--prod--home--v1.0"
  listen_address: "0.0.0.0"
  api_port: 8080           # Public API port (HTTP for testing, HTTPS for prod)
  moderator_backend:
    address: "localhost"   # M1 on same host
    port: 8443
  security:
    enable_tls: false      # Set to true for production with proper cert
```

### 5.2 Start C1 Service

**Systemd Service** (`/etc/systemd/system/hcna-c1.service`):

```ini
[Unit]
Description=HCNA-3 Composite (C1) API Gateway
After=network.target hcna-m1.service

[Service]
Type=simple
User=hcna-admin
WorkingDirectory=/opt/hcna-c1
ExecStart=/usr/bin/python3 /opt/hcna-c1/composite_api.py
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

**Enable and Start**:
```bash
sudo systemctl daemon-reload
sudo systemctl enable hcna-c1
sudo systemctl start hcna-c1
sudo systemctl status hcna-c1  # Should show "active (running)"
```

### 5.3 Test C1 API

**From any client** (laptop, phone with curl):

```bash
# Health check
curl http://hcna-c1.local:8080/health

# Expected response:
# {
#   "status": "healthy",
#   "node_id": "HCNA-C1--prod--home--v1.0",
#   "moderator_connected": true,
#   "worker_nodes": 1
# }
```

**Submit a Test Task**:

```bash
curl -X POST http://hcna-c1.local:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "task_type": "echo",
    "payload": {"message": "Hello, HCNA-3!"}
  }'

# Expected response:
# {
#   "task_id": "task-abc123",
#   "status": "queued",
#   "message": "Task submitted to M1 for scheduling"
# }
```

**Check Task Status**:

```bash
curl http://hcna-c1.local:8080/tasks/task-abc123

# Expected response (after W1 completes):
# {
#   "task_id": "task-abc123",
#   "status": "completed",
#   "result": {
#     "message": "Hello, HCNA-3!",
#     "processed_by": "HCNA-W1--prod--home--v1.0",
#     "duration_ms": 42
#   }
# }
```

---

## Phase 6: Observability Setup

### 6.1 Prometheus (Metrics Collection)

**Install Prometheus** (on M1 or dedicated monitoring host):

```bash
# Download Prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
sudo mv prometheus-2.45.0.linux-amd64 /opt/prometheus
cd /opt/prometheus

# Create config (prometheus.yml)
cat <<EOF | sudo tee prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'hcna-m1'
    static_configs:
      - targets: ['hcna-m1.local:8443']

  - job_name: 'hcna-w1'
    static_configs:
      - targets: ['hcna-w1.local:9443']

  - job_name: 'hcna-c1'
    static_configs:
      - targets: ['hcna-c1.local:8080']
EOF

# Start Prometheus
./prometheus --config.file=prometheus.yml &
```

**Access Prometheus UI**: `http://hcna-m1.local:9090`

### 6.2 Grafana (Visualization)

**Install Grafana**:

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana

sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

**Access Grafana**: `http://hcna-m1.local:3000` (default login: admin/admin)

**Add Prometheus Data Source**:
1. Login to Grafana
2. Configuration → Data Sources → Add Prometheus
3. URL: `http://localhost:9090`
4. Save & Test

**Import HCNA-3 Dashboard**:
- Dashboard JSON available in `/docs/monitoring/grafana-dashboard.json` (coming soon)

### 6.3 Logging (Centralized)

**Option A: Journalctl** (simplest)
- Logs stored locally on M1/W1
- View with: `journalctl -u hcna-m1 -f`

**Option B: Loki + Promtail** (production)
- Centralized log aggregation
- Query logs from Grafana UI

---

## Phase 7: Testing & Validation

### 7.1 Functional Tests

**Test 1: M1↔W1 Communication**:
```bash
# On M1, send test intent to W1:
curl -X POST http://localhost:8443/admin/send-intent \
  -H "Content-Type: application/json" \
  -d '{
    "dst": "HCNA-W1--prod--home--v1.0",
    "intent": "test.ping",
    "payload": {}
  }'

# Check W1 received and responded:
journalctl -u hcna-w1 -n 10 | grep "test.ping"
# Should see: "Received intent test.ping from M1"
```

**Test 2: End-to-End Task Execution**:
```bash
# Submit task via C1:
TASK_ID=$(curl -X POST http://hcna-c1.local:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"task_type": "echo", "payload": {"msg": "test"}}' \
  | jq -r '.task_id')

# Poll for completion:
while true; do
  STATUS=$(curl -s http://hcna-c1.local:8080/tasks/$TASK_ID | jq -r '.status')
  echo "Task $TASK_ID status: $STATUS"
  [ "$STATUS" = "completed" ] && break
  sleep 2
done

# Get result:
curl http://hcna-c1.local:8080/tasks/$TASK_ID | jq '.result'
```

### 7.2 Performance Tests

**Test 3: Throughput**:
```bash
# Submit 100 tasks in parallel:
for i in {1..100}; do
  curl -X POST http://hcna-c1.local:8080/tasks \
    -H "Content-Type: application/json" \
    -d "{\"task_type\": \"echo\", \"payload\": {\"id\": $i}}" &
done
wait

# Check M1 metrics for task completion rate:
curl http://hcna-m1.local:8443/metrics | grep 'tasks_completed_total'
```

**Test 4: Latency**:
```bash
# Measure end-to-end latency for single task:
time (TASK_ID=$(curl -s -X POST http://hcna-c1.local:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"task_type": "echo", "payload": {}}' \
  | jq -r '.task_id') && \
  while [ "$(curl -s http://hcna-c1.local:8080/tasks/$TASK_ID | jq -r '.status')" != "completed" ]; do sleep 0.1; done)
```

### 7.3 Failure Scenarios

**Test 5: W1 Node Failure**:
```bash
# Stop W1:
ssh hcna-admin@hcna-w1.local "sudo systemctl stop hcna-w1"

# Submit task (should queue, not complete):
curl -X POST http://hcna-c1.local:8080/tasks \
  -d '{"task_type": "echo", "payload": {}}'

# Check M1 logs (should show "No healthy workers available"):
journalctl -u hcna-m1 -n 20

# Restart W1:
ssh hcna-admin@hcna-w1.local "sudo systemctl start hcna-w1"

# Queued tasks should now execute
```

**Test 6: Network Partition**:
```bash
# On M1, block traffic to W1 (simulate network failure):
sudo iptables -A OUTPUT -d 192.168.1.101 -j DROP

# M1 should detect W1 unhealthy after health check timeout:
journalctl -u hcna-m1 -f | grep "health check failed"

# Restore connectivity:
sudo iptables -F  # Clear all rules
```

---

## Phase 8: Production Hardening (Optional)

### 8.1 Security Checklist

- [ ] Enable mTLS for all M1↔W1 communication (already done if using certs)
- [ ] Enable HTTPS for C1 API (get Let's Encrypt cert or use self-signed)
- [ ] Firewall rules:
  - M1: Allow 8443 (control), 8080 (C1 API)
  - W1: Allow 9443 (worker)
  - Block all other inbound ports
- [ ] Rotate TLS certificates every 90 days (automate with `certbot` or custom script)
- [ ] Enable audit logging for all API calls (store in `/var/log/hcna/audit.log`)

### 8.2 High Availability (HA)

**M1 HA** (Multi-Moderator with Raft/etcd consensus):
- Deploy 3 M1 nodes (M1, M2, M3) with Raft leader election
- Active M1 handles scheduling, standby M1s take over on failure
- Requires etcd or Consul for distributed state

**W1 HA** (Multiple Workers):
- Deploy W2, W3, ... additional workers
- M1 load-balances tasks across W1, W2, W3
- If W1 fails, M1 reschedules tasks to W2

**C1 HA** (Load Balancer):
- Use NGINX or HAProxy in front of multiple C1 instances
- VIP (192.168.1.100) points to load balancer, not single M1

### 8.3 Backup & Disaster Recovery

**What to Backup**:
- M1: Task queue state, configuration files, certificates
- W1: Work-in-progress data in `/data` directory
- Metrics: Prometheus TSDB (optional, can re-collect)

**Backup Script** (run nightly via cron):
```bash
#!/bin/bash
# /opt/hcna-backup.sh

BACKUP_DIR="/mnt/backup/hcna-$(date +%Y-%m-%d)"
mkdir -p $BACKUP_DIR

# Backup M1 config and certs
tar czf $BACKUP_DIR/m1-config.tar.gz /opt/hcna-m1/config.yaml /opt/hcna-m1/certs/

# Backup W1 data directory
tar czf $BACKUP_DIR/w1-data.tar.gz /data/

# Backup Prometheus metrics (optional)
tar czf $BACKUP_DIR/prometheus-data.tar.gz /opt/prometheus/data/

# Sync to remote storage (S3, NAS, etc.)
# aws s3 sync $BACKUP_DIR s3://my-bucket/hcna-backups/
```

---

## Phase 9: Integration with Proprietary Technologies (Commercial)

**This section requires Metaphy LLC commercial licensing.**

### 9.1 QEGG Integration

**Install QEGG**:
- Contact logan@metaphysicsandcomputing.com for QEGG licensing
- Install Blender 4.0+ on W1
- Copy proprietary QEGG Python scripts to `/opt/qegg/`
- Update W1 config to recognize `qegg.generate` intent type

**Test QEGG Task**:
```bash
curl -X POST http://hcna-c1.local:8080/tasks \
  -d '{
    "task_type": "qegg.generate",
    "payload": {
      "grid_level": 4,
      "output_formats": ["glb", "obj"]
    }
  }'
```

### 9.2 DRGFC Integration

**Install DRGFC**:
- Contact logan@metaphysicsandcomputing.com for DRGFC licensing
- Install Python implementation: `pip install drgfc` (or Rust binary)
- Update W1 config to recognize `drgfc.compress` and `drgfc.decompress` intent types

**Test DRGFC Task**:
```bash
# Upload file to W1 /data directory first:
scp my_file.txt hcna-admin@hcna-w1.local:/data/

# Submit compression task:
curl -X POST http://hcna-c1.local:8080/tasks \
  -d '{
    "task_type": "drgfc.compress",
    "payload": {
      "input_file": "/data/my_file.txt",
      "output_file": "/data/my_file.drgfc"
    }
  }'
```

### 9.3 BPCS Integration

**Install BPCS**:
- Contact logan@metaphysicsandcomputing.com for BPCS licensing
- Install BPCS Python library: `pip install bpcs-crypto`
- Update M1/W1 configs to enable BPCS encryption layer:
  ```yaml
  security:
    enable_bpcs: true
    bpcs_key_exchange: "diffie-hellman"  # Or pre-shared keys
  ```

**All M1→W1 intents and W1→M1 telemetry will now be BPCS-encrypted.**

---

## Troubleshooting

### Common Issues

**Issue 1: W1 not connecting to M1**
- **Check**: Network connectivity (`ping hcna-m1.local`)
- **Check**: Firewall rules (M1 port 8443 open?)
- **Check**: Certificate mismatch (regenerate certs if needed)
- **Logs**: `journalctl -u hcna-w1 -n 50`

**Issue 2: Tasks stuck in "queued" status**
- **Check**: W1 service running (`systemctl status hcna-w1`)
- **Check**: M1 scheduler active (`journalctl -u hcna-m1 | grep "scheduler"`)
- **Check**: Resource limits (is W1 out of CPU/RAM?)

**Issue 3: High latency (>1s per task)**
- **Check**: Network latency (`ping hcna-w1.local` — should be <5ms)
- **Check**: W1 CPU usage (`htop` on W1 — is it maxed out?)
- **Check**: M1 task queue size (if queue is large, add more workers)

**Issue 4: Certificate errors**
- **Symptom**: "TLS handshake failed" in logs
- **Fix**: Regenerate certificates, ensure CN matches hostname
- **Test**: `openssl s_client -connect hcna-m1.local:8443` (should show cert details)

---

## Next Steps

### After Successful Deployment

1. **Scale Out**: Add W2, W3 workers to increase throughput
2. **Integrate QEGG/DRGFC**: License proprietary technologies for real workloads
3. **Deploy to Cloud**: Run M1 on AWS, W1 on GCP, validate cross-region HCNA-3
4. **Explore HMSS**: Deploy 10 M+W pairs, federate with HMSS orchestrator (future)

### Community & Support

- **GitHub Issues**: Report bugs, request features
- **Discussions**: Ask questions, share your HCNA-3 builds
- **Email**: logan@metaphysicsandcomputing.com for commercial support

---

## Appendix: Reference Implementation Scripts

Sample scripts (`moderator.py`, `worker.py`, `composite_api.py`) available in `/samples/` directory.

**Disclaimer**: These are **minimal prototypes** for educational purposes. Production deployments should use battle-tested frameworks (FastAPI, gRPC, Celery) and follow software engineering best practices.

---

## Appendix: Hardware Specifications (Metaphy LLC Prototype)

**Logan's 2-Laptop HCNA-3 Build**:

**M1 (Moderator Laptop)**:
- Model: TBD (older laptop, 2-4 cores sufficient)
- OS: Ubuntu 22.04 Server
- Role: Task orchestration, scheduling, health monitoring
- Workload: Lightweight, CPU-idle most of the time

**W1 (Worker Laptop)**:
- Model: TBD (gaming laptop or workstation, 8+ cores, 16GB+ RAM)
- OS: Ubuntu 22.04 Desktop (for Blender GUI access if needed)
- Role: QEGG model generation (Blender), DRGFC compression/decompression
- Workload: CPU-intensive, high RAM usage during tasks

**Network**:
- Home LAN (1 Gbps Ethernet or WiFi 6)
- M1 and W1 on same subnet (192.168.1.x)
- VIP for C1 API: 192.168.1.100 (points to M1)

**Storage**:
- M1: 20GB used (mostly logs and configs)
- W1: 500GB+ SSD for QEGG models, DRGFC input/output files

**Future Upgrade Path**:
- Add W2 (third laptop) for parallel QEGG generation
- Add S1 (NAS or external drive) for shared storage between W1/W2
- Add O1 (Raspberry Pi) running Grafana/Prometheus for dedicated observability

---

**Document Version**: 1.0.0
**Last Updated**: 2026-02-11
**Author**: CLIO (Team Brain) on behalf of Randell Logan Smith / Metaphy LLC
