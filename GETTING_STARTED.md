# Getting Started with AWS GitOps (เริ่มต้นใช้งาน AWS GitOps)

## Quick Start Guide (คู่มือเริ่มต้นอย่างรวดเร็ว)

This guide will help you get started with GitOps on AWS in less than 30 minutes.

**ภาษาไทย:** คู่มือนี้จะช่วยให้คุณเริ่มต้นใช้งาน GitOps บน AWS ได้ภายในเวลาไม่ถึง 30 นาที

## Prerequisites Checklist

Before starting, make sure you have:

- [ ] AWS Account with admin access
- [ ] AWS CLI installed and configured
- [ ] kubectl installed (v1.28+)
- [ ] Terraform installed (v1.0+) OR eksctl
- [ ] Git installed
- [ ] Basic knowledge of Kubernetes
- [ ] Basic knowledge of AWS

## Option 1: Quick Start with eksctl (Recommended for Beginners)

**ภาษาไทย:** วิธีที่ 1: เริ่มต้นอย่างรวดเร็วด้วย eksctl (แนะนำสำหรับผู้เริ่มต้น)

### Step 1: Install Required Tools

```bash
# Install eksctl (macOS)
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl

# Install eksctl (Linux)
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Verify installation
eksctl version
```

### Step 2: Create EKS Cluster

```bash
# Create a simple cluster (takes ~15 minutes)
eksctl create cluster \
  --name my-gitops-cluster \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --managed

# Verify cluster is ready
kubectl get nodes
```

### Step 3: Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=ready pod --all -n argocd --timeout=300s

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
echo

# Access ArgoCD UI (in a new terminal)
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Step 4: Deploy Your First Application

```bash
# Clone this repository
git clone https://github.com/somkheartk/aws-gitops.git
cd aws-gitops

# Deploy the demo application
kubectl apply -f examples/simple-eks-gitops/argocd-application.yaml

# Check application status
kubectl get applications -n argocd
```

### Step 5: Access Your Application

```bash
# Check if application is synced
kubectl get application demo-app -n argocd

# Get the LoadBalancer URL
kubectl get svc -n demo-app
```

🎉 **Congratulations! You've deployed your first GitOps application!**

**ภาษาไทย:** 🎉 ยินดีด้วย! คุณได้ deploy application แบบ GitOps แรกของคุณแล้ว!

## Option 2: Production-Ready Setup with Terraform

**ภาษาไทย:** วิธีที่ 2: การตั้งค่าแบบ Production-Ready ด้วย Terraform

### Step 1: Configure Terraform

```bash
cd examples/terraform-eks-argocd

# Copy example variables
cp terraform.tfvars.example terraform.tfvars

# Edit variables
nano terraform.tfvars
```

### Step 2: Initialize and Apply

```bash
# Initialize Terraform
terraform init

# Review plan
terraform plan

# Apply configuration (takes ~20 minutes)
terraform apply

# Get kubectl config
aws eks update-kubeconfig --name $(terraform output -raw cluster_name) --region $(terraform output -raw region)
```

### Step 3: Access ArgoCD

```bash
# Get ArgoCD password
terraform output get_argocd_password | bash

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Common Commands

### kubectl Commands

```bash
# View all pods
kubectl get pods --all-namespaces

# View services
kubectl get svc --all-namespaces

# View deployments
kubectl get deployments --all-namespaces

# View logs
kubectl logs -f <pod-name> -n <namespace>

# Describe pod
kubectl describe pod <pod-name> -n <namespace>
```

### ArgoCD Commands

```bash
# Install ArgoCD CLI (macOS)
brew install argocd

# Install ArgoCD CLI (Linux)
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd /usr/local/bin/argocd

# Login to ArgoCD
argocd login localhost:8080

# List applications
argocd app list

# Get application status
argocd app get <app-name>

# Sync application
argocd app sync <app-name>

# View application history
argocd app history <app-name>
```

### AWS CLI Commands

```bash
# List EKS clusters
aws eks list-clusters --region us-west-2

# Describe cluster
aws eks describe-cluster --name my-cluster --region us-west-2

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-west-2

# List worker nodes
aws eks list-nodegroups --cluster-name my-cluster --region us-west-2
```

## Troubleshooting Common Issues

### Issue: Cannot connect to cluster

```bash
# Verify AWS credentials
aws sts get-caller-identity

# Update kubeconfig
aws eks update-kubeconfig --name <cluster-name> --region <region>

# Test connection
kubectl get nodes
```

### Issue: ArgoCD UI not accessible

```bash
# Check if ArgoCD pods are running
kubectl get pods -n argocd

# Check service
kubectl get svc argocd-server -n argocd

# Restart port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Issue: Application not syncing

```bash
# Check application status
kubectl describe application <app-name> -n argocd

# View ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller

# Force sync
argocd app sync <app-name> --force
```

### Issue: Pods not starting

```bash
# Describe pod to see events
kubectl describe pod <pod-name> -n <namespace>

# Check pod logs
kubectl logs <pod-name> -n <namespace>

# Check events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

## Next Steps

After completing the quick start:

1. ✅ **Explore Examples**
   - Try the [multi-environment setup](examples/multi-env-gitops/)
   - Learn about [Terraform + EKS](examples/terraform-eks-argocd/)

2. ✅ **Customize Your Setup**
   - Add your own applications
   - Configure monitoring and alerting
   - Implement CI/CD pipelines

3. ✅ **Learn Best Practices**
   - Review [Best Practices Guide](BEST_PRACTICES.md)
   - Implement security measures
   - Set up disaster recovery

4. ✅ **Join the Community**
   - Star this repository
   - Contribute improvements
   - Share your experience

## Estimated Costs

**Development/Testing:**
- EKS Cluster: ~$73/month
- 2x t3.medium nodes: ~$60/month
- Load Balancer: ~$20/month
- **Total: ~$150/month**

**Production:**
- EKS Cluster: ~$73/month
- 3x t3.large nodes: ~$180/month
- Multiple Load Balancers: ~$40/month
- **Total: ~$300+/month**

**ภาษาไทย - ค่าใช้จ่ายประมาณการ:**
- ระบบทดสอบ: ประมาณ 150 ดอลลาร์/เดือน
- ระบบ Production: ประมาณ 300+ ดอลลาร์/เดือน

## Cleanup Instructions (วิธีลบทรัพยากร)

### For eksctl Deployment

```bash
# Delete ArgoCD applications first
kubectl delete applications --all -n argocd

# Delete ArgoCD
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Delete cluster
eksctl delete cluster --name my-gitops-cluster --region us-west-2
```

### For Terraform Deployment

```bash
cd examples/terraform-eks-argocd

# Destroy all resources
terraform destroy

# Confirm with 'yes'
```

⚠️ **Warning:** Make sure to delete all resources to avoid ongoing charges!

**ภาษาไทย:** ⚠️ **คำเตือน:** ตรวจสอบให้แน่ใจว่าลบทรัพยากรทั้งหมดแล้ว เพื่อหลีกเลี่ยงค่าใช้จ่ายที่เกินความจำเป็น!

## Getting Help

If you encounter issues:

1. Check the [Troubleshooting section](#troubleshooting-common-issues)
2. Review AWS CloudWatch logs
3. Check ArgoCD logs: `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server`
4. Open an issue in this repository

## Additional Resources

- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---

**Ready to dive deeper? Check out our [examples](examples/) directory!**

**ภาษาไทย:** พร้อมที่จะเรียนรู้เพิ่มเติมแล้วใช่ไหม? ลองดูตัวอย่างใน [examples](examples/) directory!
