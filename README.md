# RPC101 Infrastructure

**Production-ready multi-tenant RPC infrastructure on Kubernetes with Kong API Gateway, PostgreSQL HA, Redis HA, and monitoring.**

---

## 🎯 Quick Start

### Development (Local Testing)

```bash
# Deploy development environment
cd /path/to/rpc101-infrastructure
./scripts/deploy.sh dev

# Check status
kubectl get pods -n rpc101-dev

# Access Kong Admin API
curl http://localhost:30081/status

# Cleanup
./scripts/cleanup.sh dev --force
```

### Production (8-Node Hetzner Cluster)

**See full production deployment guide:** [`docs/PRODUCTION-DEPLOYMENT.md`](docs/PRODUCTION-DEPLOYMENT.md)

```bash
# After setting up 8-node Hetzner cluster (3 masters + 5 workers):
./scripts/deploy.sh prod

# Expected pods:
# - 3x Kong Gateway replicas
# - 4x PostgreSQL instances (CloudNativePG)
# - 3x Redis + 3x Sentinel
# - 1x Prometheus + monitoring stack
```

---

## 📊 Architecture

### Development Environment

```
Single Node:
├── Kong (1 replica)
├── PostgreSQL (1 instance)
├── Redis (1 replica, no sentinel)
├── Prometheus
└── Storage: local-path
```

**Cost:** ~$15/month (1 node)  
**Use Case:** Local testing, development, POC

### Production Environment

```
8-Node Cluster (3 Masters + 5 Workers):

Control Plane (3x CPX32: 4vCPU, 8GB):
├── etcd quorum (3-way replication)
├── Kubernetes API servers
└── Schedulers

Data Plane (5x CPX42: 8vCPU, 16GB):
├── worker-1: Kong + Postgres
├── worker-2: Kong + Postgres
├── worker-3: Kong + Redis + Sentinel
├── worker-4: Redis + Sentinel + Postgres
└── worker-5: Redis + Sentinel + Prometheus

Storage: Longhorn (3-replica distributed)
```

**Cost:** $145.92/month (Hetzner Cloud)  
**Capacity:** 30-100 active tenants  
**Use Case:** Production multi-tenant RPC platform

---

## 🚀 Features

### High Availability

- ✅ **Control Plane**: 3-master etcd quorum (survives 1 master failure)
- ✅ **Kong Gateway**: 3 replicas with anti-affinity (survives 2 worker failures)
- ✅ **PostgreSQL**: CloudNativePG with 4 instances (automatic failover)
- ✅ **Redis**: 3 replicas + 3 Sentinel (automatic master election)
- ✅ **Storage**: Longhorn 3-way replication (survives 2 node failures)

### Multi-Tenant Support

- **Route Isolation**: Path-based routing (`/api/tenant-a/*`)
- **API Key Auth**: Per-tenant authentication (Kong key-auth plugin)
- **Rate Limiting**: Configurable per-tenant (Redis-backed)
- **Monitoring**: Per-tenant metrics via Prometheus
- **Onboarding**: Automated via `scripts/add-tenant.sh`

### Monitoring & Observability

- **Prometheus**: Metrics collection and alerting
- **Grafana**: Dashboards (optional)
- **Metrics Gateway**: Unified /metrics endpoint (nginx proxy)
- **Component Exporters**: Redis, PostgreSQL, Kong, Kubernetes state

---

## 📁 Repository Structure

```
rpc101-infrastructure/
├── README.md                          # This file
├── docs/
│   ├── PRODUCTION-DEPLOYMENT.md       # Complete production setup guide
│   └── KONG-OPERATIONS.md             # Kong configuration reference
├── helm/
│   ├── Chart.yaml                     # Umbrella chart definition
│   ├── values.yaml                    # Default values
│   ├── values-dev.yaml                # Development overrides
│   ├── values-prod.yaml               # Production overrides (8-node)
│   ├── charts/
│   │   ├── kong/                      # Kong Gateway chart
│   │   ├── postgresql/                # CloudNativePG chart
│   │   ├── redis/                     # Redis HA chart
│   │   └── monitoring/                # Prometheus + exporters
│   └── templates/
│       └── namespace.yaml             # Single namespace template
├── scripts/
│   ├── deploy.sh                      # Environment deployment
│   ├── cleanup.sh                     # Environment cleanup
│   └── add-tenant.sh                  # Tenant onboarding (created in Phase 5)
└── .gitignore
```

---

## 🛠️ Prerequisites

### Development

- **Kubernetes**: Any single-node cluster (k3s, RKE2, minikube)
- **kubectl**: Configured and working
- **Helm**: v3.12+
- **Storage**: local-path provisioner

### Production

- **Cloud Provider**: Hetzner Cloud (or equivalent)
- **Nodes**: 8 servers (3 masters + 5 workers)
- **Kubernetes**: RKE2 (installed via guide)
- **Storage**: Longhorn distributed storage
- **Network**: Private network (10.0.0.0/16)

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [`docs/PRODUCTION-DEPLOYMENT.md`](docs/PRODUCTION-DEPLOYMENT.md) | **Complete production deployment guide**<br>• Hetzner Cloud setup<br>• RKE2 HA cluster installation<br>• Longhorn storage<br>• Multi-tenant configuration<br>• Backup & disaster recovery |
| [`docs/KONG-OPERATIONS.md`](docs/KONG-OPERATIONS.md) | **Kong operations reference**<br>• Adding routes/services<br>• Consumer management<br>• Plugin configuration<br>• Admin API examples |

