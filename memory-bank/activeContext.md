# Active Context

## Current Work Focus
**COMPLETED:** Azure DevOps pipeline template fixes for multiple repository checkout scenarios and Java v24/v25 template updates.

## Recent Changes
### Azure DevOps Pipeline Template Fixes (Latest - September 26, 2025)

**Multiple Repository Checkout Issue Resolved:**
- **Root Cause:** When Azure DevOps pipelines checkout multiple repositories, they cannot all use the `/s` directory and get placed in repository-named directories (e.g., `/home/vsts/work/1/s/nfos-mashgin/`)
- **Error Fixed:** "Not found wrapperScript: /home/vsts/work/1/s/gradlew" 
- **Solution Applied:** Updated all Java v24/v25 pipeline templates to use repository-specific paths for Gradle wrapper scripts

**Pipeline Template Updates Applied:**
- **ndevops-infrastructure/pipeline-jobs/tests-java-v24.yaml:**
  - Modified gradleWrapperFile parameter from `"./gradlew"` to `"./${{ parameters.repository }}/gradlew"`
  - Ensures Gradle wrapper is found in correct repository-specific directory
  
- **ndevops-infrastructure/pipeline-jobs/tests-java-v25.yaml:**
  - Applied same gradleWrapperFile fix as v24 template
  - Repository-specific path resolution for multiple checkout scenarios

- **ndevops-infrastructure/pipeline-tasks/tests-java-v6.yaml:**
  - Added repository parameter with default `"self"`
  - Removed duplicate repository parameter that was causing confusion
  - Underlying task template used by both v24 and v25 job templates

- **ndevops-infrastructure/pipeline-tasks/src-build-java-v1.yaml:**
  - Added repository parameter with default `"self"`
  - Updated to support repository-specific Gradle wrapper paths
  - Source build task template for Gradle builds

**Microsoft Documentation Reference:**
- **Issue:** According to Microsoft docs, when multiple repositories are checked out, they cannot all use the `/s` directory
- **Behavior:** Each repository gets its own directory named after the repository (e.g., `/home/vsts/work/1/s/repository-name/`)
- **Impact:** Gradle wrapper scripts must be referenced with repository-specific paths

**Changes Committed:**
- **Branch:** `feat/pipeline_template`
- **Commit Hash:** `0928984`
- **Status:** All Java v24/v25 templates updated and ready for use

### Java 25 Pipeline Parameter Fixes (September 25, 2025)

**Pipeline Validation Errors Resolved:**
- **Missing Parameters Fixed:** Added required template parameters to both pipelines
  - `enableTests: true`
  - `repositoryName: 'nfos-service-mashgin-fueling'` 
  - `options: '-Xmx4096m'`
- **Template Verification:** Confirmed `tests-java-v24.yaml` fully supports Java 25
- **Comprehensive Java 25 Support:** Template includes custom installation logic, caching, and environment configuration

**NFOS Mashgin Fueling Service Pipelines:**
- **Main Branch Pipeline (`azure-pipelines-main.yml`):** Updated to use Java 25
  - `containerImage: 'eclipse-temurin:25'`
  - `jdkVersionOption: '1.25'`
  - `javaVersion: '25.0.0+11'`
  - **FIXED:** Added all required template parameters (`enableTests`, `repositoryName`, `options`)
  - Parallel deployment to dev/qa/uat environments
  - Trigger: main branch only (`include: - main`)

- **Feature Branch Pipeline (`azure-pipelines.yml`):** Updated to use Java 25
  - `containerImage: 'eclipse-temurin:25'`
  - `jdkVersionOption: '1.25'`
  - `javaVersion: '25.0.0+11'`
  - **FIXED:** Added all required template parameters (`enableTests`, `repositoryName`, `options`)
  - Sequential deployment dev → qa → uat
  - Trigger: all branches except main (`exclude: - main`)

**Template Compatibility Verified:**
- **tests-java-v24.yaml**: Confirmed full Java 25 support with comprehensive features
  - Java 25 caching mechanism: `key: 'java25-$(Agent.OS)-openjdk25'`
  - Custom installation logic with Eclipse Temurin download
  - Environment variable configuration for Java 25
  - Fallback installation methods
  - All required parameters: `enableTests`, `repositoryName`, `options`
- **Java Version Parameters**: Successfully configured through template parameters
- **Backward Compatibility**: Existing caching and optimization patterns maintained
- **Installation Logic**: Custom Java 25 installation from Eclipse Temurin with proper caching

### Terraform Infrastructure Alignment (September 25, 2025)

