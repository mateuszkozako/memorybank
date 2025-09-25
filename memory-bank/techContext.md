# Technical Context

## Technologies Used

### Core Technologies
- **Azure DevOps Pipelines**: CI/CD orchestration platform
- **Java 25**: Application runtime (Eclipse Temurin) - **UPGRADED FROM JAVA 24**
- **Gradle**: Build automation tool with wrapper (gradlew)
- **Docker**: Containerization platform
- **Azure Container Registry**: Docker image storage

### Pipeline Templates
- **tests-java-v24.yaml**: Advanced testing template with comprehensive caching
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
- **Java 25**: Requires custom installation as not pre-installed on agents (upgraded from Java 24)
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
# tests-java-v24.yaml Parameters (Java 25 Configuration)
parameters:
  containerImage: 'eclipse-temurin:25'         # Java 25 container image
  jdkVersionOption: '1.25'                     # JDK version specification
  javaVersion: '25.0.0+11'                     # Specific Java 25 version
  gradleTasks: 'clean build'                   # Gradle command tasks
  gradleOptions: '-Xmx4096m'                   # JVM options
  options: '-Dorg.gradle.parallel=true...'     # Gradle system properties
  enableGradleBuild: true                      # Enable artifact building
  publishBuildArtifacts: true                  # Publish for reuse
  cacheDockerImages: true                      # Cache Docker images

# build-java-v5.yaml Parameters  
parameters:
  downloadBuildArtifacts: true                 # Download from test stage
  restoreDockerCache: true                     # Restore cached images
  buildArtifactName: 'build-artifacts'        # Artifact reference
```

## Architecture Decisions

### Template Selection
- **Decision**: Use `tests-java-v24.yaml` for both pipelines with Java 25 parameters
- **Rationale**: Provides comprehensive caching and flexible Java version support through parameters
- **Alternative Considered**: `tests-java-v8.yaml` (limited caching)
- **Trade-offs**: More complex configuration but significantly better performance
- **Java 25 Implementation**: Template supports Java 25 via containerImage, jdkVersionOption, and javaVersion parameters

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
*Technical Status: Java 25 upgraded and validated*
