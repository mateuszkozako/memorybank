# File Patterns Reference

## Configuration Files

### Terraform Files
- **`main.tf`**: Primary resource definitions and module calls
- **`variables.tf`**: Input parameters with validation rules and descriptions
- **`outputs.tf`**: Resource outputs for chaining and external references
- **`versions.tf`**: Provider version constraints and required providers
- **`data.tf`**: Data source lookups for existing resources
- **`locals.tf`**: Local value calculations and transformations

### GitOps and Deployment
- **`catalog-info.yaml`**: Backstage service catalog entries for documentation
- **`renovate.json`**: Dependency update automation configuration
- **GitRepository CRDs**: FluxCD source definitions for Git repositories
- **Terraform CRDs**: FluxCD deployment manifests for infrastructure

### Examples and Documentation
- **`README.md`**: Module documentation with usage examples
- **`examples/*.tfvars`**: Example configurations for different use cases
- **`CHANGELOG.md`**: Version history and breaking changes
- **`.gitignore`**: Git exclusion patterns for sensitive files

## Module Structure Patterns

### Root Module (`jeppesen-eks/src/`)
```
src/
├── main.tf              # Main EKS cluster resources
├── variables.tf         # Input variables with validation
├── outputs.tf           # Cluster outputs (endpoint, cert, etc.)
├── versions.tf          # Provider requirements
├── data.tf             # AWS data sources
├── locals.tf           # Local calculations
├── modules/            # Feature modules
│   ├── istio/
│   ├── adot/
│   ├── karpenter/
│   └── ...
├── eni-config/         # Custom networking configs
├── files/              # Static configuration files
└── namespace-roles/    # RBAC configurations
```

### Feature Modules (`modules/*/`)
```
modules/feature-name/
├── main.tf            # Feature-specific resources
├── variables.tf       # Feature configuration options
├── outputs.tf         # Feature outputs
├── versions.tf        # Feature provider requirements
├── README.md          # Feature documentation
└── examples/          # Feature usage examples
    └── basic.tfvars
```

### Test Structure (`test/`)
```
test/
├── *_test.go          # Go-based integration tests
├── go.mod             # Go module dependencies
├── go.sum             # Go dependency checksums
├── terraform.tfvars   # Test configuration
├── README.md          # Testing documentation
└── kyverno-policy/    # Policy validation tests
```

## Testing Patterns

### Go-based Infrastructure Tests
- **`alpha_eks_test.go`**: Tests against alpha environment
- **`branch_eks_test.go`**: Feature branch validation
- **`destroy_eks_test.go`**: Cleanup and destruction tests
- **`main_eks_test.go`**: Core functionality tests
- **`pull_helm_from_ecr_test.go`**: Helm chart validation

### Python-based API Tests
- **`test_*.py`**: Python-based API/integration tests
- **`conftest.py`**: Pytest configuration and fixtures
- **`requirements.txt`**: Python test dependencies

### Example Configurations
- **`examples/basic.tfvars`**: Minimal working configuration
- **`examples/bottlerocket.tfvars`**: Bottlerocket node groups
- **`examples/istio.tfvars`**: Service mesh enabled
- **`examples/production.tfvars`**: Production-ready setup

## FluxCD Deployment Manifests

### GitRepository Sources
```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: jeppesen-eks
  namespace: flux-system
spec:
  url: https://gitlab-ext.digitalaviationservices.com/das-platform/jeppesen-eks
  ref:
    branch: main
  interval: 5m
```

### Terraform Resources
```yaml
apiVersion: infra.contrib.fluxcd.io/v1alpha2
kind: Terraform
metadata:
  name: cluster-name
  namespace: flux-system
spec:
  sourceRef:
    kind: GitRepository
    name: jeppesen-eks
  path: "./src"
  interval: 10m
  vars:
    - name: cluster_name
      value: "my-cluster"
    - name: kubernetes_version
      value: "1.30"
```

## OCM Client Structure

### Azure Functions
```
ocm-client/
├── src/                    # Function app source code
│   ├── function_app.py     # Main application entry
│   ├── shared/             # Shared utilities
│   └── functions/          # Individual function definitions
├── tests/                  # Test suites
│   ├── unit/              # Unit tests
│   ├── integration/       # Integration tests
│   └── fixtures/          # Test data
├── tools/                 # Development tools
├── scripts/               # Automation scripts
└── docs/                  # Documentation
```

### Configuration Files
- **`host.json`**: Azure Functions runtime configuration
- **`requirements.txt`**: Python dependencies
- **`local.settings.json.example`**: Local development settings template
- **`.env.example`**: Environment variable template

## Integration Test Module

### Test Runner Structure
```
integration-test-module/
├── src/                           # Test framework source
│   ├── scripts/                   # Automation scripts
│   └── tofu_ctrl_claim_checker/   # Terraform controller validation
├── tests/                         # Test definitions
│   ├── ciam-apigw/               # API Gateway tests
│   └── ciamg-alb/                # Load balancer tests
├── integration-tests-apps/        # Test applications
├── integration-tests-claims/      # Resource claims
└── integration-test-runner-role/  # IAM roles and permissions
```

## Common File Recognition Patterns

### Infrastructure as Code
- **`.tf` extensions**: Terraform configuration files
- **`.tfvars` extensions**: Terraform variable definitions
- **`.yaml/.yml` with `apiVersion`**: Kubernetes/FluxCD manifests
- **`terragrunt.hcl`**: Terragrunt configuration (if used)

### Application Code
- **`function_app.py`**: Azure Functions entry point
- **`requirements.txt`**: Python dependencies
- **`go.mod`**: Go module definition
- **`package.json`**: Node.js dependencies (if applicable)

### Documentation and Configuration
- **`README.md`**: Primary documentation
- **`CHANGELOG.md`**: Version history
- **`renovate.json`**: Dependency management
- **`catalog-info.yaml`**: Backstage service catalog

### Security and Credentials
- **`.env.example`**: Environment variable templates
- **`local.settings.json.example`**: Local configuration templates
- **Never commit**: `.env`, `local.settings.json`, or any actual credentials

## Directory Naming Conventions

### Project Structure
- **`src/`**: Source code and primary modules
- **`test/` or `tests/`**: Test suites and validation
- **`examples/`**: Usage examples and sample configurations
- **`docs/`**: Detailed documentation
- **`scripts/` or `tools/`**: Automation and utility scripts

### Module Organization
- **`modules/feature-name/`**: Kebab-case for feature modules
- **`namespace-roles/`**: Hyphenated compound names
- **`eni-config/`**: Short, descriptive directory names

### FluxCD Naming
- **GitRepository names**: Match repository names (e.g., `jeppesen-eks`)
- **Terraform resource names**: Descriptive cluster or resource names
- **Namespace consistency**: Use `flux-system` for GitOps resources