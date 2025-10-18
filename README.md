# AWS GitOps Tutorial (บทเรียน AWS GitOps)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 📚 Table of Contents
- [Introduction (แนะนำ)](#introduction)
- [What is GitOps?](#what-is-gitops)
- [Prerequisites (สิ่งที่ต้องเตรียม)](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Getting Started](#getting-started)
- [Examples](#examples)
- [Best Practices](#best-practices)
- [Resources](#resources)

## Introduction

This repository provides a comprehensive tutorial and examples for implementing GitOps practices on AWS infrastructure. You'll learn how to automate your AWS infrastructure deployments using GitOps principles.

**ภาษาไทย:** Repository นี้เป็นคู่มือและตัวอย่างการใช้งาน GitOps บน AWS คุณจะได้เรียนรู้วิธีการจัดการและอัตโนมัติการ deploy infrastructure บน AWS โดยใช้หลักการ GitOps

## What is GitOps?

GitOps is a modern approach to continuous deployment that uses Git as the single source of truth for declarative infrastructure and applications. The core principles include:

1. **Declarative Description**: Everything is described declaratively
2. **Version Control**: All configurations are stored in Git
3. **Automated Deployment**: Changes are automatically applied
4. **Continuous Reconciliation**: System state matches Git state

**ภาษาไทย:** GitOps คือแนวทางการ deploy แบบอัตโนมัติที่ใช้ Git เป็นแหล่งข้อมูลหลักสำหรับ infrastructure และ application โดยมีหลักการสำคัญคือ ทุกอย่างถูกเก็บไว้ใน Git, มีการควบคุมเวอร์ชัน, deploy อัตโนมัติ และระบบจะปรับให้ตรงกับสิ่งที่อยู่ใน Git เสมอ

## Prerequisites

Before you begin, ensure you have:

- **AWS Account**: Active AWS account with appropriate permissions
- **AWS CLI**: Installed and configured ([Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html))
- **kubectl**: Kubernetes command-line tool
- **Terraform** or **AWS CDK**: Infrastructure as Code tools
- **Git**: Version control system
- **Docker**: For containerization (optional)

**ภาษาไทย - สิ่งที่ต้องเตรียม:**
- บัญชี AWS ที่ใช้งานได้
- ติดตั้ง AWS CLI และตั้งค่าแล้ว
- ติดตั้ง kubectl
- ติดตั้ง Terraform หรือ AWS CDK
- Git
- Docker (ถ้าต้องการ)

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        Developer                             │
│                     (Push to Git Repo)                       │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│                    Git Repository                            │
│              (Infrastructure & App Configs)                  │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│                   GitOps Operator                            │
│              (ArgoCD / Flux / Jenkins)                       │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│                      AWS Infrastructure                      │
│         (EKS, EC2, RDS, S3, Lambda, etc.)                   │
└─────────────────────────────────────────────────────────────┘
```

## Getting Started

### Step 1: Set Up AWS Credentials

```bash
# Configure AWS CLI
aws configure

# Verify configuration
aws sts get-caller-identity
```

### Step 2: Clone This Repository

```bash
git clone https://github.com/somkheartk/aws-gitops.git
cd aws-gitops
```

### Step 3: Choose Your Path

This repository includes multiple examples:
- [Terraform + EKS + ArgoCD](./examples/terraform-eks-argocd/)
- [AWS CDK + EKS + Flux](./examples/cdk-eks-flux/)
- [GitHub Actions + ECS](./examples/github-actions-ecs/)

## Examples

### Example 1: Simple EKS Cluster with GitOps

See [examples/simple-eks-gitops/](./examples/simple-eks-gitops/) for a basic example.

### Example 2: Multi-Environment Setup

See [examples/multi-env-gitops/](./examples/multi-env-gitops/) for production-grade example.

### Example 3: Serverless GitOps

See [examples/serverless-gitops/](./examples/serverless-gitops/) for Lambda deployments.

## Best Practices

### 1. Repository Structure
```
aws-gitops/
├── infrastructure/          # Infrastructure as Code
│   ├── terraform/
│   └── cloudformation/
├── kubernetes/             # K8s manifests
│   ├── base/
│   └── overlays/
├── applications/           # Application configs
└── ci-cd/                 # Pipeline definitions
```

### 2. Security Best Practices
- Never commit secrets to Git
- Use AWS Secrets Manager or Parameter Store
- Implement RBAC (Role-Based Access Control)
- Enable audit logging
- Use encrypted Git repositories

### 3. GitOps Workflow
1. Make changes in Git
2. Create Pull Request
3. Review and approve
4. Merge to main branch
5. GitOps operator auto-deploys

**ภาษาไทย - ขั้นตอนการทำงาน:**
1. แก้ไขโค้ดใน Git
2. สร้าง Pull Request
3. รีวิวและอนุมัติ
4. Merge เข้า main branch
5. ระบบจะ deploy อัตโนมัติ

## Resources

### AWS Services for GitOps
- **Amazon EKS**: Managed Kubernetes service
- **AWS CodePipeline**: CI/CD service
- **AWS CodeCommit**: Git repository service
- **Amazon ECR**: Container registry
- **AWS Systems Manager**: Parameter store and secrets

### GitOps Tools
- **ArgoCD**: Declarative GitOps CD for Kubernetes
- **Flux CD**: GitOps toolkit for Kubernetes
- **Jenkins X**: Cloud-native CI/CD for Kubernetes

### Learning Resources
- [AWS EKS Workshop](https://www.eksworkshop.com/)
- [GitOps Working Group](https://opengitops.dev/)
- [CNCF GitOps](https://www.cncf.io/blog/2021/08/11/gitops-is-here-to-stay/)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**🇹🇭 สร้างโดยชุมชน Developer ไทย เพื่อชุมชน Developer ไทย**

For questions or support, please open an issue in this repository.