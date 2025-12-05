# Sealed Secrets

This directory contains encrypted secrets for different environments. These files are **safe to commit** to Git as they can only be decrypted by the specific Kubernetes cluster they were created for.

## Directory Structure

```
sealed-secrets/
├── dev/                  # Development environment secrets
│   ├── postgres-credentials-sealed.yaml
│   ├── redis-auth-sealed.yaml
│   └── basic-auth-sealed.yaml
├── production/           # Production environment secrets
│   ├── postgres-credentials-sealed.yaml
│   ├── redis-auth-sealed.yaml
│   └── basic-auth-sealed.yaml
└── README.md
```

## Deployment

Apply sealed secrets **before** deploying with Helm:

```bash
# 1. Apply sealed secrets (they get decrypted by the controller)
kubectl apply -f sealed-secrets/production/

# 2. Deploy with Helm
helm install rpc101-prod ./helm -n rpc101-production --create-namespace \
  -f helm/values-production.yaml
```

## Creating New Sealed Secrets

```bash
# 1. Create plain secret (don't commit!)
kubectl create secret generic my-secret \
  --from-literal=key=value \
  --dry-run=client -o yaml > /tmp/plain-secret.yaml

# 2. Seal it
kubeseal -f /tmp/plain-secret.yaml \
  -w sealed-secrets/production/my-secret-sealed.yaml

# 3. Commit sealed version
git add sealed-secrets/production/my-secret-sealed.yaml
git commit -m "Add my-secret"

# 4. Delete plain version
rm /tmp/plain-secret.yaml
```

## Important

- ✅ **DO** commit sealed secrets to Git
- ✅ **DO** create separate sealed secrets for each environment/cluster
- ❌ **DON'T** commit plain secrets or values files with real passwords
- ❌ **DON'T** commit plain (unsealed) secrets
- ❌ **DON'T** share sealed secrets between different clusters
