# AWS GitOps Cheat Sheet (สูตรลัด AWS GitOps)

Quick reference for common commands and workflows.

**ภาษาไทย:** คำสั่งที่ใช้บ่อยสำหรับ AWS GitOps

---

## 🚀 Quick Setup

### Install Tools

```bash
# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Terraform (check latest version at https://www.terraform.io/downloads)
TERRAFORM_VERSION="1.6.0"  # Update this to latest version
wget https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip
unzip terraform_${TERRAFORM_VERSION}_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# ArgoCD CLI
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd /usr/local/bin/argocd
```

---

## ☁️ AWS Commands

### Configure AWS

```bash
# Set up credentials
aws configure

# Verify identity
aws sts get-caller-identity

# List regions
aws ec2 describe-regions --query 'Regions[].RegionName' --output table
```

### EKS Management

```bash
# List clusters
aws eks list-clusters --region us-west-2

# Get cluster info
aws eks describe-cluster --name my-cluster --region us-west-2

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-west-2

# List node groups
aws eks list-nodegroups --cluster-name my-cluster --region us-west-2

# Describe node group
aws eks describe-nodegroup --cluster-name my-cluster --nodegroup-name my-nodes --region us-west-2
```

---

## ⚓ kubectl Commands

### Cluster Info

```bash
# Get cluster info
kubectl cluster-info

# View nodes
kubectl get nodes

# Node details
kubectl describe node <node-name>

# Check API versions
kubectl api-resources
```

### Resources

```bash
# List all resources
kubectl get all --all-namespaces

# List pods
kubectl get pods -n <namespace>
kubectl get pods --all-namespaces
kubectl get pods -o wide

# List services
kubectl get svc --all-namespaces

# List deployments
kubectl get deployments --all-namespaces

# List namespaces
kubectl get namespaces
```

### Debugging

```bash
# Describe resource
kubectl describe pod <pod-name> -n <namespace>

# View logs
kubectl logs <pod-name> -n <namespace>
kubectl logs -f <pod-name> -n <namespace>  # Follow logs

# Previous logs (if pod crashed)
kubectl logs <pod-name> -n <namespace> --previous

# Execute command in pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

# Port forward
kubectl port-forward pod/<pod-name> 8080:80 -n <namespace>
kubectl port-forward svc/<service-name> 8080:80 -n <namespace>

# Events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# Resource usage
kubectl top nodes
kubectl top pods -n <namespace>
```

### Apply/Delete

```bash
# Apply manifest
kubectl apply -f manifest.yaml

# Apply directory
kubectl apply -f ./manifests/

# Apply with kustomize
kubectl apply -k ./overlays/dev/

# Delete resources
kubectl delete -f manifest.yaml
kubectl delete pod <pod-name> -n <namespace>

# Delete all in namespace
kubectl delete all --all -n <namespace>
```

---

## 🏗️ eksctl Commands

### Cluster Management

```bash
# Create cluster
eksctl create cluster \
  --name my-cluster \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --managed

# Create with config file
eksctl create cluster -f cluster.yaml

# Delete cluster
eksctl delete cluster --name my-cluster --region us-west-2

# Get cluster
eksctl get cluster --name my-cluster --region us-west-2

# List clusters
eksctl get clusters
```

### Node Groups

```bash
# List node groups
eksctl get nodegroup --cluster=my-cluster

# Scale node group
eksctl scale nodegroup --cluster=my-cluster --name=my-nodes --nodes=3

# Delete node group
eksctl delete nodegroup --cluster=my-cluster --name=my-nodes
```

---

## 🔄 ArgoCD Commands

### Installation

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ready
kubectl wait --for=condition=ready pod --all -n argocd --timeout=300s

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### ArgoCD CLI

```bash
# Login
argocd login localhost:8080

# List applications
argocd app list

# Get app details
argocd app get <app-name>

# Create app
argocd app create <app-name> \
  --repo https://github.com/user/repo.git \
  --path ./manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Sync app
argocd app sync <app-name>

# Force sync
argocd app sync <app-name> --force

# Delete app
argocd app delete <app-name>

# View history
argocd app history <app-name>

# Rollback
argocd app rollback <app-name>
argocd app rollback <app-name> <revision-id>

# Set auto-sync
argocd app set <app-name> --sync-policy automated

# Diff
argocd app diff <app-name>
```

---

## 🔧 Terraform Commands

### Basic Workflow

```bash
# Initialize
terraform init

# Validate
terraform validate

# Format
terraform fmt

# Plan
terraform plan

# Apply
terraform apply

# Apply with auto-approve
terraform apply -auto-approve

# Destroy
terraform destroy

# Destroy specific resource
terraform destroy -target=aws_instance.example
```

### State Management

```bash
# List resources
terraform state list

# Show resource
terraform state show <resource>

# Remove from state
terraform state rm <resource>

# Import existing resource
terraform import <resource> <id>
```

