# Context - ocm-client

## Repository Information
- **Location**: `/home/hypeit/platform/ocm-client`
- **Type**: OAuth Client Management Service
- **Current Status**: Enhanced Testing & Production Ready
- **Current Branch**: `module`

## Latest Updates (17.09.2025) - Major Testing Enhancements

### Recent Activity - 10 Critical Commits
**Focus**: Testing robustness, error diagnostics, and deployment improvements

#### Enhanced Error Handling & Diagnostics
- **e7b41ed** - [tf-test] Guard plan against missing ECR image
- **d8429d2** - [tf-test] Share minimal payload log in onboarding
- **999a81e** - [tf-test] Add artifact cleanup guidance
- **bb536ca** - [tf-test] Log minimal payload validation artifacts
- **ae2f794** - [tf-tests] Add 4xx diagnostics; set default timeout=30s in module
- **1f40e00** - [tf-tests] Add response headers and body length logging for 4xx diagnostics

#### Deployment & Configuration
- **dd22737** - fix: add development environment variables for OCM client deployment
- **4062462** - [tf-test] Pre-check existence; avoid retries on 4xx; force name=clientId in payload
- **4620a9e** - [tf-tests] Add request/response debug logging; align sample client inventory with template defaults

### Key Improvements Implemented
- **4xx Response Diagnostics**: Comprehensive error logging with headers and body length
- **Debug Logging**: Full request/response logging for troubleshooting
- **Timeout Management**: Configurable 30s default timeout for reliability
- **Pre-check Logic**: Existence validation to avoid unnecessary API retries
- **Development Environment**: Enhanced variable support for deployment
- **Payload Optimization**: Force name=clientId, align with template defaults
- **Artifact Management**: Validation logging and cleanup guidance

## Previous Infrastructure Fixes (Completed)
- Fixed GitLab CI TF_VAR variable mismatches
- Updated from deprecated image_uri to image_tag
- Added optional job dependencies for conditional builds
- Enhanced handler with restrictToDefaultAccessTokenManager parameter
- 5 commits focused on pipeline reliability

## Key Files & Structure
```
src/
├── main.tf                 # Terraform infrastructure (enhanced with ECR protection)
├── variables.tf           # Input variables (updated with dev environment support)
├── terraform.tfvars      # Variable values (aligned with CI pipeline)
├── backend-dev.hcl       # Dev backend config
├── backend-prod.hcl      # Prod backend config
└── lambda/
    ├── handler.py         # Main Lambda function (enhanced error handling)
    ├── requirements.txt   # Python dependencies
    ├── Dockerfile        # Container definition
    └── test_handler.py   # Unit tests (enhanced diagnostics)

.gitlab-ci.yml            # CI/CD pipeline (optimized with latest fixes)
examples/
├── client-inventory.yaml # Example configurations (aligned with defaults)
docs/                     # Documentation
az_automation/           # Azure integration scripts
tests/                   # Integration tests (enhanced with 4xx diagnostics)
```

## Dependencies & Integrations
- **PingFederate API**: OAuth server integration (enhanced error handling)
- **AWS Lambda**: Serverless execution (improved timeout management)
- **AWS Secrets Manager**: Credential storage
- **DynamoDB**: State management
- **GitLab CI/CD**: Deployment automation (optimized pipeline)
- **Docker Registry**: Container storage (ECR image protection)
- **Azure AD**: Authentication integration