**Store Locator ADF Infrastructure (`stoloc-infrastructure/terraform/us/adf/`):**
- **UAT Environment:** Complete configuration alignment with existing Azure infrastructure
- **Production Environment:** Added missing managed identity configuration for consistency
- **Data Flow Import:** Successfully imported "own_prices_data_flow" into UAT Terraform state
- **Final Verification:** UAT terraform plan shows "No changes. Your infrastructure matches the configuration."

### Specific Changes Made

**UAT Configuration (`stoloc-infrastructure/terraform/us/adf/uat/`):**
- **main.tf:** Updated all resource configurations to match existing Azure infrastructure
  - SQL Database: Changed SKU from "Basic" to "S3" to match deployed resource
  - SQL Server: Added `express_vulnerability_assessment_enabled = true`
  - Storage Account: Updated to match existing settings (public access, retention policies, versioning)
  - Data Factory: Added correct Application and Compute tags
  - **NEW:** Added `azurerm_data_factory_data_flow` resource for "own_prices_data_flow"
- **variables.tf:** Updated Compute tag from "GLEUAKS" to "GLUSAKS" to match existing infrastructure
- **Import:** Successfully imported existing data flow into Terraform state

**Production Configuration (`stoloc-infrastructure/terraform/us/adf/prd/`):**
- **main.tf:** Added missing `use_managed_identity = true` to SQL linked service for consistency

### Data Flow Details
**Imported Data Flow: "own_prices_data_flow"**
- **Purpose:** Processes CSV fuel price data with validation and transformation
- **Source:** CSVFuelPriceFlow (dataset: CSVFuelPriceFile_v2)
- **Sink:** CSVFuelPriceFlowSink (dataset: own_prices_stg)
- **Transformations:**
  - CSVFuelPriceFilteredNumericSiteIds: Filters non-numeric site IDs
  - CSVFuelPriceFilteredRealSiteIds: Filters fake site IDs (7-digit, starts with 2 or 4)
  - CSVFuelPriceNotNullValuesAssertion: Validates required fields are not null
- **Script:** 37-line data flow script with comprehensive data validation and transformation logic
- **Configuration:** Uses `script_lines` array format with lifecycle ignore for script changes

## Next Steps
**Pipeline Template Work Complete** - Key objectives achieved:
- ✅ Azure DevOps multiple repository checkout issue resolved
- ✅ Java v24/v25 templates updated with repository-specific paths
- ✅ Gradle wrapper script path resolution fixed for all scenarios
- ✅ All changes committed to `feat/pipeline_template` branch
- ✅ Templates ready for use in multi-repository pipeline scenarios

**Outstanding Tasks:**
- [ ] Original nfos-service-mashgin-fueling main-branch pipeline creation (pending)
- [ ] Review and merge `feat/pipeline_template` branch changes
- [ ] Validate template fixes in actual multi-repository pipeline scenarios

## Active Decisions and Considerations

### Pipeline Template Architecture Strategy
- **Decision:** Update pipeline templates to handle multiple repository checkouts correctly
- **Rationale:** Microsoft documentation indicates multiple repositories get separate directories, not shared `/s` directory
- **Implementation:** Modified gradleWrapperFile parameter to use repository-specific paths
- **Result:** Gradle wrapper scripts now found correctly in multi-repository scenarios

### Template Parameter Design
- **Decision:** Add repository parameter with sensible default to underlying task templates
- **Process:** 
  1. Identified job templates need repository-aware paths
  2. Added repository parameter to underlying task templates
  3. Updated gradleWrapperFile to use `./${{ parameters.repository }}/gradlew`
  4. Maintained backward compatibility with default `"self"` value
- **Outcome:** Templates work for both single and multiple repository scenarios

### Infrastructure Alignment Strategy (Previous Work)
- **Decision:** Update Terraform to match existing infrastructure rather than modify infrastructure
- **Rationale:** Preserve existing working infrastructure while bringing it under Terraform management
- **Implementation:** Analyzed terraform plan output and adjusted configurations to eliminate drift
- **Result:** Zero-drift infrastructure management achieved

### Data Flow Import Approach
- **Decision:** Import existing data flow rather than recreate
- **Process:** 
  1. Discovered using Azure CLI (`az datafactory data-flow list`)
  2. Added Terraform resource configuration
  3. Imported using `terraform import` command
  4. Adjusted configuration to match existing resource
- **Outcome:** Seamless integration with zero disruption to existing data processing

### Configuration Management Pattern
- **UAT vs Production Differences:** Maintained appropriate environment-specific configurations
  - Database SKU: UAT uses S3, PRD uses S3 with size limits
  - Storage replication: UAT uses LRS, PRD uses RAGRS
  - Network access: Different security configurations per environment
  - Admin logins: Environment-specific SQL admin accounts
