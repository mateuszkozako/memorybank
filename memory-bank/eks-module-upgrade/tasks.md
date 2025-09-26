# Task Breakdown - eks-module-upgrade

## Task List

### EKSUP-1: Capture Official EKS Module Baseline
- **Description**: Gather current state of the upstream Terraform AWS EKS module (release cadence, supported Kubernetes versions, provider requirements) from the official GitHub repository to contextualize the jeppesen-eks upgrade path.
- **Acceptance Criteria**:
  - [ ] Latest tagged release and corresponding changelog URL documented
  - [ ] Supported Kubernetes version range captured
  - [ ] Provider version requirements (terraform, aws, kubernetes, null) summarized
  - [ ] Notes captured in `memory-bank/eks-module-upgrade/context.md`
- **Effort**: M
- **Files/Modules**: `memory-bank/eks-module-upgrade/context.md`

### EKSUP-2: Audit jeppesen-eks Null Resource Usage
- **Description**: Inspect the local `jeppesen-eks` Terraform module to catalog every `null_resource` and determine execution patterns, triggers, and dependency implications for the upgrade.
- **Acceptance Criteria**:
  - [ ] Inventory of `null_resource` blocks with file paths and purpose documented
  - [ ] Any embedded `local-exec`/`remote-exec` behaviors summarized with upgrade risk notes
  - [ ] Identification of provider versions used for the `null` provider (via lock files or module configuration)
  - [ ] Findings captured in `memory-bank/eks-module-upgrade/context.md`
- **Effort**: M
- **Files/Modules**: `jeppesen-eks/**`, `.terraform.lock.hcl`, `memory-bank/eks-module-upgrade/context.md`

### EKSUP-3: Verify jeppesen-eks Source Module Integration Points
- **Description**: Trace how the `jeppesen-eks` module is referenced in the wider platform (Flux manifests or Terraform wrappers) to confirm source URLs, version pins, and dependency constraints relevant to the upgrade.
- **Acceptance Criteria**:
  - [ ] Source manifest or module callsites identified with exact version/tag references
  - [ ] Any pipeline or GitOps automation touching the module documented
  - [ ] Risks or blockers for bumping module/Kubernetes versions noted
  - [ ] Summary added to `memory-bank/eks-module-upgrade/context.md`
- **Effort**: S
- **Files/Modules**: `jeppesen-eks/**`, `integration-test-module/**`, `memory-bank/eks-module-upgrade/context.md`

