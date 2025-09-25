# Active Context

## Current Work Focus
**COMPLETED:** Java 25 upgrade for NFOS Mashgin Fueling Service pipelines and Terraform infrastructure alignment for Store Locator ADF.

## Recent Changes
### Java 25 Pipeline Upgrade (Latest)

**NFOS Mashgin Fueling Service Pipelines:**
- **Main Branch Pipeline (`azure-pipelines-main.yml`):** Updated to use Java 25
  - `containerImage: 'eclipse-temurin:25'`
  - `jdkVersionOption: '1.25'`
  - `javaVersion: '25.0.0+11'`
  - Parallel deployment to dev/qa/uat environments
  - Trigger: main branch only (`include: - main`)

- **Feature Branch Pipeline (`azure-pipelines.yml`):** Updated to use Java 25
  - `containerImage: 'eclipse-temurin:25'`
  - `jdkVersionOption: '1.25'`
  - `javaVersion: '25.0.0+11'`
  - Sequential deployment dev → qa → uat
  - Trigger: all branches except main (`exclude: - main`)

**Template Compatibility:**
- **tests-java-v24.yaml**: Confirmed parameterizable for Java 25 usage
- **Java Version Parameters**: Successfully configured through template parameters
- **Backward Compatibility**: Existing caching and optimization patterns maintained

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
**Project Complete** - All infrastructure alignment objectives achieved:
- ✅ UAT Terraform configuration matches existing infrastructure (no drift)
- ✅ Production configuration updated for consistency
- ✅ Data flow successfully imported and managed by Terraform
- ✅ All 9 UAT resources now managed by Terraform
- ✅ Infrastructure as Code fully implemented

## Active Decisions and Considerations

### Infrastructure Alignment Strategy
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

### Terraform State Management
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

### Terraform Import Best Practices
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
*Last Updated: September 25, 2025*
*Status: Terraform Infrastructure Alignment Complete*
*Current Focus: Store Locator ADF infrastructure fully managed by Terraform*
