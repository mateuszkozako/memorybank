# Project Brief

## Overview
DAS Platform Infrastructure is a comprehensive cloud infrastructure platform that provides production-ready Kubernetes clusters and supporting services for enterprise applications.

## Core Goals
- **Standardized EKS Clusters**: Deploy consistent, production-ready Kubernetes environments with 20+ optional components
- **OAuth Client Management**: Secure client credential management via Azure Functions + AWS Secrets Manager
- **GitOps Automation**: Infrastructure provisioning through FluxCD + Terraform patterns
- **Compliance & Security**: Enterprise-grade security controls and audit capabilities

## Project Scope
### In Scope
- EKS cluster provisioning with modular components (Istio, ADOT, Karpenter, etc.)
- OAuth client lifecycle management (OCM Client)
- Automated testing infrastructure for infrastructure validation
- Cross-cloud integration (AWS + Azure) for authentication

### Out of Scope
- Application deployment (handled by ArgoCD post-cluster creation)
- Custom application code (platform provides infrastructure only)
- Manual infrastructure management (everything through GitOps)

## Success Criteria
- Teams can deploy production EKS clusters via YAML manifests
- Zero-touch infrastructure provisioning through git commits
- Automated testing validates all infrastructure changes
- OAuth clients managed securely without manual credential handling