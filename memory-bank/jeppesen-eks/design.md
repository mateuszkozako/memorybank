# Design Document - jeppesen-eks

## Overview & Goals
Production-ready EKS module that provides standardized Kubernetes clusters with multiple OS support, automated updates, and enterprise security controls.

## Architecture
```mermaid
graph TB
    A[Terraform Module] --> B[EKS Control Plane]
    A --> C[Node Groups]
    C --> D[Amazon Linux 2 Nodes]
    C --> E[Bottlerocket Nodes]
    B --> F[Security Groups]
    B --> G[IAM Roles]
    A --> H[Add-ons]
    H --> I[VPC CNI]
    H --> J[CoreDNS]
    H --> K[kube-proxy]
```

## Tech Stack & Decisions
- **Terraform/OpenTofu**: Infrastructure as Code
- **AWS EKS**: Managed Kubernetes service
- **Amazon Linux 2**: Default worker node OS
- **Bottlerocket**: Container-optimized OS option
- **Managed Node Groups**: AWS-managed worker nodes

## Data Models / APIs
- EKS cluster configuration via Terraform variables
- Node group specifications with OS selection
- Security group and IAM role definitions
- Add-on configurations

## Non-functional Requirements
- **Security**: Pod security standards, network policies
- **Scalability**: Auto-scaling node groups
- **Reliability**: Multi-AZ deployment, health checks
- **Maintainability**: Automated updates, clear documentation