# EKS Module Context

## Module Architecture Deep Dive

### Core Structure (`jeppesen-eks/src/`)
```
src/
├── main.tf              # Core EKS cluster definition
├── variables.tf         # Input parameters (30+ variables)
├── outputs.tf           # Cluster outputs for chaining
├── data.tf              # Data sources (VPC, subnets, AMIs)
├── versions.tf          # Provider constraints
└── modules/             # 20+ optional components
    ├── istio/           # Service mesh + authorization policies
    ├── adot/            # AWS Distro for OpenTelemetry
    ├── karpenter/       # Just-in-time node autoscaling
    ├── argocd/          # Application GitOps
    ├── velero/          # Backup and disaster recovery
    ├── lb-controller/   # AWS Load Balancer Controller
    ├── external-dns/    # Route53 integration
    ├── secrets-driver/  # AWS Secrets Manager CSI
    ├── ebs-driver/      # EBS CSI driver
    ├── efs-driver/      # EFS CSI driver
    └── [10+ more]/
```

### Feature Toggle Patterns
All optional components follow this pattern:
```hcl
variable "enable_istio" {
  description = "Set to true to install Istio service mesh"
  type        = bool
  default     = true
}

# In main.tf
module "istio" {
  count  = var.enable_istio ? 1 : 0
  source = "./modules/istio"
  # ... configuration
}
```

### Critical Variables

#### Required Variables
- `cluster_name`: Unique identifier (max 23 chars, alphanumeric + hyphens)
- `cluster_version`: Kubernetes version (1.28-1.32 supported)
- `vpc_name`: VPC where cluster will be deployed
- `region`: AWS region (us-west-2, us-east-1, eu-west-1, eu-central-1, etc.)
- `default_tags`: Must include `esatsId` for billing

#### Key Optional Variables
- `os_image`: "AL2" or "BOTTLEROCKET" (worker node OS)
- `node_instance_type`: Worker node size (default: t3a.xlarge)
- `use_secondary_subnet`: Use secondary CIDR for more pod IPs (default: true)
- `enable_*`: Feature toggles for each component

### Version Upgrade Constraints

#### Sequential Upgrades Required
- Current: 1.28 → Target: 1.30
- Must go: 1.28 → 1.29 → 1.30 (cannot skip versions)
- Control plane and worker nodes must stay within 2 minor versions

#### Breaking Changes by Version
- **v2.0.0**: Region must be explicit, secondary subnet default
- **v3.0.0**: Crossplane components removed
- **v4.0.0**: Karpenter 1.1.1 (requires CRD cleanup)
- **v5.0.0**: Istio gateway namespace changes

### Operating System Selection

#### Amazon Linux 2 (AL2)
- Traditional Linux with package management
- SSH access for debugging
- Bash-based bootstrap scripts
- Root volume: `/dev/xvda`
- Use case: Development, debugging, traditional tooling

#### Bottlerocket
- Container-optimized, minimal OS
- Read-only root filesystem
- No SSH (use SSM Session Manager)
- TOML-based configuration
- Root volume: `/dev/xvdb`
- Use case: Production, security-focused environments

### Testing Infrastructure

#### Go-based Terratest (`test/`)
```go
// test/alpha_eks_test.go
func TestAlphaEKS(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../src",
        VarFiles:     []string{"terraform.tfvars"},
    }
    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)
    // Validation logic...
}
```

#### Test Categories
- `alpha_eks_test.go`: Basic cluster functionality
- `branch_eks_test.go`: Feature branch validation
- `destroy_eks_test.go`: Cleanup verification
- `main_eks_test.go`: Core functionality suite

### Known Issues & Workarounds

#### EKS + EFS Driver
- **Issue**: EFS CSI driver fails with < 2 nodes
- **Solution**: Set `node_min_size` and `node_desired_size` to at least 2

#### Karpenter Migration
- **Issue**: Cannot upgrade Karpenter in-place
- **Workaround**: Manual CRD cleanup before upgrade
```bash
helm uninstall -n karpenter karpenter
kubectl delete ec2nodeclasses.karpenter.k8s.aws -n default default
kubectl delete customresourcedefinitions.apiextensions.k8s.io \
  nodeclaims.karpenter.sh nodepools.karpenter.sh ec2nodeclasses.karpenter.k8s.aws
```

#### Git Source Naming
- **Issue**: Non-`jeppesen-eks` prefixed sources fail with permissions
- **Solution**: Ensure GitRepository names start with `jeppesen-eks`

### Istio Configuration Patterns

#### Authorization Policies
```yaml
# Variable: istio_allowed_ccids
istio_allowed_ccids = [
  {
    namespace = "production-app"
    ccids     = ["TVF", "Boeing_SST"]
  }
]
```

#### Gateway Configuration
```yaml
# Variable: istio_ciamg_alb_hosts
istio_ciamg_alb_hosts = ["https://app.example.com"]
```

### ADOT (Observability) Configuration

#### Namespace Monitoring
```yaml
# Variable: adot_std_namespaces
adot_std_namespaces = ["app-namespace", "monitoring"]

# Variable: adot_namespace_pod_filters
adot_namespace_pod_filters = [
  {
    namespace    = "app-namespace"
    pod_patterns = ["api-*", "worker-*"]
  }
]
```

### Security Patterns

#### IRSA (IAM Roles for Service Accounts)
Each AWS integration gets dedicated IRSA:
- `cluster_name-adot-collector`: ADOT CloudWatch/X-Ray access
- `cluster_name-external-dns`: Route53 management
- `cluster_name-lb-controller`: ALB/NLB management
- `cluster_name-velero`: S3 backup access

#### Network Security
- **VPC isolation**: Clusters use dedicated subnets
- **Security groups**: Minimal required access
- **Istio mTLS**: Service-to-service encryption
- **Authorization policies**: CCID-based access control