# Platform Architecture Guide

## Platform Overview
This is a **cloud infrastructure platform** built around:
- **EKS Module (`jeppesen-eks/`)**: Production-ready Kubernetes clusters with 20+ optional components (Istio, ADOT, Karpenter, etc.)
- **OCM Client (`ocm-client/`)**: OAuth Client Management with Azure Functions + AWS Secrets Manager integration
- **Test Infrastructure (`digx-test-infra/`)**: FluxCD + Terraform manifests for automated deployment validation
- **Integration Test Module (`integration-test-module/`)**: Automated testing framework for infrastructure validation

## Key Development Patterns

### 1. Infrastructure as Code (IaC) Workflow
- **Terraform modules** define infrastructure components
- **FluxCD Terraform CRDs** deploy via GitOps in manifests like:
  ```yaml
  apiVersion: infra.contrib.fluxcd.io/v1alpha2
  kind: Terraform
  metadata:
    name: cluster-name
  spec:
    sourceRef:
      name: jeppesen-eks
    vars:
      - name: cluster_name
        value: "my-cluster"
  ```
- **Git references** use branches/tags for version control in manifests

### 2. EKS Module Architecture (`jeppesen-eks/src/`)
- **Modular design**: Each feature in separate modules (`modules/istio/`, `modules/adot/`, etc.)
- **Feature toggles**: Enable components via boolean variables (`enable_istio`, `enable_karpenter`)
- **Breaking changes**: Major versions require careful migration (see README upgrade guides)
- **OS selection**: AL2 vs Bottlerocket via `os_image` variable
- **Version constraints**: Kubernetes upgrades must be sequential (1.28→1.29→1.30)

### 3. Testing Strategy
- **Integration tests**: Go-based terratest in `jeppesen-eks/test/`
- **Test manifests**: Live validation using `digx-test-infra/` YAML files
- **Feature branches**: All changes go through `alpha` branch with automated testing
- **Manual validation**: Use test accounts (`digx-uat`) before production

### 4. Security & Compliance
- **🔒 NEVER** commit secrets - use placeholders in `.env.example` files
- **Scope-based access**: OAuth client management with fine-grained permissions
- **AWS Secrets Manager**: Secure credential storage for production workloads
- **IRSA patterns**: Service accounts use IAM roles rather than access keys

## Integration Points & Dependencies

### External Systems
- **GitLab EXT**: Source code and CI/CD (gitlab-ext.digitalaviationservices.com)
- **AWS Accounts**: Multi-account strategy (dev, uat, prod)
- **Azure Active Directory**: OAuth provider integration
- **Backstage**: Service catalog and developer portal

### Cross-Component Communication
- **VPC sharing**: EKS clusters reference VPC by name
- **Service meshes**: Istio for inter-service communication
- **Observability**: ADOT for metrics/logs to AWS services
- **GitOps**: ArgoCD for application deployment

## Component Details

### EKS Module (`jeppesen-eks/`)
Provides production-ready Kubernetes clusters with:
- **Core components**: EKS cluster, managed node groups, networking
- **Optional features**: Istio service mesh, ADOT observability, Karpenter autoscaling
- **Security integrations**: IRSA, pod security standards, network policies
- **Monitoring stack**: Prometheus, Grafana, CloudWatch integration

### OCM Client (`ocm-client/`)
OAuth Client Management system featuring:
- **Azure Functions**: Serverless API endpoints for client operations
- **AWS Secrets Manager**: Secure credential storage and rotation
- **Multi-environment**: Dev, UAT, and production configurations
- **Automated testing**: Comprehensive integration test suite

### Test Infrastructure (`digx-test-infra/`)
GitOps deployment manifests for:
- **FluxCD sources**: Git repositories and Helm charts
- **Terraform resources**: Infrastructure as code deployments
- **Test environments**: Validation clusters and workloads
- **CI/CD integration**: Automated deployment pipelines

When working on this platform, prioritize understanding the **GitOps workflow** and **module composition patterns** over individual file edits. Changes typically involve updating variables in deployment manifests rather than direct infrastructure code modifications.