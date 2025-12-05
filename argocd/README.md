# Environment-Specific Configuration

This directory contains environment-specific files that are **gitignored** for security.

## Setup Instructions

### 1. Create your ArgoCD application

Copy the template and customize:

```bash
cp application.yaml application-prod.yaml
# or
cp application.yaml application-dev.yaml
```

Update the placeholders:
- `ENVIRONMENT` → `prod` or `dev`
- Add your domain/IP in the `values:` section

### 2. Apply to cluster

```bash
kubectl apply -f application-prod.yaml
```

## Files

- `application.yaml` - Template (in git)
- `application-*.yaml` - Your configs (gitignored)

**Never commit** `application-prod.yaml` or `application-dev.yaml`!
