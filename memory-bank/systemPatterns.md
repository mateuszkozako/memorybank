# System Patterns

## Architecture Overview
### System Type
**Azure DevOps CI/CD Pipeline Infrastructure with Multi-Repository Support**

### High-Level Architecture
```
Azure DevOps Pipeline
      |
      v
[Multiple Repository Checkout]
      |
      v
[Repository-Specific Directories]
   /s/repo-name1/
   /s/repo-name2/
      |
      v
[Template Hierarchy]
Job Templates (v24, v25) --> Task Templates (v6, src-build-v1)
      |
      v
[Dynamic Path Resolution]
./${{ parameters.repository }}/gradlew
```

### Core Components
| Component | Purpose | Technology | Location |
|-----------|---------|------------|----------|
| **Job Templates** | High-level pipeline orchestration with Java setup | Azure DevOps YAML | `ndevops-infrastructure/pipeline-jobs/` |
| **Task Templates** | Low-level build and test execution | Azure DevOps YAML | `ndevops-infrastructure/pipeline-tasks/` |
| **Repository Parameter** | Multi-repo path resolution | Template Parameters | All pipeline templates |
| **Gradle Wrapper** | Build tool execution | Gradle | Repository-specific paths |

## Design Patterns
### Pipeline Template Hierarchy Pattern
- **Pattern:** Job Templates → Task Templates
  - **Implementation:** Job templates (`tests-java-v24.yaml`, `tests-java-v25.yaml`) call task templates (`tests-java-v6.yaml`, `src-build-java-v1.yaml`)
  - **Benefits:** Separation of concerns, reusability, maintainability

### Repository Parameter Propagation Pattern
- **Pattern:** Parameter Flow Down Template Hierarchy
  - **Used In:** All pipeline templates
  - **Purpose:** Enable multi-repository support while maintaining backward compatibility

### Dynamic Path Resolution Pattern  
- **Pattern:** Template Parameter Interpolation for File Paths
  - **Implementation:** `./${{ parameters.repository }}/gradlew` instead of `./gradlew`
  - **Benefits:** Works for both single and multiple repository checkout scenarios

## Data Flow
### Pipeline Execution Flow
1. **Entry Point:** Azure DevOps trigger (branch push, PR)
2. **Repository Checkout:** Multiple repositories checked out to separate directories
3. **Template Resolution:** Job template parameters passed to task templates
4. **Path Construction:** Dynamic paths built using repository parameter
5. **Build Execution:** Gradle wrapper executed from correct repository directory

### Template Parameter Pipeline
```
Pipeline Trigger --> [Job Template] --> [Parameter Validation] --> [Task Template] --> [Build Execution]
                           |                    |                        |                    |
                           v                    v                        v                    v
                    [Repository Param]   [Path Resolution]         [Gradle Wrapper]    [Artifact Output]
```

## Key Abstractions
### Template Interfaces
- **Interface:** Job Template Contract
  - **Purpose:** Abstracts high-level pipeline orchestration
  - **Implementations:** `tests-java-v24.yaml`, `tests-java-v25.yaml`

- **Interface:** Task Template Contract  
  - **Purpose:** Abstracts low-level build and test operations
  - **Implementations:** `tests-java-v6.yaml`, `src-build-java-v1.yaml`

### Parameter Abstractions
- **Parameter:** `repository`
  - **Purpose:** Abstracts repository-specific path resolution
  - **Default:** `"self"` for backward compatibility
  - **Usage:** Dynamic path construction in multi-repo scenarios

## State Management
### Pipeline State
- **State Type:** Stateless Pipeline Execution
- **Management:** Azure DevOps agent state per pipeline run
- **Persistence:** Artifacts cached between stages, build outputs persisted

### Repository State
- **Primary Store:** Git repositories with separate checkout directories
- **Caching Strategy:** Gradle dependencies, Java installations, Docker images cached
- **Directory Structure:** Each repository gets dedicated directory in multi-repo scenarios

