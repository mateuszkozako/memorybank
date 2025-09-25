# Copilot Rules

## 🚨 Security & Secrets
- **Never Upload Secrets**: Do not store API keys, tokens, or `.env` files in repo
- **Use Placeholders**: Create `.env.example` with safe placeholder values
- **Secret Leaks**: If leaked, immediately rotate credentials, purge history, notify team
- **Terraform Variables**: Use TF_VAR_ environment variables, never hardcode sensitive values

## 🏗️ Infrastructure Standards
- **GitOps First**: All infrastructure changes via Git commits and automated pipelines
- **Environment Parity**: Dev environments must mirror production configurations
- **Backwards Compatibility**: Maintain compatibility when updating modules
- **Documentation Required**: Every module change requires updated README and variables documentation

## 🔄 Pipeline Best Practices
- **Optional Dependencies**: Use `needs:optional` for conditional job dependencies
- **Variable Alignment**: Ensure GitLab CI TF_VAR variables match terraform.tfvars exactly
- **Environment Separation**: Maintain distinct dev/prod variable sets
- **Test Coverage**: All infrastructure changes require validation tests

## 📝 Code Quality
- **Descriptive Variables**: All Terraform variables need clear descriptions with examples
- **Breaking Changes**: Document all breaking changes with migration guidance
- **Consistent Formatting**: Use terraform fmt and consistent naming conventions

## 🔍 Terraform Validation Process
- **Always Run Validation**: Execute `terraform validate` before any plan/apply operations
- **Required Variables**: Ensure all non-default variables are defined in terraform.tfvars
- **Variable Naming**: Use consistent, descriptive variable names - avoid deprecated patterns
- **OS Configuration**: Use `os_image = "BOTTLEROCKET"` or `os_image = "default"` (maps to AL2)
- **SSM Parameter Safety**: Use `local.resolved_os_image` to avoid indexing empty data source tuples
- **Module Input Validation**: Pass resolved values to prevent null variable errors in submodules
- **Test Configuration**: Validate test tfvars files have all required variables (aws_account_id, cluster_name, default_tags)

## 🐳 Image Builder Best Practices  
- **Bottlerocket Constraints**: Keep OS modifications minimal due to OSTree immutability
- **Helper Scripts**: Place utilities in writable paths like `/opt` rather than modifying core OS
- **Device Mounting**: Use privileged containers or first-boot scripts for EBS device formatting
- **NVMe Considerations**: Account for device name mapping differences across instance types