- **Consistency Areas:** Ensured common patterns like managed identity usage

## Important Patterns and Preferences

### Azure DevOps Pipeline Template Patterns
- **Multi-Repository Support:** Always use repository-specific paths for file references
- **Parameter Propagation:** Ensure parameters flow correctly from job templates to task templates
- **Backward Compatibility:** Maintain default values for optional parameters
- **Template Hierarchy:** Job templates call task templates, both need repository awareness
- **Path Resolution:** Use `./${{ parameters.repository }}/filename` pattern for repository-specific files

### Template Parameter Management
- **Repository Parameter:** Use `repository` parameter with default `"self"` for single repository scenarios
- **gradleWrapperFile:** Always construct using repository parameter for multi-repo support
- **Parameter Validation:** Ensure all required parameters are defined at appropriate template level
- **Default Values:** Provide sensible defaults that work for most common scenarios

### Terraform State Management (Previous Work)
- **Import Strategy:** Use full Azure resource IDs for terraform import commands
- **Configuration Alignment:** Match Terraform configuration exactly to existing resource properties
- **Lifecycle Management:** Use `ignore_changes` for dynamic properties (script_lines, activities_json)
- **Verification:** Always run `terraform plan` after import to ensure zero drift

### Azure Data Factory Patterns
- **Data Flow Configuration:** Use `script_lines` array format, not single `script` string
- **Resource Dependencies:** Data flows depend on datasets and linked services
- **Script Management:** Ignore script changes in lifecycle to prevent unwanted updates
- **Naming Conventions:** Follow existing Azure resource naming patterns

### Infrastructure Drift Resolution
- **Analysis:** Use `terraform plan` output to identify specific configuration mismatches
- **Resolution Priority:** 
  1. Tags and metadata (low risk)
  2. Configuration settings (medium risk)
  3. Resource properties (high risk)
- **Validation:** Ensure final plan shows "No changes" before considering complete

## Learnings and Project Insights

### Azure DevOps Multiple Repository Checkout
- **Key Learning:** Multiple repository checkouts change directory structure significantly
- **Microsoft Behavior:** Each repository gets its own directory instead of sharing `/s` directory
- **Path Impact:** All file references must be repository-aware in multi-repo scenarios
- **Template Design:** Job templates and task templates both need repository parameter support
- **Backward Compatibility:** Default `"self"` parameter ensures single-repo scenarios continue working

### Pipeline Template Architecture
- **Template Hierarchy:** Job templates (`tests-java-v24.yaml`) call task templates (`tests-java-v6.yaml`)
- **Parameter Flow:** Repository parameter must flow from job to task level
- **File Path Construction:** Use parameter interpolation for dynamic paths (`./${{ parameters.repository }}/gradlew`)
- **Testing Strategy:** Template changes affect both single and multiple repository scenarios
- **Version Management:** Both v24 and v25 templates needed identical fixes

### Terraform Import Best Practices (Previous Work)
- **Resource Discovery:** Use Azure CLI to discover existing resources and their properties
- **Configuration First:** Add Terraform resource configuration before importing
- **Iterative Alignment:** Import, plan, adjust configuration, repeat until zero drift
- **State Verification:** Confirm successful import with terraform plan validation

### Azure Data Factory Terraform Patterns
- **Data Flow Complexity:** Data flows have complex nested structures requiring careful configuration
- **Script Format:** Azure provider expects `script_lines` array, not concatenated script string
- **Dynamic Properties:** Some properties change frequently and should be ignored in lifecycle
- **Resource Relationships:** Data flows reference datasets and linked services by name

### Infrastructure Alignment Challenges
- **Tag Inconsistencies:** Common source of drift, especially computed vs. configured tags
- **Property Defaults:** Azure may set defaults that differ from Terraform defaults
- **Resource State:** Some properties reflect current Azure state vs. desired configuration
- **Environment Differences:** Legitimate differences between UAT/PRD must be preserved

### Multi-Environment Management
- **Consistency vs. Differences:** Balance between standardization and environment-specific needs
- **Configuration Patterns:** Use similar structures but different values per environment
- **Security Considerations:** Production requires stricter network and access controls
- **Resource Sizing:** Different SKUs and configurations appropriate for different environments

---
*Last Updated: September 26, 2025*
*Status: Azure DevOps Pipeline Template Fixes for Multiple Repository Checkout Complete*
*Current Focus: Java v24/v25 templates updated and ready for multi-repository scenarios*
