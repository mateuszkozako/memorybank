# Task Breakdown - jeppesen-eks

## Task List

### JEKS-1: Bottlerocket OS Support
- **Description**: Add support for Bottlerocket container-optimized OS
- **Acceptance Criteria**: 
  - [ ] `os_image = "BOTTLEROCKET"` variable works
  - [ ] Bottlerocket AMI selection automated
  - [ ] Documentation updated with OS comparison
- **Effort**: M
- **Files/Modules**: variables.tf, main.tf, README.md

### JEKS-2: Node Image Management Enhancement
- **Description**: Improve node_image_id variable with comprehensive guidance
- **Acceptance Criteria**: 
  - [ ] Clear AMI sourcing instructions
  - [ ] Security warning about manual AMI management
  - [ ] Examples for both OS types
- **Effort**: S
- **Files/Modules**: variables.tf, README.md

### JEKS-3: Variable Cleanup and Simplification
- **Description**: Remove deprecated Kyverno variables
- **Acceptance Criteria**: 
  - [ ] Remove kyverno_mutate_on_create_only variable
  - [ ] Update documentation
  - [ ] Ensure backward compatibility
- **Effort**: S
- **Files/Modules**: variables.tf, main.tf

### JEKS-4: Documentation v5.6.0 Update
- **Description**: Complete documentation for breaking changes
- **Acceptance Criteria**: 
  - [ ] Breaking changes section updated
  - [ ] Migration guide provided
  - [ ] Examples refreshed
- **Effort**: M
- **Files/Modules**: README.md

### JEKS-5: Integration Testing
- **Description**: Validate both OS options in test environment
- **Acceptance Criteria**: 
  - [ ] Amazon Linux 2 deployment tested
  - [ ] Bottlerocket deployment tested
  - [ ] Node image override tested
- **Effort**: L
- **Files/Modules**: test/ directory