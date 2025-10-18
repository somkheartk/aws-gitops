# AWS GitOps Tutorial Overview (สอนและทำตัวอย่าง AWS GitOps)

## 📖 What's Included (สิ่งที่รวมอยู่ในคู่มือนี้)

This comprehensive tutorial provides everything you need to implement GitOps on AWS, with bilingual support (English/Thai).

**ภาษาไทย:** คู่มือฉบับสมบูรณ์นี้มีทุกอย่างที่คุณต้องการสำหรับการใช้งาน GitOps บน AWS พร้อมภาษาไทย

---

## 📚 Documentation Files

### 1. [README.md](README.md)
**Main repository overview**
- What is GitOps?
- Architecture diagrams
- Prerequisites
- Quick links to examples
- Learning resources

**ภาษาไทย:** หน้าหลักของ repository พร้อมอธิบาย GitOps และสถาปัตยกรรม

### 2. [GETTING_STARTED.md](GETTING_STARTED.md)
**Step-by-step quick start guide**
- Two paths: Quick start (eksctl) and Production-ready (Terraform)
- Complete installation instructions
- Common commands reference
- Troubleshooting guide
- Cleanup instructions

**ภาษาไทย:** คู่มือเริ่มต้นอย่างละเอียด พร้อมคำแนะนำทีละขั้นตอน

### 3. [BEST_PRACTICES.md](BEST_PRACTICES.md)
**Production-grade best practices**
- Repository structure recommendations
- Security best practices (secrets, IAM, network)
- Deployment strategies (canary, blue-green)
- Monitoring and observability
- Disaster recovery
- Cost optimization

**ภาษาไทย:** แนวทางปฏิบัติที่ดีที่สุดสำหรับระดับ Production

### 4. [CONTRIBUTING.md](CONTRIBUTING.md)
**Contribution guidelines**
- How to report bugs
- How to suggest enhancements
- Pull request process
- Code style guidelines

**ภาษาไทย:** แนวทางการมีส่วนร่วมในโปรเจกต์

### 5. [LICENSE](LICENSE)
**MIT License** - Free to use, modify, and distribute

---

## 🚀 Examples (ตัวอย่าง)

### Example 1: Simple EKS GitOps
**Location:** [`examples/simple-eks-gitops/`](examples/simple-eks-gitops/)

**What's included:**
- Basic EKS cluster setup with eksctl
- ArgoCD installation and configuration
- Simple nginx demo application
- Kubernetes manifests (deployment, service, namespace)
- Step-by-step tutorial

**Best for:** Beginners, learning GitOps concepts

**Time to complete:** ~30 minutes

**Files:**
- `README.md` - Complete tutorial
- `argocd-application.yaml` - ArgoCD app definition
- `app-manifests/` - Kubernetes resources

---

### Example 2: Terraform + EKS + ArgoCD
**Location:** [`examples/terraform-eks-argocd/`](examples/terraform-eks-argocd/)

**What's included:**
- Production-ready Terraform code
- VPC with public/private subnets
- EKS cluster with managed node groups
- ArgoCD installed via Helm
- Full IaC approach

**Best for:** Production deployments, infrastructure automation

**Time to complete:** ~45-60 minutes (including EKS cluster creation which takes 15-20 minutes)

**Files:**
- `README.md` - Detailed guide
- `main.tf` - Main configuration
- `vpc.tf` - Network configuration
- `eks.tf` - EKS cluster setup
- `argocd.tf` - ArgoCD installation
- `variables.tf` - Input variables
- `outputs.tf` - Output values
- `terraform.tfvars.example` - Example variables

**Cost estimate:** ~$200-300/month
- EKS Control Plane: $73/month
- 2x t3.medium nodes: ~$60/month
- NAT Gateway: ~$45/month
- Load Balancers: ~$20-40/month
- Data transfer and storage: ~$10-20/month

---

### Example 3: Multi-Environment GitOps
**Location:** [`examples/multi-env-gitops/`](examples/multi-env-gitops/)

**What's included:**
- Kustomize-based configuration
- Base + overlays pattern
- Three environments: dev, staging, production
- Different configurations per environment
- Progressive delivery workflow
- ArgoCD applications for each environment

