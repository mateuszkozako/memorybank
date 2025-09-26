# Product Thinking - eks-module-upgrade

## Problem Statement
The existing EKS clusters deployed through the jeppesen-eks Terraform module are running Kubernetes v1.19, which is end-of-life and unsupported by AWS. We must upgrade to a supported version to maintain security patches, API compatibility, and AWS supportability.

## User Story
As a platform engineer, I need the jeppesen-eks module and related deployment assets to support upgrading managed EKS clusters from v1.19 through the required intermediate versions (v1.20) and ideally landing on v1.21 so that clusters stay compliant and receive ongoing security updates without service disruption.

## Success Metrics
- All targeted clusters operate on Kubernetes v1.21 (or at minimum v1.20) after the upgrade window.
- Zero critical regression incidents (P0/P1) during or immediately after the upgrade.
- Terraform plans for the upgrade execute cleanly with no manual interventions.
- Post-upgrade validation (smoke tests, workload health checks) report green status within agreed SLAs.

## Scope & Constraints
- Must follow AWS EKS version upgrade sequencing (v1.19 → v1.20 → v1.21); skipping intermediate versions is not allowed.
- Upgrade path should account for managed node groups, add-ons, and Bottlerocket/AL2 OS combinations.
- Downtime must be minimized; workloads should continue running via rolling node group updates.
- Any Terraform null_resource usage must be reviewed to ensure idempotency across version upgrades.
- Documentation and runbooks for executing the upgrade must accompany the technical changes.

## Dependencies
- AWS EKS control plane upgrade scheduling and service quotas.
- jeppesen-eks Terraform module features, null_resource behavior, and provider version compatibility (including null provider).
- CI/CD pipelines performing terraform apply for cluster environments.
- Cluster add-ons (e.g., VPC CNI, CoreDNS, KubeProxy, Istio, ADOT) and AWS-managed EKS addons requiring compatible versions.
