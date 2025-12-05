# 🎯 RPC101 Helm Chart

Single-command deployment of the complete RPC101 infrastructure using Helm umbrella chart architecture.

## 🚀 Quick Start

```bash
# Deploy everything
helm install rpc101 . \
  -n rpc101-platform \
  --create-namespace \
  --dependency-update \
  --timeout 15m

# Verify
kubectl get pods -n redis-ha -n postgres-ha -n kong -n monitoring

# Test ETH RPC
curl -X POST http://$(minikube ip):30080/eth \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Uninstall
helm uninstall rpc101 -n rpc101-platform
kubectl delete namespace rpc101-platform redis-ha postgres-ha kong monitoring
```

## 📦 Architecture

This is an **umbrella chart** with 4 subcharts:

```
rpc101-infrastructure/
├── Chart.yaml              # Parent chart with dependencies
├── values.yaml             # Parent values (overrides subcharts)
├── templates/
│   └── namespaces.yaml    # Pre-creates all namespaces
└── charts/
    ├── redis/             # Redis HA with Sentinel
    ├── postgresql/        # PostgreSQL HA with CloudNativePG
    ├── kong/              # Kong Gateway with migrations
    └── monitoring/        # Prometheus + Exporters
```

## 🔧 Configuration

### Default Values

```yaml
redis:
  password: "redis-secret-password"
  replicas: 3

postgresql:
  username: "kong"
  password: "kong-secret-password"
  instances: 3

kong:
  replicas: 3
  adminGuiUrl: "http://138.201.120.45:8002"

monitoring:
  enabled: true
```

### Custom Values

```bash
# Override default values
helm install rpc101 . \
  -n rpc101-platform \
  --create-namespace \
  --dependency-update \
  --set redis.replicas=5 \
  --set kong.replicas=5 \
  --set postgresql.instances=5
```

Or create a custom `values-prod.yaml`:

```yaml
redis:
  replicas: 5
  password: "${REDIS_PASSWORD}"

postgresql:
  instances: 5
  password: "${POSTGRES_PASSWORD}"

kong:
  replicas: 5
```

Then install:

```bash
helm install rpc101 . \
  -n rpc101-platform \
  --create-namespace \
  --dependency-update \
  -f values-prod.yaml
```

## 🎨 Why Umbrella Chart?

### ✅ Advantages

1. **Single Command Deployment**: One `helm install` for everything
2. **Centralized Configuration**: Parent values override subcharts
3. **Dependency Management**: Automatic subchart coordination
4. **Version Control**: Lock subchart versions in `Chart.yaml`
5. **Clean Structure**: Each component isolated in its subchart
6. **No Build Artifacts**: No `.tgz` files or `Chart.lock` needed in git

### 🏗️ How It Works

```yaml
# Chart.yaml
dependencies:
  - name: redis
    version: 1.0.0
    repository: "file://charts/redis"
    condition: redis.enabled
```

When you run `helm install` with `--dependency-update`:
1. Helm reads dependencies from `Chart.yaml`
2. Loads subcharts from `file://charts/*` paths
3. Merges parent `values.yaml` with subchart values
4. Deploys everything in correct order

**No `helm dependency build` needed!** ✨

## 📊 Components

| Component | Namespace | Pods | Purpose |
|-----------|-----------|------|---------|
| Redis HA | redis-ha | 3+3 | Cache & rate limiting with Sentinel |
| PostgreSQL HA | postgres-ha | 3 | Database with CloudNativePG |
| Kong Gateway | kong | 3 | API Gateway with Admin API |
| Monitoring | monitoring | 4 | Prometheus + Exporters |

## 🔍 Troubleshooting

### Check deployment status
```bash
helm status rpc101 -n rpc101-platform
```

### View logs
```bash
kubectl logs -n kong -l app=kong --tail=50
kubectl logs -n postgres-ha -l cnpg.io/cluster=kong-postgres
kubectl logs -n redis-ha -l app=redis
```

### Debug values
```bash
# See computed values
helm get values rpc101 -n rpc101-platform

# See all values (including defaults)
helm get values rpc101 -n rpc101-platform --all
```

### Re-deploy
```bash
# Upgrade (keeps data)
helm upgrade rpc101 . -n rpc101-platform --dependency-update

# Force re-deploy
helm uninstall rpc101 -n rpc101-platform
helm install rpc101 . -n rpc101-platform --create-namespace --dependency-update
```

## 📁 Files Not in Git

```
# .gitignore excludes:
helm/charts/*.tgz        # Packaged charts
helm/Chart.lock          # Dependency lock file
```

These are generated at install time with `--dependency-update` and not needed in version control.

## 🎯 Production Checklist

- [ ] Change default passwords in `values.yaml`
- [ ] Set proper `kong.adminGuiUrl` for your domain
- [ ] Configure ETH node endpoints via Kong Admin API
- [ ] Enable rate limiting plugins
- [ ] Set up persistent volume claims for production
- [ ] Configure backup schedules for PostgreSQL
- [ ] Set resource limits/requests for all pods
- [ ] Enable TLS for Kong Admin API
- [ ] Configure monitoring alerts
- [ ] Test failover scenarios

## 🚀 Next Steps

1. **Configure ETH Nodes**: See [../docs/KONG-OPERATIONS.md](../docs/KONG-OPERATIONS.md)
2. **Production Deployment**: See [../docs/PRODUCTION-DEPLOYMENT.md](../docs/PRODUCTION-DEPLOYMENT.md)
3. **Multi-Tenant Setup**: See [../docs/MULTI-TENANT-ARCHITECTURE.md](../docs/MULTI-TENANT-ARCHITECTURE.md)

---

**Need help?** Check the main [README.md](../README.md) or documentation in `docs/` folder.
