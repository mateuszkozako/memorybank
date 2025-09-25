# Cosmos DB Indexing Optimization - Mashgin Fueling Service
**Date:** 2025-09-16  
**Task:** Optimize Cosmos DB indexing policy for fueling-processes container across all environments

## Overview
Updated the indexing policy for the `fueling-processes` container in Cosmos DB to use `refNumber` as the ONLY indexed field, replacing the previous wildcard indexing policy that indexed all fields.

## Problem Statement
The original indexing policy used wildcard indexing (`"/*"`), which:
- Indexed all fields in documents, creating unnecessary overhead
- Increased storage costs due to excessive indexing
- Potentially impacted write performance
- Did not optimize for the primary query pattern using `refNumber`

## Solution Implemented

### Indexing Policy Changes
**Before:**
```hcl
indexing_policy {
  indexing_mode = "consistent"
  
  included_path {
    path = "/*"  # All fields indexed
  }
}
```

**After:**
```hcl
indexing_policy {
  indexing_mode = "consistent"
  
  included_path {
    path = "/refNumber/?"  # Only refNumber indexed
  }
  
  excluded_path {
    path = "/*"  # All other fields excluded
  }
}
```

### Environments Updated
1. **Development**: `nfos-infrastructure/terraform/us/mashgin-fueling/dev/main.tf`
2. **QA**: `nfos-infrastructure/terraform/us/mashgin-fueling/qa/main.tf`
3. **UAT**: `nfos-infrastructure/terraform/us/mashgin-fueling/uat/main.tf`
4. **Production**: `nfos-infrastructure/terraform/us/mashgin-fueling/prd/main.tf`

## Technical Details

### Terraform Configuration
Each environment's `fueling-processes` container was updated with:
- `included_path` set to `/refNumber/?` for specific field indexing
- `excluded_path` set to `/*` to explicitly exclude all other fields
- `partition_key_kind = "Hash"` maintained for consistency

### Validation Process
- Ran `terraform init` and `terraform plan` for each environment
- Verified that terraform plan showed the expected changes:
  - Adding `excluded_path` for `"/*"`
  - Changing `included_path` from `"/*"` to `"/refNumber/?"`
- Confirmed no other resources were affected by the changes

## Expected Benefits

### Performance Improvements
1. **Query Performance**: Optimized for `refNumber`-based queries
2. **Write Performance**: Reduced indexing overhead during document writes
3. **Storage Efficiency**: Significantly reduced index storage requirements

### Cost Optimization
- Lower Request Unit (RU) consumption for write operations
- Reduced storage costs due to smaller index footprint
- More predictable performance characteristics

## Implementation Results

### Terraform Plan Output (Example from Dev)
```
# azurerm_cosmosdb_sql_container.fueling-processes will be updated in-place
~ resource "azurerm_cosmosdb_sql_container" "fueling-processes" {
    # (5 unchanged attributes hidden)

  ~ indexing_policy {
      # (1 unchanged attribute hidden)

    + excluded_path {
        + path = "/*"
      }

    ~ included_path {
        ~ path = "/*" -> "/refNumber/?"
      }
  }
}
```

### Verification Status
- ✅ **Dev Environment**: Plan validated successfully
- ✅ **QA Environment**: Plan validated successfully  
- ✅ **UAT Environment**: Plan validated successfully
- ✅ **PRD Environment**: Plan validated successfully (after fixing module paths)

## Additional Changes Made

### Production Environment Fix
Fixed incorrect module paths in production environment:
- Changed from `../../../../ndevops-infrastructure/terraform/modules/`
- To `../../../../../ndevops-infrastructure/terraform/modules/`

This resolved terraform initialization errors in the production environment.

## Deployment Considerations

### Rollout Strategy
1. **Recommended Order**: Dev → QA → UAT → PRD
2. **Monitoring**: Monitor query performance after each deployment
3. **Rollback Plan**: Previous terraform state can be used to revert if needed

### Potential Impact
- **Queries using refNumber**: Improved performance
- **Queries using other fields**: May experience performance degradation (requires full scan)
- **Write operations**: Improved performance due to reduced indexing overhead

## Best Practices Applied

### Multi-Environment Consistency
- Applied identical indexing policy across all environments
- Maintained consistent terraform configuration patterns
- Used proper module path references

### Infrastructure as Code
- All changes implemented through terraform
- Changes validated before deployment
- Proper version control and documentation

## Next Steps
1. Deploy changes to environments in recommended order
2. Monitor query performance metrics post-deployment
3. Validate that refNumber-based queries perform as expected
4. Consider adding composite indexes if additional query patterns emerge

## Files Modified
- `nfos-infrastructure/terraform/us/mashgin-fueling/dev/main.tf`
- `nfos-infrastructure/terraform/us/mashgin-fueling/qa/main.tf`
- `nfos-infrastructure/terraform/us/mashgin-fueling/uat/main.tf`
- `nfos-infrastructure/terraform/us/mashgin-fueling/prd/main.tf`

---
**Task Status:** ✅ Complete  
**Ready for Deployment:** Yes  
**Validation:** Terraform plan successful across all environments