### Workspaces

```bash
# List workspaces
terraform workspace list

# Create workspace
terraform workspace new dev

# Select workspace
terraform workspace select dev

# Delete workspace
terraform workspace delete dev
```

---

## 📦 Kustomize Commands

```bash
# Build kustomization
kubectl kustomize ./base

# Build overlay
kubectl kustomize ./overlays/dev

# Apply with kustomize
kubectl apply -k ./overlays/dev/

# View diff
kubectl diff -k ./overlays/dev/

# Validate
kustomize build ./overlays/dev/ --validate
```

---

## 🔐 Secrets Management

### Kubernetes Secrets

```bash
# Create secret from literal
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Create secret from file
kubectl create secret generic my-secret --from-file=./secret.txt

# Get secret
kubectl get secret my-secret -o yaml

# Decode secret
kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 -d
```

### AWS Secrets Manager

```bash
# Create secret
aws secretsmanager create-secret \
  --name prod/database/password \
  --secret-string "mypassword123"

# Get secret value
aws secretsmanager get-secret-value \
  --secret-id prod/database/password \
  --query SecretString \
  --output text

# Update secret
aws secretsmanager update-secret \
  --secret-id prod/database/password \
  --secret-string "newpassword456"

# Delete secret
aws secretsmanager delete-secret \
  --secret-id prod/database/password
```

---

## 📊 Monitoring

### CloudWatch Logs

```bash
# List log groups
aws logs describe-log-groups

# Tail logs
aws logs tail /aws/eks/cluster/cluster --follow

# Filter logs
aws logs filter-log-events \
  --log-group-name /aws/eks/cluster \
  --filter-pattern "ERROR"
```

### Prometheus Queries

```promql
# CPU usage
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Pod restart count
kube_pod_container_status_restarts_total

# HTTP request rate
sum(rate(http_requests_total[5m])) by (service)
```

---

## 🐛 Troubleshooting

### Common Issues

```bash
# Pod not starting
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>

# Service not accessible
kubectl get endpoints <service-name> -n <namespace>
kubectl describe svc <service-name> -n <namespace>

# Node issues
kubectl describe node <node-name>
kubectl get events --all-namespaces

# ArgoCD not syncing
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
argocd app get <app-name>

# Check resource quotas
kubectl describe resourcequota -n <namespace>

# Check pod limits
kubectl describe limitrange -n <namespace>
```

---

## 🔄 GitOps Workflow

### Standard Workflow

```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes
vim manifests/deployment.yaml

# 3. Commit changes
git add .
git commit -m "Update deployment"

# 4. Push to remote
git push origin feature/new-feature

# 5. Create PR (via GitHub/GitLab UI)

# 6. After merge, ArgoCD syncs automatically
argocd app sync <app-name>
```

### Rollback

```bash
# Via Git
git revert <commit-hash>
git push origin main

# Via ArgoCD
argocd app rollback <app-name>

# Via kubectl
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

---

## 💰 Cost Management

```bash
# List all EC2 instances
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]' \
  --output table

# List all EBS volumes
aws ec2 describe-volumes \
  --query 'Volumes[*].[VolumeId,Size,State]' \
  --output table

# List all load balancers
aws elb describe-load-balancers --query 'LoadBalancerDescriptions[*].LoadBalancerName'

# Get cost estimate
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost
```

---

## 🧹 Cleanup

```bash
# Delete all resources in namespace
kubectl delete all --all -n <namespace>

# Delete namespace
kubectl delete namespace <namespace>

# Delete cluster (eksctl)
eksctl delete cluster --name my-cluster --region us-west-2

# Delete cluster (terraform)
cd infrastructure/
terraform destroy

# Uninstall ArgoCD
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```

---

## 📖 Useful Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
# kubectl aliases
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kga='kubectl get all'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias klf='kubectl logs -f'
alias ke='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'

# ArgoCD aliases
alias ac='argocd'
alias acl='argocd app list'
alias acs='argocd app sync'
alias acg='argocd app get'

# Terraform aliases
alias tf='terraform'
alias tfi='terraform init'
alias tfp='terraform plan'
alias tfa='terraform apply'
alias tfd='terraform destroy'

# AWS aliases
alias eks-update='aws eks update-kubeconfig --name'
```

---

## 🔗 Quick Links

- **AWS EKS**: https://console.aws.amazon.com/eks
- **AWS CloudWatch**: https://console.aws.amazon.com/cloudwatch
- **ArgoCD Docs**: https://argo-cd.readthedocs.io
- **Kubernetes Docs**: https://kubernetes.io/docs
- **Terraform Docs**: https://www.terraform.io/docs

---

**ภาษาไทย:** บันทึกไฟล์นี้ไว้เพื่อใช้อ้างอิงเมื่อทำงานกับ AWS GitOps!

**Tip:** Bookmark this page for quick reference! 🔖
