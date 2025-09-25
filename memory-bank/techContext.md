# Technical Context

## Technologies Used

### Core Technologies
- **Azure DevOps Pipelines**: CI/CD orchestration platform
- **Java 25**: Application runtime (Eclipse Temurin) - **UPGRADED FROM JAVA 24**
- **Gradle**: Build automation tool with wrapper (gradlew)
- **Docker**: Containerization platform
- **Azure Container Registry**: Docker image storage

### Pipeline Templates
- **tests-java-v24.yaml**: Advanced testing template with comprehensive caching and **VERIFIED Java 25 support**
  - Custom Java 25 installation logic with Eclipse Temurin download
  - Java 25-specific caching: `key: 'java25-$(Agent.OS)-openjdk25'`
  - Environment configuration for Java 25 (JAVA_HOME, PATH setup)
  - All required parameters: `enableTests`, `repositoryName`, `options`
  - Fallback installation methods for reliability
- **build-java-v5.yaml**: Docker build template with artifact download support
- **tests-secrets-v3.yaml**: Security scanning template
- **tests-container-v3.yaml**: Container security scanning

### Caching Technologies
- **Azure DevOps Cache@2 Task**: Pipeline caching mechanism
- **Gradle Build Cache**: Incremental build optimization
- **Docker Layer Cache**: Container image optimization

## Development Setup

### Pipeline Structure
```
nfos-service-mashgin-fueling/
├── pipelines/
│   └── us/
│       ├── azure-pipelines-main.yml    # Main branch pipeline
│       └── azure-pipelines.yml         # Feature branch pipeline
├── Dockerfile                          # Container build definition
├── build.gradle.kts                    # Gradle build configuration
└── gradlew                             # Gradle wrapper script
```

### Template Repository
```
ndevops-infrastructure/
├── pipeline-jobs/
│   ├── tests-java-v24.yaml            # Advanced Java testing with caching
│   ├── build-java-v5.yaml             # Docker build with artifact support
│   └── tests-secrets-v3.yaml          # Security scanning
└── pipeline-tasks/
    ├── container-build-and-publish-v1.yaml
    └── replace-values-v1.yaml
```

## Technical Constraints

### Platform Limitations
- **Azure DevOps**: Limited to available pipeline templates and caching mechanisms
- **Java 25**: Requires custom installation as not pre-installed on agents (upgraded from Java 24, **template verified to support Java 25**)
- **Template Parameters**: Must include all required parameters (`enableTests`, `repositoryName`, `options`) for validation
- **Gradle**: Must use wrapper for consistency across environments

### Performance Constraints
- **Agent Resources**: Limited memory and CPU on Azure DevOps agents
- **Cache Size**: Azure DevOps cache storage limitations
- **Network**: Download speeds for dependencies and base images

### Compatibility Requirements
- **Template Versions**: Must use compatible template versions from ndevops-infrastructure
- **Parameter Schemas**: Must match expected parameter types and formats
- **Java Version**: Application requires Java 25 specifically (upgraded from Java 24)

## Dependencies

### External Dependencies
- **ndevops-infrastructure**: Pipeline template repository
- **Azure Container Registry**: Image storage and retrieval
- **Eclipse Temurin**: Java 25 distribution source (upgraded from Java 24)
- **Docker Hub**: Base container images

### Build Dependencies
- **Gradle Dependencies**: Managed through build.gradle.kts
- **Docker Base Images**: eclipse-temurin:25-jre-alpine (upgraded from Java 24)
- **Test Dependencies**: CosmosDB emulator, Kafka, testcontainers

### Runtime Dependencies
- **Azure Cosmos DB**: Database service
- **Apache Kafka**: Message streaming
- **Azure Key Vault**: Secret management
- **Application Insights**: Monitoring and telemetry

## Tool Usage Patterns

### Gradle Configuration
```yaml
# Optimal Gradle Options
gradleTasks: 'clean build'
options: '-Dorg.gradle.parallel=true -Dorg.gradle.daemon=true -Dorg.gradle.caching=true --console verbose'
gradleOptions: '-Xmx4096m'
```

### Caching Strategy
```yaml
# Multi-layer Caching Implementation
enableGradleCache: true
gradleCachePath: '$(Pipeline.Workspace)/.gradle'
cacheDockerImages: true
dockerImageCacheKey: 'docker-images-$(Agent.OS)-$(SERVICE_NAME)-$(Build.SourceVersion)'
publishBuildArtifacts: true
restoreDockerCache: true
```

