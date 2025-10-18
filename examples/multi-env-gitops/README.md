# Multi-Environment GitOps Setup

## Overview

This example demonstrates how to manage multiple environments (dev, staging, production) using GitOps principles.

**ภาษาไทย:** ตัวอย่างนี้แสดงวิธีจัดการหลาย environment (dev, staging, production) โดยใช้หลักการ GitOps

## Architecture

```
Git Repository (Single Source of Truth)
│
├── base/                    # Common configurations
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
│
└── overlays/               # Environment-specific configs
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patches/
    ├── staging/
    │   ├── kustomization.yaml
    │   └── patches/
    └── production/
        ├── kustomization.yaml
        └── patches/
```

## Strategy

We use **Kustomize** for managing environment-specific configurations:

1. **Base**: Common configuration shared across all environments
2. **Overlays**: Environment-specific customizations
3. **GitOps**: ArgoCD monitors and syncs each environment

## Prerequisites

- Multiple EKS clusters (or namespaces for different environments)
- ArgoCD installed
- Kustomize (built into kubectl)

## Directory Structure

```
multi-env-gitops/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   ├── replica-count.yaml
│   │   └── resource-limits.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   ├── replica-count.yaml
│   │   └── resource-limits.yaml
│   └── production/
│       ├── kustomization.yaml
│       ├── replica-count.yaml
│       ├── resource-limits.yaml
│       └── hpa.yaml
└── argocd-apps/
    ├── dev-app.yaml
    ├── staging-app.yaml
    └── production-app.yaml
```

## Environment Differences

| Feature | Dev | Staging | Production |
|---------|-----|---------|------------|
| Replicas | 1 | 2 | 3+ |
| Resources | Small | Medium | Large |
| Auto-scaling | No | Optional | Yes |
| Monitoring | Basic | Enhanced | Full |
| Secrets | Shared | Dedicated | Dedicated |

## Setup Instructions (คำแนะนำการตั้งค่า)

### Step 1: Create Base Configuration

The base configuration contains resources common to all environments.

```bash
kubectl kustomize base/
```

### Step 2: Create Environment Overlays

Each overlay modifies the base for specific environment needs.

**Dev Environment:**
```bash
kubectl kustomize overlays/dev/
```

**Staging Environment:**
```bash
kubectl kustomize overlays/staging/
```

**Production Environment:**
```bash
kubectl kustomize overlays/production/
```

### Step 3: Deploy with ArgoCD

```bash
# Deploy dev environment
kubectl apply -f argocd-apps/dev-app.yaml

# Deploy staging environment
kubectl apply -f argocd-apps/staging-app.yaml

# Deploy production environment
kubectl apply -f argocd-apps/production-app.yaml
```

### Step 4: Verify Deployments

```bash
# Check ArgoCD applications
kubectl get applications -n argocd

# Check deployed resources in each environment
kubectl get all -n myapp-dev
kubectl get all -n myapp-staging
kubectl get all -n myapp-production
```

## GitOps Workflow (ขั้นตอนการทำงาน)

### Development Workflow

1. **Developer makes changes**
   ```bash
   git checkout -b feature/new-feature
   # Make changes to base/ or overlays/dev/
   git commit -m "Add new feature"
   git push origin feature/new-feature
   ```

2. **Create Pull Request**
   - PR is reviewed by team
   - Automated tests run
   - Changes are validated

3. **Merge to main**
   ```bash
   git checkout main
   git merge feature/new-feature
   git push origin main
   ```

4. **ArgoCD auto-deploys to dev**
   - ArgoCD detects changes
   - Syncs dev environment automatically

5. **Promote to staging**
   ```bash
   # Update staging overlay to use new version
   git checkout -b promote/staging
   # Update overlays/staging/kustomization.yaml
   git commit -m "Promote to staging"
   git push origin promote/staging
   ```

6. **Manual approval for production**
   - Create PR for production promotion
   - Require approvals from team leads
   - Run smoke tests
   - Manual sync in ArgoCD (if not auto)

## Promotion Strategy

### Automatic Promotion (Dev → Staging)

```yaml
# In ArgoCD Application
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Manual Promotion (Staging → Production)

```yaml
# In ArgoCD Application  
spec:
  syncPolicy:
    # No automated section = manual sync required
    syncOptions:
      - CreateNamespace=true
```

## Best Practices

1. **Never commit to production directly**
   - Always promote through environments
   - Require multiple approvals

2. **Use separate AWS accounts per environment**
   ```
   Dev Account      → Staging Account → Production Account
   (123456789)        (234567890)        (345678901)
   ```

3. **Implement progressive delivery**
   - Canary deployments
   - Blue-green deployments
   - Feature flags

4. **Monitor everything**
   - Application metrics
   - Infrastructure metrics
   - Cost metrics

5. **Automate testing**
   - Unit tests on PR
   - Integration tests on dev
   - Smoke tests on staging
   - Full E2E on production promotion

## Rollback Strategy

### Quick Rollback

```bash
# Rollback via ArgoCD
argocd app rollback myapp-production

# Or via Git
git revert <commit-hash>
git push origin main
```

### Emergency Rollback

```bash
# Scale down problematic deployment
kubectl scale deployment myapp --replicas=0 -n myapp-production

# Restore previous version from Git history
git checkout <previous-commit> -- overlays/production/
git commit -m "Emergency rollback"
git push origin main
```

## Monitoring and Observability

### Prometheus Queries

```promql
# Request rate by environment
sum(rate(http_requests_total[5m])) by (environment)

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) by (environment)

# Deployment success rate
sum(argocd_app_sync_total{phase="Succeeded"}) by (name)
```

### ArgoCD Health Checks

```bash
# Check application health
argocd app get myapp-production

# View sync history
argocd app history myapp-production

# View application details
argocd app diff myapp-production
```

## Troubleshooting

### Issue: Application not syncing

```bash
# Check ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller

# Manual sync
argocd app sync myapp-dev --force
```

### Issue: Kustomize build fails

```bash
# Validate kustomization
kubectl kustomize overlays/dev/ --validate

# Check for syntax errors
kustomize build overlays/dev/
```

### Issue: Different behavior across environments

```bash
# Compare rendered manifests
diff <(kubectl kustomize overlays/dev/) <(kubectl kustomize overlays/staging/)
```

## Security Considerations

1. **Secrets Management**
   - Use AWS Secrets Manager
   - Or External Secrets Operator
   - Never commit secrets to Git

2. **RBAC**
   - Developers can deploy to dev
   - Team leads approve staging
   - Admins control production

3. **Network Policies**
   - Isolate environments
   - Control inter-namespace communication

4. **Audit Logging**
   - Track all changes
   - Monitor ArgoCD sync events

## Cost Optimization

- **Dev**: Use spot instances
- **Staging**: Smaller instance types
- **Production**: Reserved instances
- **All**: Implement auto-scaling

## Next Steps

- Implement progressive delivery with Argo Rollouts
- Add automated testing pipelines
- Set up monitoring and alerting
- Implement disaster recovery procedures

## Resources

- [Kustomize Documentation](https://kustomize.io/)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
- [GitOps Principles](https://opengitops.dev/)
