# Product Thinking - jeppesen-eks

## Problem Statement
Enterprise teams need production-ready EKS clusters with standardized configurations, security controls, and support for multiple OS options (Amazon Linux 2 and Bottlerocket) while maintaining operational simplicity.

## User Story
As a platform engineer, I need to provision secure, scalable EKS clusters with minimal configuration that support both Amazon Linux 2 and Bottlerocket OS, so that development teams can deploy applications consistently across environments.

## Success Metrics
- Deployment time: < 15 minutes for standard cluster
- Zero-touch worker node updates with rolling deployment
- 100% compatibility with existing workloads
- Support for both Amazon Linux 2 and Bottlerocket OS

## Scope & Constraints
**In Scope:**
- EKS cluster provisioning with node groups
- Bottlerocket OS support 
- Automatic worker node image updates
- Security controls and compliance
- Documentation and examples

**Constraints:**
- Must maintain backward compatibility
- AWS EKS service limitations
- Cost optimization requirements

## Dependencies
- AWS EKS service
- Terraform/OpenTofu
- Integration with platform GitOps workflow