# System Patterns

## GitOps Infrastructure Pattern

### Core Workflow
1. **Git Repository**: Contains FluxCD manifests with Terraform CRDs
2. **FluxCD Controller**: Watches git changes and applies Terraform
3. **Terraform Execution**: Provisions AWS resources via modules
4. **State Management**: Terraform state stored in S3 backends

### Standard Manifest Structure
```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: jeppesen-eks-source
spec:
  interval: 8760h
  url: https://gitlab-ext.digitalaviationservices.com/das-platform/terraform-modules/aws/jeppesen-eks.git
  ref:
    tag: v5.0.0  # Use tags for stable, branches for testing
---
apiVersion: infra.contrib.fluxcd.io/v1alpha2
kind: Terraform
metadata:
  name: my-cluster
spec:
  sourceRef:
    name: jeppesen-eks-source
  vars:
    - name: cluster_name
      value: "my-cluster"
    - name: enable_istio
      value: true
```

## EKS Module Architecture

### Modular Design
- **Core Module**: `jeppesen-eks/src/main.tf` - Base EKS cluster
- **Feature Modules**: `src/modules/*/` - Optional components
- **Feature Toggles**: Boolean variables (`enable_*`) control component installation

### Module Hierarchy
```
jeppesen-eks/src/
├── main.tf              # Core EKS cluster
├── modules/
│   ├── istio/           # Service mesh
│   ├── adot/            # Observability
│   ├── karpenter/       # Node autoscaling
│   ├── argocd/          # Application GitOps
│   └── [17+ more]/
```

### Variable Patterns
- **Required**: `cluster_name`, `vpc_name`, `region`, `default_tags`
- **Feature Toggles**: `enable_istio`, `enable_karpenter`, etc.
- **Configuration**: Component-specific variables (`istio_allowed_ccids`)

## Security Patterns

### IRSA (IAM Roles for Service Accounts)
- Service accounts use IAM roles, not access keys
- Each AWS service integration gets dedicated IRSA role
- Principle of least privilege per component

### Secret Management
- **Never commit secrets** to git repositories
- Use `.env.example` with placeholder values
- Production secrets in AWS Secrets Manager
- OCM client stores OAuth credentials securely

### Network Security
- **VPC Isolation**: Each cluster in dedicated VPC or secondary subnet
- **Service Mesh**: Istio for encrypted inter-service communication
- **Authorization Policies**: CCID-based access control in Istio

## Testing Patterns

### Multi-Layer Testing
1. **Unit Tests**: Individual module validation (`terraform plan`)
2. **Integration Tests**: Go + Terratest (`jeppesen-eks/test/`)
3. **Live Validation**: Real deployments in `digx-test-infra/`
4. **API Tests**: Python-based OCM client testing

### Test Environment Flow
```
feature-branch → alpha-branch → digx-uat → production
```

### Common Test Scenarios
- **Alpha EKS Test**: Basic cluster provisioning
- **Branch EKS Test**: Feature branch validation  
- **Destroy Test**: Cleanup validation
- **Upgrade Test**: Version migration testing

## Debugging Patterns

### FluxCD Troubleshooting
```bash
# Check reconciliation status
kubectl get terraform -A
flux get sources git
flux get terraforms

# Suspend problematic deployments
kubectl patch terraform cluster-name --type json \
  --patch='[{"op": "add", "path": "/spec/suspend", "value": true}]'
```

### EKS Access Patterns
```bash
# Update kubeconfig
aws eks update-kubeconfig --name cluster-name --region region

# Check node status
kubectl get nodes -o wide

# Verify Istio service mesh
kubectl get pods -n istio-system
```

### Terraform State Inspection
```bash
# List resources
terraform state list

# Show specific resource
terraform show 'module.eks.aws_eks_cluster.cluster'

# Import existing resources if needed
terraform import 'resource.name' 'resource-id'
```

## Integration Patterns

### Cross-Component Dependencies
- **VPC → EKS**: Clusters reference VPC by name
- **EKS → Applications**: ArgoCD manages app deployment post-cluster
- **Istio → External Services**: Service mesh controls egress traffic
- **ADOT → AWS Services**: Observability data flows to CloudWatch/X-Ray

### External System Integration
- **GitLab EXT**: Source code and CI/CD pipelines
- **Backstage**: Service catalog and developer portal
- **AWS Accounts**: Multi-account deployment strategy
- **Azure AD**: OAuth provider for authentication