## Error Handling
### Strategy
- **Approach:** Template parameter validation and fallback patterns
- **Logging:** Azure DevOps pipeline logs with detailed error messages
- **Recovery:** Default parameter values provide graceful degradation

### Common Error Patterns
| Error Type | Handling | Recovery |
|------------|----------|----------|
| **Gradle Wrapper Not Found** | Repository parameter path resolution | Default to `"self"` repository |
| **Template Parameter Missing** | Parameter validation in templates | Fail fast with clear error message |
| **Multiple Repository Path Issues** | Dynamic path construction | Repository-specific directory detection |

## Security Patterns
### Authentication
- **Method:** Azure DevOps Service Connections and Managed Identities
- **Implementation:** Pipeline templates inherit security context from calling pipeline

### Authorization
- **Strategy:** Azure DevOps project-level permissions
- **Implementation:** Templates execute with pipeline service account permissions

### Data Protection
- **Encryption:** Secrets managed through Azure DevOps variable groups and Key Vault
- **Validation:** Template parameter validation prevents injection attacks
- **Path Security:** Repository parameter validation prevents directory traversal

## Performance Patterns
### Optimization Strategies
- **Caching:** Java installations, Gradle dependencies, Docker images, build artifacts
- **Lazy Loading:** Java installation only when not cached
- **Batch Processing:** Parallel environment deployments in main branch pipeline

### Bottlenecks & Solutions
| Bottleneck | Solution | Impact |
|------------|----------|--------|
| **Repeated Java Downloads** | Java installation caching with version-specific keys | Faster pipeline execution |
| **Gradle Dependency Downloads** | Gradle cache with source-based cache keys | Reduced build times |
| **Multiple Repository Checkout Overhead** | Repository-specific path optimization | Correct build execution |

## Testing Patterns
### Template Testing Structure
- **Template Validation:** Azure DevOps YAML schema validation
- **Parameter Testing:** Test templates with various parameter combinations
- **Multi-Repository Testing:** Validate templates work in both single and multi-repo scenarios

### Pipeline Testing Strategy
- **What's Tested:** Build execution, test execution, deployment processes
- **How:** Real pipeline runs with test repositories and environments
- **Validation:** Successful artifact generation and deployment verification

## Deployment Patterns
### Build Process
1. **Repository Checkout:** Multiple repositories checked out to separate directories
2. **Template Resolution:** Job templates resolve parameters and call task templates  
3. **Path Construction:** Dynamic paths built using repository parameter
4. **Gradle Execution:** Gradle wrapper executed from correct repository location
5. **Artifact Generation:** Build artifacts cached for reuse in deployment stages

### Deployment Strategy
- **Method:** Azure DevOps YAML pipelines with template inheritance
- **Environments:** Dev/QA/UAT with environment-specific configurations
- **Branching:** Main branch (parallel deployment), feature branches (sequential deployment)

## Multi-Repository Architecture Patterns

### Directory Structure Pattern
```
Azure DevOps Agent Workspace:
/home/vsts/work/1/s/
├── repository-name-1/
│   ├── gradlew
│   ├── build.gradle.kts  
│   └── src/
├── repository-name-2/
│   ├── gradlew
│   ├── pom.xml
│   └── src/
└── ndevops-infrastructure/
    ├── pipeline-jobs/
    └── pipeline-tasks/
```

### Template Parameter Flow Pattern
```
Pipeline Definition
    ↓ (calls)
Job Template (tests-java-v24.yaml)
    ↓ (parameters: repository, enableTests, etc.)
Task Template (tests-java-v6.yaml)
    ↓ (constructs: ./${{ parameters.repository }}/gradlew)
Gradle Execution
```

### Backward Compatibility Pattern
- **Default Values:** All new parameters have sensible defaults (`repository: "self"`)
- **Single Repository Support:** Default behavior unchanged for single repository scenarios
- **Multi-Repository Enhancement:** Additional functionality available when repository parameter specified

---
*Last Updated: September 26, 2025*
*Architecture Version: Azure DevOps Multi-Repository Pipeline Templates v1.0*
*Pattern Focus: Multi-repository checkout support with backward compatibility*
