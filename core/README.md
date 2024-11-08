# Package: core

This package provides the core functionality for the application, including managing services, configuration, and application state.

### Imports:
- github.com/mudler/LocalAI/core/config
- github.com/mudler/LocalAI/core/services
- github.com/mudler/LocalAI/pkg/model

### External Data, Input Sources:
- ApplicationConfig: Application-Level Config
- BackendConfigLoader: Core Low-Level Services
- ModelLoader: Core Low-Level Services
- BackendMonitorService: LocalAI System Services
- GalleryService: LocalAI System Services
- LocalAIMetricsService: LocalAI System Services

### Application Structure:
The Application struct is designed to hold pointers to all initialized services, simplifying the plumbing process. It includes application-level configuration, core low-level services, and local AI system services.

### ApplicationState:
The ApplicationState struct is a placeholder for future implementation, potentially holding runtime information not set via configuration.

### Services:
- BackendMonitorService: Monitors backend services.
- GalleryService: Manages image galleries.
- LocalAIMetricsService: Tracks LocalAI system metrics.

### TODO:
- Break up ApplicationConfig and migrate non-config-based runtime information.