### Template Parameter Patterns
```yaml
# tests-java-v24.yaml Parameters (Java 25 Configuration - VERIFIED WORKING)
parameters:
  containerImage: 'eclipse-temurin:25'         # Java 25 container image
  jdkVersionOption: '1.25'                     # JDK version specification
  javaVersion: '25.0.0+11'                     # Specific Java 25 version
  gradleTasks: 'clean build'                   # Gradle command tasks
  gradleOptions: '-Xmx4096m'                   # JVM options
  options: '-Xmx4096m'                         # REQUIRED: Template validation parameter
  enableGradleBuild: true                      # Enable artifact building
  publishBuildArtifacts: true                  # Publish for reuse
  cacheDockerImages: true                      # Cache Docker images
  enableTests: true                            # REQUIRED: Enable test execution
  repositoryName: 'nfos-service-mashgin-fueling'  # REQUIRED: Repository identifier

# build-java-v5.yaml Parameters  
parameters:
  downloadBuildArtifacts: true                 # Download from test stage
  restoreDockerCache: true                     # Restore cached images
  buildArtifactName: 'build-artifacts'        # Artifact reference
```

### Java 25 Template Implementation Details
```yaml
# Custom Java 25 Installation Logic (Verified in tests-java-v24.yaml)
- task: Cache@2
  displayName: 'Cache Java 25 Installation'
  inputs:
    key: 'java25-$(Agent.OS)-openjdk25'
    path: '$(Agent.ToolsDirectory)/Java_Adopt_jdk/25.0.0-11/x64'
  condition: eq(variables['jdkVersionOption'], '1.25')

- script: |
    # Custom Java 25 installation with Eclipse Temurin
    if [ ! -d "$(Agent.ToolsDirectory)/Java_Adopt_jdk/25.0.0-11/x64" ]; then
      echo "Installing Java 25 from Eclipse Temurin..."
      # Download and extract Java 25
      # Set up JAVA_HOME and PATH
    fi
  displayName: 'Install Java 25 (Custom)'
  condition: eq(variables['jdkVersionOption'], '1.25')
```

## Architecture Decisions

### Template Selection
- **Decision**: Use `tests-java-v24.yaml` for both pipelines with Java 25 parameters
- **Rationale**: Provides comprehensive caching and **verified Java 25 support** through parameters
- **Alternative Considered**: `tests-java-v8.yaml` (limited caching)
- **Trade-offs**: More complex configuration but significantly better performance
- **Java 25 Implementation**: Template **verified to fully support Java 25** via:
  - containerImage, jdkVersionOption, and javaVersion parameters
  - Custom installation logic with Eclipse Temurin download
  - Java 25-specific caching mechanisms
  - Proper environment variable configuration
- **Parameter Validation**: **Fixed missing required parameters** (`enableTests`, `repositoryName`, `options`)

### Caching Architecture
- **Decision**: Multi-layer caching (Gradle + Java + Docker + Artifacts)
- **Implementation**: Separate cache keys for different artifact types
- **Benefits**: Maximum cache hit rates and minimal redundant work
- **Maintenance**: Cache keys include source version for proper invalidation

### Artifact Flow
- **Decision**: Build-once-use-everywhere pattern
- **Flow**: tests-java-v24 → publish artifacts → build-java-v5 → download artifacts
- **Benefits**: Eliminates duplicate builds, ensures consistency
- **Risk Mitigation**: Proper dependency management between pipeline jobs

### Parameter Organization
- **Decision**: Separate JVM options from Gradle options
- **Implementation**: `gradleOptions` for JVM, `options` for Gradle properties
- **Rationale**: Prevents "Unknown command-line option" errors
- **Best Practice**: Clear separation of concerns in parameter usage

## Performance Optimizations

### Build Performance
- **Parallel Execution**: `-Dorg.gradle.parallel=true`
- **Daemon Mode**: `-Dorg.gradle.daemon=true` 
- **Build Cache**: `-Dorg.gradle.caching=true`
- **Memory Allocation**: `-Xmx4096m` for adequate heap space

### Cache Performance
- **Gradle Cache**: Source version-based keys for precise invalidation
- **Docker Cache**: Image-specific caching with manifest files
- **Java Cache**: Installation caching to avoid repeated downloads
- **Artifact Cache**: Pipeline artifact reuse between stages

### Network Optimization
- **Docker Layer Cache**: Reduces image download times
- **Dependency Cache**: Gradle dependencies cached across builds
- **Base Image Cache**: Common images cached and reused

---
*Last Updated: September 25, 2025*
*Technical Status: Java 25 pipeline parameter fixes completed and template verification confirmed*
*Template Validation: tests-java-v24.yaml fully supports Java 25 with comprehensive installation, caching, and environment setup*
