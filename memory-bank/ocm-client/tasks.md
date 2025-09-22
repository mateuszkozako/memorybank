# Task Breakdown - ocm-client

## Task List

### OCM-1: CI/CD Variable Alignment
- **Description**: Fix TF_VAR variables to match terraform.tfvars exactly
- **Acceptance Criteria**: 
  - [x] GitLab CI variables match terraform.tfvars
  - [x] Dev/prod environment separation
  - [x] Remove deprecated variables
- **Effort**: S
- **Files/Modules**: .gitlab-ci.yml, terraform.tfvars

### OCM-2: Image Tag Migration
- **Description**: Update from deprecated image_uri to image_tag variable
- **Acceptance Criteria**: 
  - [x] TF_VAR_image_tag replaces TF_VAR_image_uri
  - [x] Pipeline uses commit SHA as tag
  - [x] Terraform receives correct variable
- **Effort**: S
- **Files/Modules**: .gitlab-ci.yml, variables.tf

### OCM-3: Pipeline Dependencies Fix
- **Description**: Add optional job dependencies for conditional builds
- **Acceptance Criteria**: 
  - [x] Use needs:optional for ECR push job
  - [x] Use needs:optional for validation job
  - [x] Pipeline runs without dependency errors
- **Effort**: S
- **Files/Modules**: .gitlab-ci.yml

### OCM-4: Handler Enhancement
- **Description**: Add restrictToDefaultAccessTokenManager parameter
- **Acceptance Criteria**: 
  - [x] Parameter added to client payload
  - [x] Set to true by default
  - [x] Tests updated
- **Effort**: S
- **Files/Modules**: handler.py

### OCM-5: Production Pipeline Preparation
- **Description**: Prepare production deployment configuration
- **Acceptance Criteria**: 
  - [ ] Production variables configured
  - [ ] Manual approval workflow ready
  - [ ] Production testing completed
- **Effort**: M
- **Files/Modules**: .gitlab-ci.yml, backend-prod.hcl

### OCM-6: Integration Testing
- **Description**: Validate end-to-end OAuth client creation
- **Acceptance Criteria**: 
  - [ ] Dev environment tested
  - [ ] PingFederate integration verified
  - [ ] Error handling validated
- **Effort**: L
- **Files/Modules**: test_handler.py, integration tests