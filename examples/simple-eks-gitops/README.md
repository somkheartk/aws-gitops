# Simple EKS GitOps Example

## Overview (ภาพรวม)

This example demonstrates a basic GitOps setup with Amazon EKS and ArgoCD.

**ภาษาไทย:** ตัวอย่างนี้แสดงการตั้งค่า GitOps พื้นฐานด้วย Amazon EKS และ ArgoCD

## Architecture

```
GitHub Repository
       ↓
    ArgoCD
       ↓
  Amazon EKS Cluster
       ↓
   Applications
```

## Prerequisites

- AWS CLI configured
- kubectl installed
- eksctl installed
- Terraform installed (optional)

## Step-by-Step Guide (คำแนะนำทีละขั้นตอน)

### Step 1: Create EKS Cluster

```bash
# Using eksctl
eksctl create cluster \
  --name gitops-demo \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

**ภาษาไทย:** สร้าง EKS cluster ด้วยคำสั่งข้างต้น

### Step 2: Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=ready pod --all -n argocd --timeout=300s
```

### Step 3: Access ArgoCD UI

```bash
# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access UI at: `https://localhost:8080`
- Username: `admin`
- Password: (from command above)

### Step 4: Deploy Application

```bash
# Apply the application manifest
kubectl apply -f application.yaml
```

### Step 5: Verify Deployment

```bash
# Check application status
kubectl get applications -n argocd

# Check deployed resources
kubectl get all -n demo-app
```

## Files in This Example

- `eks-cluster.yaml`: EKS cluster configuration
- `argocd-application.yaml`: ArgoCD application definition
- `app-manifests/`: Kubernetes manifests for the demo app

## Cleanup (ลบ Resources)

```bash
# Delete the application
kubectl delete -f application.yaml

# Delete ArgoCD
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Delete EKS cluster
eksctl delete cluster --name gitops-demo --region us-west-2
```

## Troubleshooting

### Issue: ArgoCD pods not starting
```bash
# Check pod status
kubectl get pods -n argocd

# Check pod logs
kubectl logs -n argocd <pod-name>
```

### Issue: Application not syncing
```bash
# Check application status
argocd app get <app-name>

# Force sync
argocd app sync <app-name>
```

## Next Steps

- Explore [multi-environment setup](../multi-env-gitops/)
- Learn about [Terraform + EKS](../terraform-eks-argocd/)
- Try [serverless GitOps](../serverless-gitops/)
