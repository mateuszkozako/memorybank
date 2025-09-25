# Mashgin Production Infrastructure Analysis - US Region
**Date:** 2025-01-15  
**Task:** Analyze NFOS Mashgin production infrastructure and evaluate `azurerm_client_config` security

## Overview
Analysis of `nfos-infrastructure/terraform/us/prd/main.tf` on the `feat/prd_infra` branch to understand production resources and security implications.

## Key Production Resources

### 1. **Azure Key Vault**
- Managed through the `app-v2` module
- Access policies configured for multiple service principals:
  - APIM service principal: `cb3e84df-e567-4d6d-9756-245234b781d8`
  - NFOS PRD owners: `a962d3d1-2c6d-49a1-a02a-a9630a5259aa`
  - QA service connection: `52d6fda8-3ba8-4dd3-ad5d-fc07b9939281`
  - Dev service connection: `254e1e4f-4fb8-46a7-8a3a-6353259a6a27`
  - PRD service connection: `18d4a524-c7f9-422c-9b11-3221349da589`
  - Azure NGRP NFOS prod contributors: `62744219-8418-45f3-b82e-06092a1cdde1`

### 2. **Cosmos DB (DocumentDB)**
- **Account**: `ngrp-prd-nfos-us-cosmos`
  - Type: GlobalDocumentDB with Serverless capability
  - Consistency: Bounded Staleness (300 seconds, 100000 prefix)
  - TLS: Minimum version 1.2
  - Network restrictions:
    - Virtual network filtering enabled (AKS subnet)
    - IP range filtering for Azure portal and data centers
  
- **Database**: `ngrp-prd-nfos-us-db`
  
- **Containers**:
  1. `eventdescriptors` - Partition key: `/stream/id`
  2. `leaseeventdescriptors` - Partition key: `/id`
  3. `leasedomaineventdescriptors` - Partition key: `/id`
  4. `fueling-data-for-reconciliation` - Partition key: `/transactionId`
  5. `lease-read-model` - Partition key: `/id`

### 3. **Event Grid**
- **Topic**: `ngrp-prd-nfos-us-nfos`
  - Deployed via ARM template (API version 2025-02-15)
  - Minimum TLS version: 1.2
  
- **Event Subscription**: `ngrp-prd-ext-api-us-nfos-event-sub`
  - Target: External API Event Hub
  - Retry policy: 30 attempts, 1440 minutes TTL
  - Partition key based on `data.fuelingOrderId`

### 4. **Event Hub**
- **Namespace**: `ngrp-prd-nfos-us-namespace`
  - SKU: Standard
  - Capacity: 1 unit
  
- **Event Hub**: `ngrp-prd-nfos-us-event-hub`
  - Partitions: 2
  - Message retention: 1 day
  - Authorization rule: `AppSharedAccessKey` (Listen + Send permissions)

### 5. **Storage Account**
- **Name**: `ngrpprdnfosuscheckpoints`
  - Purpose: Event Hub checkpointing
  - Tier: Standard
  - Container: `ngrp-prd-nfos-us-event-hub`

### 6. **Application Insights & Log Analytics**
- Managed through the `app-v2` module
- Daily quota: 3 GB

## Security Analysis of `azurerm_client_config` Usage

### What Gets Stored in State File
The `data "azurerm_client_config" "current" {}` retrieves:
- **tenant_id**: Azure AD tenant ID (stored in state)
- **client_id**: Service principal/user client ID (stored in state)
- **object_id**: Service principal/user object ID (stored in state)
- **subscription_id**: Azure subscription ID (stored in state)

### Security Implications
1. **Low Risk**: The data source only exposes IDs, not secrets or credentials
2. **State File Contains**: Only the identity information of whoever ran Terraform
3. **Safe Usage Pattern**: Used only for setting `tenant_id` in Key Vault access policies

### Current Usage in Production
```hcl
data "azurerm_client_config" "current" {}

resource "azurerm_key_vault_access_policy" "kv-access" {
  key_vault_id = module.application.kv_id
  tenant_id    = data.azurerm_client_config.current.tenant_id  # Only uses tenant_id
  # ... rest of configuration
}
```

## Production Environment Architecture

### Network Configuration
- Virtual Network filtering enabled on Cosmos DB
- AKS subnet integration configured
- IP allowlist for Azure portal access

### High Availability & Reliability
- Cosmos DB with serverless scaling
- Event Grid with robust retry policy (30 attempts)
- Event Hub with 2 partitions for parallel processing
- Storage account for reliable checkpointing

### Compliance & Security
- TLS 1.2 minimum across all services
- Managed identities and service principals for access control
- No public blob access on storage accounts
- Virtual network service endpoints configured

## Key Integration Points
1. **Event Flow**: Event Grid → External API Event Hub
2. **Cosmos DB**: Multiple containers for event sourcing pattern
3. **Key Vault**: Central secret management for all services
4. **Application Insights**: Centralized monitoring and logging

## Findings Summary

### Security Assessment
✅ **SAFE**: The `azurerm_client_config` usage is secure
- Only exposes non-sensitive IDs (no credentials)
- Used appropriately for tenant_id reference
- Standard practice in Terraform Azure deployments

### Infrastructure Highlights
- **Event Sourcing Pattern**: Well-implemented with Cosmos DB containers
- **Security-First Design**: Network isolation, managed identities, TLS enforcement
- **Scalable Architecture**: Serverless Cosmos DB, partitioned Event Hub
- **Monitoring**: Comprehensive with Application Insights

## Recommendations
1. ✅ Continue using `azurerm_client_config` - it's safe and follows best practices
2. Consider implementing resource locks on critical production resources
3. Review Event Hub retention policy (currently 1 day) for production needs
4. Consider geo-redundancy for Cosmos DB in production scenarios
5. Ensure backup strategies are implemented for all stateful resources
6. Document disaster recovery procedures

## Commands Executed
```bash
# Checked out to production infrastructure branch
cd nfos-infrastructure && git checkout feat/prd_infra

# Analyzed Terraform configuration
cd terraform/us/prd
terraform init  # Would be needed for actual deployment
```

## Files Analyzed
- `nfos-infrastructure/terraform/us/prd/main.tf`
- `nfos-infrastructure/terraform/us/prd/variables.tf`
- `nfos-infrastructure/terraform/us/prd/backend.tf`

---
**Task Status:** ✅ Complete  
**Next Steps:** Review additional infrastructure components if needed
