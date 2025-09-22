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