**Best for:** Managing multiple environments, real-world scenarios

**Structure:**
```
multi-env-gitops/
├── base/                    # Common configs
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/                 # Dev-specific
│   ├── staging/             # Staging-specific
│   └── production/          # Prod-specific (with HPA)
└── argocd-apps/            # ArgoCD definitions
    ├── dev-app.yaml
    ├── staging-app.yaml
    └── production-app.yaml
```

**Features:**
- ✅ Environment-specific resource limits
- ✅ Different replica counts per environment
- ✅ Horizontal Pod Autoscaler for production
- ✅ Automated deployment for dev/staging
- ✅ Manual approval for production

---

### Example 4: Serverless GitOps
**Location:** [`examples/serverless-gitops/`](examples/serverless-gitops/)

**What's included:**
- AWS Lambda functions
- Infrastructure as Code (Terraform & SAM)
- API Gateway configuration
- CI/CD for serverless
- Multi-environment management

**Best for:** Serverless applications, cost-conscious deployments

**Key concepts:**
- GitOps for Lambda functions
- Version control for serverless
- Automated testing and deployment
- Cost optimization strategies

**Cost estimate:** Much lower than EKS
- Low traffic (< 1M requests): ~$5-10/month
- Medium traffic (1-10M requests): ~$20-40/month  
- High traffic (10M+ requests): $50+/month
- Plus API Gateway costs: ~$3.50 per million requests

---

## 🔄 CI/CD Pipeline

**Location:** [`.github/workflows/deploy-to-eks.yml`](.github/workflows/deploy-to-eks.yml)

**Features:**
- ✅ Kubernetes manifest validation
- ✅ Security scanning with Trivy
- ✅ Multi-environment deployment (dev, staging, production)
- ✅ Automated testing
- ✅ Manual approval for production
- ✅ Rollout verification

**Workflow stages:**
1. Validate - Check YAML syntax and Kustomize
2. Security scan - Scan for vulnerabilities
3. Deploy to dev - Auto-deploy on develop branch
4. Deploy to staging - Auto-deploy on main branch
5. Deploy to production - Manual approval required

---

## 📊 Feature Comparison

| Feature | Simple EKS | Terraform EKS | Multi-Env | Serverless |
|---------|-----------|---------------|-----------|------------|
| Difficulty | ⭐ Easy | ⭐⭐ Medium | ⭐⭐⭐ Advanced | ⭐⭐ Medium |
| Time to Deploy | 30-40 min | 45-60 min | 15-20 min | 10-15 min |
| Cost (monthly) | $150-200 | $200-300 | $200-300 | $5-50+ |
| IaC Tool | eksctl | Terraform | kubectl | Terraform/SAM |
| Production Ready | No | Yes | Yes | Yes |
| Multi-Env Support | No | Limited | Yes | Yes |
| Auto-scaling | No | Optional | Yes | Built-in |
| Best Use Case | Learning | Production | Enterprise | Microservices |

---

## 🎯 Learning Path (เส้นทางการเรียนรู้)

### Beginner (ผู้เริ่มต้น)
1. Read [README.md](README.md) - Understand GitOps concepts
2. Follow [GETTING_STARTED.md](GETTING_STARTED.md) - Set up first cluster
3. Try [Simple EKS example](examples/simple-eks-gitops/) - Deploy first app
4. Explore ArgoCD UI - Understand GitOps in action

**Time:** 2-3 hours
**Cost:** ~$5-10 (run for few hours then delete)

### Intermediate (ระดับกลาง)
1. Study [Terraform EKS example](examples/terraform-eks-argocd/) - Learn IaC
2. Implement [Multi-environment setup](examples/multi-env-gitops/) - Real-world pattern
3. Review [BEST_PRACTICES.md](BEST_PRACTICES.md) - Production guidelines
4. Set up CI/CD pipeline - Automate deployments

**Time:** 1-2 days
**Cost:** ~$20-40 (run for a day then delete)

