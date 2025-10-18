# Terraform + EKS + ArgoCD Example

## Overview

This example shows how to provision an EKS cluster using Terraform and set up GitOps with ArgoCD.

**ภาษาไทย:** ตัวอย่างนี้แสดงวิธีสร้าง EKS cluster ด้วย Terraform และตั้งค่า GitOps ด้วย ArgoCD

## What You'll Learn

- Infrastructure as Code with Terraform
- AWS EKS cluster provisioning
- ArgoCD installation via Terraform
- GitOps workflow implementation

## Prerequisites

- AWS CLI configured with credentials
- Terraform >= 1.0
- kubectl
- Basic understanding of Terraform and Kubernetes

## Project Structure

```
terraform-eks-argocd/
├── main.tf              # Main Terraform configuration
├── variables.tf         # Input variables
├── outputs.tf           # Output values
├── provider.tf          # Provider configuration
├── vpc.tf              # VPC and networking
├── eks.tf              # EKS cluster configuration
├── argocd.tf           # ArgoCD installation
└── README.md           # This file
```

## Quick Start (เริ่มต้นอย่างรวดเร็ว)

### Step 1: Initialize Terraform

```bash
cd examples/terraform-eks-argocd
terraform init
```

### Step 2: Review and Customize Variables

Edit `terraform.tfvars`:

```hcl
cluster_name = "my-gitops-cluster"
region       = "us-west-2"
environment  = "dev"
```

### Step 3: Plan Infrastructure

```bash
terraform plan -out=tfplan
```

### Step 4: Apply Configuration

```bash
terraform apply tfplan
```

This will create:
- VPC with public and private subnets
- EKS cluster with managed node groups
- ArgoCD installed on the cluster
- All necessary IAM roles and policies

**Estimated time:** 15-20 minutes

### Step 5: Configure kubectl

```bash
aws eks update-kubeconfig --name $(terraform output -raw cluster_name) --region $(terraform output -raw region)
```

### Step 6: Access ArgoCD

```bash
# Get ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Open browser at https://localhost:8080
```

## Architecture

```
┌─────────────────────────────────────────────────┐
│               AWS Account                        │
│                                                  │
│  ┌────────────────────────────────────────┐    │
│  │           VPC (10.0.0.0/16)            │    │
│  │                                         │    │
│  │  ┌──────────────┐  ┌──────────────┐   │    │
│  │  │Public Subnet │  │Public Subnet │   │    │
│  │  │  (AZ-1)      │  │  (AZ-2)      │   │    │
│  │  └──────────────┘  └──────────────┘   │    │
│  │                                         │    │
│  │  ┌──────────────┐  ┌──────────────┐   │    │
│  │  │Private Subnet│  │Private Subnet│   │    │
│  │  │  (AZ-1)      │  │  (AZ-2)      │   │    │
│  │  │              │  │              │   │    │
│  │  │ ┌──────────┐ │  │ ┌──────────┐ │   │    │
│  │  │ │EKS Nodes │ │  │ │EKS Nodes │ │   │    │
│  │  │ │(t3.medium)│ │  │ │(t3.medium)│ │   │    │
│  │  │ └──────────┘ │  │ └──────────┘ │   │    │
│  │  └──────────────┘  └──────────────┘   │    │
│  │                                         │    │
│  └────────────────────────────────────────┘    │
│                                                  │
└─────────────────────────────────────────────────┘
```

## Cost Estimate

**Monthly AWS Costs (approximate):**
- EKS Cluster: $73/month
- t3.medium nodes (2x): ~$60/month
- Load Balancers: ~$20/month
- Data Transfer: Variable

**Total: ~$150-200/month**

**ภาษาไทย - ค่าใช้จ่าย:** ประมาณ 150-200 ดอลลาร์ต่อเดือน

## Cleanup (ลบทรัพยากร)

⚠️ **Warning:** This will delete all resources and cannot be undone.

```bash
# Delete all Kubernetes resources first
kubectl delete applications --all -n argocd

# Destroy infrastructure
terraform destroy
```

## Customization

### Change Node Instance Type

Edit `variables.tf`:

```hcl
variable "node_instance_type" {
  default = "t3.large"  # Change from t3.medium
}
```

### Enable Cluster Autoscaler

Add to `eks.tf`:

```hcl
# Uncomment cluster autoscaler section
```

### Add Additional Add-ons

Edit `eks.tf` to include:
- AWS Load Balancer Controller
- External DNS
- Cluster Autoscaler
- Metrics Server

## Troubleshooting

### Issue: Terraform timeout

```bash
# Increase timeout in provider configuration
terraform apply -timeout=30m
```

### Issue: kubectl access denied

```bash
# Verify AWS credentials
aws sts get-caller-identity

# Update kubeconfig
aws eks update-kubeconfig --name <cluster-name> --region <region>
```

### Issue: ArgoCD pods not running

```bash
# Check pod status
kubectl get pods -n argocd

# Check events
kubectl get events -n argocd --sort-by='.lastTimestamp'
```

## Security Best Practices

1. **Enable EKS encryption** - Encrypt secrets at rest
2. **Use private endpoints** - Restrict API server access
3. **Enable audit logging** - Track all API calls
4. **Implement network policies** - Control pod-to-pod communication
5. **Use IAM roles for service accounts** - Fine-grained permissions

## Next Steps

- Deploy your first application with ArgoCD
- Set up multi-environment deployments
- Implement progressive delivery with Argo Rollouts
- Configure monitoring and alerting

## Resources

- [Terraform EKS Module](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [EKS Best Practices](https://aws.github.io/aws-eks-best-practices/)