---

## 🎯 Common Tasks

### Deploy Development Environment

```bash
# Clone repository
git clone https://github.com/your-org/rpc101-infrastructure.git
cd rpc101-infrastructure

# Deploy
./scripts/deploy.sh dev

# Verify
kubectl get pods -n rpc101-dev
kubectl get svc -n rpc101-dev

# Test Kong
curl http://localhost:30081/status
```

### Deploy Production Environment

```bash
# 1. Setup 8-node Hetzner cluster (see PRODUCTION-DEPLOYMENT.md Phase 1-3)
# 2. Configure secrets
vim helm/values-prod.yaml  # Update passwords

# 3. Deploy
./scripts/deploy.sh prod

# 4. Add first tenant
./scripts/add-tenant.sh tenant-alice http://eth-node.example.com:8545

# 5. Test tenant endpoint
curl -X POST http://<KONG_IP>:30080/api/tenant-alice \
  -H "apikey: <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","id":1}'
```

### Upgrade Existing Deployment

```bash
# Edit values
vim helm/values-dev.yaml  # or values-prod.yaml

# Apply changes
./scripts/deploy.sh dev --upgrade  # or prod
```

### Cleanup Environment

```bash
# Remove development
./scripts/cleanup.sh dev --force

# Remove production (WARNING: deletes all data!)
./scripts/cleanup.sh prod --force

# Remove all environments
./scripts/cleanup.sh all --force
```

---

## 🔧 Configuration

### Development Values (`helm/values-dev.yaml`)

```yaml
# Minimal resources for single-node testing
kong:
  replicas: 1
  resources:
    requests: { memory: "512Mi", cpu: "250m" }

postgresql:
  instances: 1

redis:
  replicas: 1
  sentinel:
    enabled: false  # No sentinel in dev
```

### Production Values (`helm/values-prod.yaml`)

```yaml
# Full HA configuration for 8-node cluster
kong:
  replicas: 3
  resources:
    requests: { memory: "1Gi", cpu: "500m" }
  affinity:
    podAntiAffinity: ...  # Spread across workers

postgresql:
  instances: 4
  storage:
    storageClass: longhorn
    size: 50Gi

redis:
  replicas: 3
  sentinel:
    enabled: true
    replicas: 3
```

---

## 🚨 Troubleshooting

### Pods Stuck in Pending

```bash
kubectl describe pod <pod-name> -n rpc101-dev
# Check: Insufficient resources, PV not bound, node affinity mismatch
```

### Kong 502 Bad Gateway

```bash
kubectl logs -n rpc101-dev deployment/kong --tail=50
# Check: PostgreSQL connection, Redis connectivity
```

### Longhorn Volume Mount Failure (Production)

```bash
kubectl -n longhorn-system logs -l app=longhorn-manager
# Fix: Ensure open-iscsi installed on all workers
ssh root@worker-1
systemctl status iscsid
```

### Storage Class Not Found

```bash
# Development
kubectl get storageclass
# Should show: local-path

# Production
kubectl get storageclass
# Should show: longhorn (default)
```

---

## 📊 Monitoring

### Access Prometheus

```bash
kubectl port-forward -n rpc101-dev svc/prometheus 9090:9090
# Open: http://localhost:9090
```

### Access Metrics Gateway

```bash
# All metrics in one place
curl http://localhost:30091/metrics/kong
curl http://localhost:30091/metrics/redis
curl http://localhost:30091/metrics/postgres
curl http://localhost:30091/metrics/prometheus
```

### View Logs

```bash
# Kong logs
kubectl logs -n rpc101-dev -l app=kong --tail=50 -f

# All pods
kubectl logs -n rpc101-dev --all-containers --tail=50
```

---

## 🎓 Next Steps

1. **Deploy Development**: Test locally with `./scripts/deploy.sh dev`
2. **Read Production Guide**: [`docs/PRODUCTION-DEPLOYMENT.md`](docs/PRODUCTION-DEPLOYMENT.md)
3. **Setup Hetzner Cluster**: Provision 8 nodes (Phase 1-3)
4. **Deploy Production**: `./scripts/deploy.sh prod`
5. **Onboard Tenants**: Use `scripts/add-tenant.sh`
6. **Setup Monitoring**: Configure Grafana dashboards
7. **Configure Backup**: PostgreSQL scheduled backups
8. **Setup TLS**: cert-manager + Let's Encrypt

---

## 📞 Support

For production deployment assistance, see:
- **Full Setup Guide**: [`docs/PRODUCTION-DEPLOYMENT.md`](docs/PRODUCTION-DEPLOYMENT.md)
- **Kong Operations**: [`docs/KONG-OPERATIONS.md`](docs/KONG-OPERATIONS.md)

---

**Version:** 1.0  
**Last Updated:** November 6, 2025  
**Maintained By:** RPC101 Infrastructure Team