### Advanced (ระดับสูง)
1. Customize for your use case - Adapt to your needs
2. Implement monitoring and alerting - Production observability
3. Set up disaster recovery - Business continuity
4. Optimize costs - Reduce AWS bills
5. Contribute improvements - Give back to community

**Time:** Ongoing
**Cost:** Varies by scale

---

## 💡 Common Use Cases

### 1. Startup / Small Team
- **Use:** Simple EKS or Serverless
- **Why:** Quick setup, low learning curve
- **Cost:** $50-200/month

### 2. Growing Company
- **Use:** Terraform EKS + Multi-env
- **Why:** Scalable, repeatable, professional
- **Cost:** $300-1000/month

### 3. Enterprise
- **Use:** Multi-env + Full CI/CD + Monitoring
- **Why:** Multiple teams, compliance, governance
- **Cost:** $1000+/month

### 4. Microservices
- **Use:** Combination of EKS and Serverless
- **Why:** Right tool for each service
- **Cost:** Variable

---

## 🛠️ Tools Covered

### Infrastructure
- ✅ **AWS EKS** - Managed Kubernetes
- ✅ **AWS Lambda** - Serverless compute
- ✅ **Amazon VPC** - Networking
- ✅ **Terraform** - Infrastructure as Code
- ✅ **AWS CloudFormation/SAM** - AWS-native IaC

### GitOps & CD
- ✅ **ArgoCD** - Declarative GitOps for Kubernetes
- ✅ **Kustomize** - Kubernetes native configuration management
- ✅ **GitHub Actions** - CI/CD automation

### Monitoring & Security
- ✅ **Prometheus** - Metrics collection
- ✅ **CloudWatch** - AWS native monitoring
- ✅ **Trivy** - Security scanning
- ✅ **AWS IAM** - Identity and access management

---

## 📞 Getting Help

### Self-Service Resources
1. Check example READMEs - Detailed instructions
2. Review troubleshooting sections - Common issues solved
3. Read AWS documentation - Official resources
4. Search GitHub issues - Community Q&A

### Community Support
- Open an issue in this repository
- Join AWS community forums
- Check ArgoCD community
- Thai developer communities on Facebook/Discord

---

## 🎓 What You'll Learn

After completing this tutorial, you will:

✅ Understand GitOps principles and benefits
✅ Deploy and manage EKS clusters
✅ Implement Infrastructure as Code with Terraform
✅ Set up automated deployments with ArgoCD
✅ Manage multiple environments effectively
✅ Implement CI/CD pipelines
✅ Apply security best practices
✅ Monitor and troubleshoot applications
✅ Optimize AWS costs
✅ Design for disaster recovery

**ภาษาไทย:** หลังจากเรียนจบคุณจะสามารถจัดการ infrastructure บน AWS แบบ GitOps ได้อย่างมืออาชีพ

---

## 📈 Next Steps

1. **Start Learning**: Pick an example and follow it
2. **Experiment**: Modify configurations, try changes
3. **Build Real Projects**: Apply to your work
4. **Share Knowledge**: Teach others, contribute back
5. **Stay Updated**: Star this repo for updates

---

## 🌟 Why GitOps?

### Traditional Deployment
```
Developer → Manual kubectl → Production ❌
```
- Error-prone
- No audit trail
- Hard to rollback
- Team bottlenecks

### GitOps Deployment
```
Developer → Git PR → Review → Merge → Auto Deploy ✅
```
- Automated
- Version controlled
- Easy rollback
- Scalable process

---

## 📝 Summary

This tutorial provides:
- ✅ 4 complete working examples
- ✅ 35+ configuration files
- ✅ Bilingual documentation (English/Thai)
- ✅ Production-ready patterns
- ✅ Step-by-step instructions
- ✅ Best practices guide
- ✅ CI/CD pipelines
- ✅ Troubleshooting help

**Ready to start? Head to [GETTING_STARTED.md](GETTING_STARTED.md)!**

**ภาษาไทย:** พร้อมเริ่มต้นแล้วใช่ไหม? ไปที่ [GETTING_STARTED.md](GETTING_STARTED.md) เลย!

---

**Made with ❤️ for the DevOps community | สร้างด้วยความตั้งใจเพื่อชุมชน DevOps**
