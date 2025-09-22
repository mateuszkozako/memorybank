# Context - ocm-client

## Repository Information
- **Location**: `/home/hypeit/platform/ocm-client`
- **Type**: OAuth Client Management Service
- **Current Status**: CI/CD Optimization Complete
- **Current Branch**: `module`

## Recent Activity
- Fixed GitLab CI TF_VAR variable mismatches
- Updated from deprecated image_uri to image_tag
- Added optional job dependencies for conditional builds
- Enhanced handler with restrictToDefaultAccessTokenManager parameter
- 5 commits focused on pipeline reliability

## Key Files & Structure
```
src/
├── main.tf                 # Terraform infrastructure
├── variables.tf           # Input variables
├── terraform.tfvars      # Variable values
├── backend-dev.hcl       # Dev backend config
├── backend-prod.hcl      # Prod backend config
└── lambda/
    ├── handler.py         # Main Lambda function
    ├── requirements.txt   # Python dependencies
    ├── Dockerfile        # Container definition
    └── test_handler.py   # Unit tests

.gitlab-ci.yml            # CI/CD pipeline
examples/
├── client-inventory.yaml # Example configurations
docs/                     # Documentation
az_automation/           # Azure integration scripts
tests/                   # Integration tests
```

## Dependencies & Integrations
- **PingFederate API**: OAuth server integration
- **AWS Lambda**: Serverless execution
- **AWS Secrets Manager**: Credential storage
- **DynamoDB**: State management
- **GitLab CI/CD**: Deployment automation
- **Docker Registry**: Container storage
- **Azure AD**: Authentication integration