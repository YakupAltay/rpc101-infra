# GitHub Actions CI + ArgoCD

This repository uses **GitHub Actions for CI** (testing/validation) and **ArgoCD for CD** (deployment).

## CI Pipeline (`.github/workflows/ci-cd.yaml`)

**Triggers:**
- Push to `main` branch
- Pull requests to `main` branch

**Jobs:**

1. **lint-and-test** - Validates Helm charts
   - Lints helm charts
   - Templates charts for dev and production
   
2. **security-scan** - Scans for vulnerabilities
   - Uses Trivy to scan Kubernetes configurations
   - Uploads results to GitHub Security tab

3. **notify-argocd** - Confirms CI passed
   - ArgoCD automatically detects and syncs changes

## ArgoCD Setup

See [`argocd/README.md`](../argocd/README.md) for complete ArgoCD setup instructions.

**Quick start:**
```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Apply sealed secrets first
kubectl apply -f sealed-secrets/production/

# Deploy ArgoCD applications
kubectl apply -f argocd/application-dev.yaml
kubectl apply -f argocd/application-prod.yaml
```

## Workflow

1. **Make changes** to helm charts or values
2. **Create PR** → GitHub Actions validates
3. **Merge to main** → GitHub Actions CI passes
4. **ArgoCD detects** changes automatically
5. **ArgoCD syncs** to cluster
6. **Monitor** in ArgoCD UI

## No Secrets Needed in GitHub

Since ArgoCD handles deployment from within the cluster, you don't need to add kubeconfig secrets to GitHub.

## Local Testing

Test before pushing:

```bash
# Lint
helm lint ./helm

# Template
helm template rpc101-test ./helm -n rpc101-test
```
