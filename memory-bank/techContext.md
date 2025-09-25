# Tech Context

## Core Technologies

### Infr### Architecture Decisions

### OS Selection & Image Builder
- **AL2**: Traditional Linux with full tooling access
- **BOTTLEROCKET**: Container-optimized, immutable OS with minimal attack surface
- **Default**: Maps to AL2 but forces launch template updates for deterministic rollback behavior
- **Custom AMIs**: EC2 Image Builder pipeline for layered OS modifications with component-based architecturecture as Code
- **Terraform**: Primary IaC tool for AWS resource provisioning
- **FluxCD**: GitOps operator for automated deployment via Terraform CRDs
- **Kubernetes**: Container orchestration (EKS focus)

### EKS Module Stack (`jeppesen-eks/`)
- **Base**: EKS cluster + managed node groups
- **Optional Components** (20+ modules):
  - **Istio**: Service mesh with authorization policies
  - **ADOT**: AWS Distro for OpenTelemetry observability
  - **Karpenter**: Just-in-time node autoscaling
  - **ArgoCD**: Application deployment GitOps
  - **Velero**: Backup and disaster recovery
  - **External DNS**: Route53 integration
  - **AWS Load Balancer Controller**: ALB/NLB management

### OCM Client Stack (`ocm-client/`)
- **Azure Functions**: Serverless OAuth client management
- **Python 3.8+**: Runtime and testing framework
- **AWS Secrets Manager**: Secure credential storage
- **Azure Active Directory**: OAuth provider integration

### Testing Infrastructure
- **Go + Terratest**: Infrastructure integration testing
- **Python**: OCM client API testing
- **FluxCD manifests**: Live deployment validation

## Development Environment

### Required Tools
- **Terraform** (latest)
- **kubectl** + AWS CLI
- **Go 1.19+** (for EKS tests)
- **Python 3.8+** (for OCM tests)
- **flux CLI** (for GitOps debugging)

### Key Constraints
- **Sequential K8s upgrades**: Must go 1.28→1.29→1.30 (no skipping)
- **Karpenter migration**: Cannot upgrade in-place (requires manual CRD cleanup)
- **EFS + minimal nodes**: Requires minimum 2 nodes for EFS driver
- **Git reference naming**: EKS sources must start with `jeppesen-eks`

## Architecture Decisions

### OS Selection
- **AL2**: Traditional Linux with full tooling access
- **Bottlerocket**: Container-optimized, security-focused (recommended for production)

### Multi-Account Strategy
- **Dev**: `digx-dev` - Development and feature testing
- **UAT**: `digx-uat` - Integration testing before production
- **Prod**: Multiple production accounts per region

### Version Control
- **Feature branches**: All changes through `alpha` branch first
- **Git references**: Use tags for stable releases, branches for testing
- **Breaking changes**: Major version bumps require migration guides