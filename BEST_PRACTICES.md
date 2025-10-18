# AWS GitOps Best Practices (แนวทางปฏิบัติที่ดีที่สุด)

## Table of Contents
- [Repository Structure](#repository-structure)
- [Security](#security)
- [Deployment Strategy](#deployment-strategy)
- [Monitoring and Observability](#monitoring-and-observability)
- [Disaster Recovery](#disaster-recovery)
- [Cost Optimization](#cost-optimization)

## Repository Structure

### Recommended Layout

```
aws-gitops/
├── infrastructure/              # Infrastructure as Code
│   ├── terraform/
│   │   ├── environments/
│   │   │   ├── dev/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   └── modules/
│   └── cloudformation/
├── kubernetes/                  # Kubernetes manifests
│   ├── base/                   # Base configurations
│   └── overlays/               # Environment overlays
│       ├── dev/
│       ├── staging/
│       └── production/
├── applications/               # Application configs
│   └── my-app/
│       ├── base/
│       └── overlays/
├── ci-cd/                     # CI/CD configurations
│   ├── github-actions/
│   ├── jenkins/
│   └── gitlab-ci/
├── docs/                      # Documentation
└── scripts/                   # Helper scripts
```

**ภาษาไทย:** โครงสร้างที่แนะนำสำหรับ Repository

### Best Practices

1. **Separate Concerns**
   - Keep infrastructure and application configs separate
   - Use different directories for different environments

2. **Use Kustomize**
   - Define base configurations
   - Use overlays for environment-specific changes

3. **Version Everything**
   - Tag all releases
   - Use semantic versioning (v1.2.3)

## Security

### Secrets Management

**❌ Never Do This:**
```yaml
# DON'T commit secrets to Git
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  password: cGFzc3dvcmQxMjM=  # ❌ BAD!
```

**✅ Do This Instead:**

#### Option 1: AWS Secrets Manager

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  annotations:
    # Reference from AWS Secrets Manager
    external-secrets.io/backend: secretsManager
    external-secrets.io/key: prod/database/password
```

#### Option 2: External Secrets Operator

```bash
# Install External Secrets Operator
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets \
  external-secrets/external-secrets \
  -n external-secrets-system \
  --create-namespace
```

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: db-secret
  data:
  - secretKey: password
    remoteRef:
      key: prod/database/password
```

#### Option 3: Sealed Secrets

```bash
# Install kubeseal
brew install kubeseal

# Encrypt secret
kubectl create secret generic db-secret \
  --from-literal=password=secret123 \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml
```

### IAM Best Practices

1. **Use IRSA (IAM Roles for Service Accounts)**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/my-app-role
```

2. **Principle of Least Privilege**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket/*",
        "arn:aws:s3:::my-bucket"
      ]
    }
  ]
}
```

3. **Enable MFA for Production**
```bash
# Require MFA for production changes
# In IAM policy
"Condition": {
  "BoolIfExists": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

### Network Security

1. **Use Private Subnets**
```hcl
# Terraform configuration
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  enable_nat_gateway = true
  private_subnets    = ["10.0.1.0/24", "10.0.2.0/24"]
}
```

2. **Implement Network Policies**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

3. **Use Security Groups**
```hcl
resource "aws_security_group" "eks_nodes" {
  name        = "eks-nodes"
  description = "Security group for EKS nodes"
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }
}
```

## Deployment Strategy

### Progressive Delivery

#### 1. Canary Deployment

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 5m}
      - setWeight: 40
      - pause: {duration: 5m}
      - setWeight: 60
      - pause: {duration: 5m}
      - setWeight: 80
      - pause: {duration: 5m}
```

#### 2. Blue-Green Deployment

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    blueGreen:
      activeService: my-app-active
      previewService: my-app-preview
      autoPromotionEnabled: false
```

### Rollback Strategy

1. **Automated Rollback**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  strategy:
    canary:
      analysis:
        templates:
        - templateName: error-rate
        startingStep: 2
      trafficRouting:
        istio:
          virtualService:
            name: my-app
```

2. **Manual Rollback**
```bash
# Rollback to previous version
argocd app rollback my-app

# Rollback to specific revision
argocd app rollback my-app 3
```

## Monitoring and Observability

### Metrics Collection

#### Prometheus Setup

```yaml
# Install Prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

#### Key Metrics to Monitor

1. **Application Metrics**
```promql
# Request rate
sum(rate(http_requests_total[5m])) by (service)

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# Latency (p95)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

2. **Cluster Metrics**
```promql
# Node CPU usage
1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))

# Pod memory usage
sum(container_memory_working_set_bytes) by (pod)

# Disk usage
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes
```

3. **GitOps Metrics**
```promql
# Application sync status
argocd_app_info{sync_status="Synced"}

# Sync duration
argocd_app_sync_duration_seconds
```

### Logging

#### CloudWatch Logs

```yaml
# Fluent Bit DaemonSet
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
data:
  output.conf: |
    [OUTPUT]
        Name cloudwatch_logs
        Match *
        region us-west-2
        log_group_name /aws/eks/cluster
        auto_create_group true
```

### Alerting

```yaml
# PrometheusRule for alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
spec:
  groups:
  - name: app
    rules:
    - alert: HighErrorRate
      expr: |
        sum(rate(http_requests_total{status=~"5.."}[5m])) / 
        sum(rate(http_requests_total[5m])) > 0.05
      for: 5m
      annotations:
        summary: High error rate detected
```

## Disaster Recovery

### Backup Strategy

1. **Velero Setup**
```bash
# Install Velero
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket my-backup-bucket \
  --backup-location-config region=us-west-2 \
  --snapshot-location-config region=us-west-2

# Create backup
velero backup create my-backup --include-namespaces myapp-production

# Restore backup
velero restore create --from-backup my-backup
```

2. **Database Backups**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:14
            command:
            - sh
            - -c
            - pg_dump -h $DB_HOST -U $DB_USER $DB_NAME | aws s3 cp - s3://backups/$(date +%Y%m%d).sql
```

### Multi-Region Setup

```hcl
# Primary region
module "eks_primary" {
  source = "./modules/eks"
  region = "us-west-2"
}

# Secondary region (DR)
module "eks_secondary" {
  source = "./modules/eks"
  region = "us-east-1"
}
```

## Cost Optimization

### 1. Right-sizing Resources

```yaml
# Use resource limits
resources:
  requests:
    cpu: 100m      # Start small
    memory: 128Mi
  limits:
    cpu: 200m      # Set reasonable limits
    memory: 256Mi
```

### 2. Autoscaling

```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

```yaml
# Cluster Autoscaler
helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=my-cluster
```

### 3. Use Spot Instances

```hcl
# Terraform - Mixed instance types
eks_managed_node_groups = {
  spot = {
    capacity_type = "SPOT"
    instance_types = ["t3.medium", "t3a.medium"]
    desired_size = 2
  }
}
```

### 4. Resource Cleanup

```bash
# Identify unused resources
kubectl get pods --all-namespaces | grep -v Running

# Clean up old images
kubectl delete pods --field-selector status.phase=Failed

# Remove unused PVCs
kubectl get pvc --all-namespaces
```

## CI/CD Best Practices

### 1. Branch Strategy

```
main (production)
  ↑
staging
  ↑
develop
  ↑
feature branches
```

### 2. Required Checks

```yaml
# GitHub Branch Protection
branches:
  - name: main
    protection:
      required_status_checks:
        strict: true
        contexts:
          - validate
          - security-scan
          - test
      required_pull_request_reviews:
        required_approving_review_count: 2
```

### 3. Automated Testing

```yaml
# Run tests before deployment
- name: Run Tests
  run: |
    npm test
    npm run e2e
```

## Summary Checklist

✅ **Security**
- [ ] No secrets in Git
- [ ] IAM roles with least privilege
- [ ] Network policies in place
- [ ] Security scanning enabled

✅ **Reliability**
- [ ] Multi-AZ deployment
- [ ] Health checks configured
- [ ] Automated backups
- [ ] Disaster recovery plan

✅ **Observability**
- [ ] Metrics collection
- [ ] Centralized logging
- [ ] Alerting configured
- [ ] Dashboards created

✅ **Cost**
- [ ] Resource limits set
- [ ] Autoscaling enabled
- [ ] Spot instances used
- [ ] Regular cleanup

✅ **Operations**
- [ ] Documentation updated
- [ ] Runbooks created
- [ ] On-call rotation
- [ ] Incident response plan

---

**ภาษาไทย:** ปฏิบัติตามแนวทางเหล่านี้เพื่อให้ระบบของคุณมีความปลอดภัย เสถียร และประหยัดค่าใช้จ่าย

**Remember:** Best practices evolve. Regularly review and update your practices!
