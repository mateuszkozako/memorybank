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
- **OS Configuration**: Use `os_image = "BOTTLEROCKET"` not legacy `use_bottlerocket = true`
- **Test Configuration**: Validate test tfvars files have all required variables (aws_account_id, cluster_name, default_tags)
