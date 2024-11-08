## Package: startup

### Imports:

```
encoding/json
fmt
os
path
path/filepath
time

github.com/fsnotify/fsnotify
dario.cat/mergo
github.com/mudler/LocalAI/core/config
github.com/rs/zerolog/log
```

### External Data, Input Sources:

1. `api_keys.json`: Contains a list of API keys to be used by the application.
2. `external_backends.json`: Contains a map of external GRPC backends to be used by the application.

### Major Code Parts:

#### 1. Config File Handler:

The `configFileHandler` struct is responsible for managing the registration and execution of handlers for dynamic configuration files. It maintains a map of file handlers, a watcher for file changes, and an instance of the application configuration.

#### 2. Registering Config File Handlers:

The `Register` method allows for the registration of file handlers for specific configuration files. It takes the filename, the handler function, and a boolean flag indicating whether to execute the handler immediately.

#### 3. Calling Config File Handlers:

The `callHandler` method is responsible for executing the registered file handlers. It reads the content of the specified configuration file, and calls the corresponding handler function, passing the file content and the application configuration as arguments.

#### 4. Watching for Configuration Changes:

The `Watch` method sets up a file watcher to monitor changes in the specified configuration directory. It uses the `fsnotify` package to watch for file changes, and triggers the execution of the corresponding file handlers when changes are detected.

#### 5. Stopping the Watcher:

The `Stop` method closes the file watcher, allowing for graceful shutdown of the application.

#### 6. Dynamic Configuration File Handlers:

Two specific file handlers are provided: `readApiKeysJson` and `readExternalBackendsJson`. These handlers parse the content of the `api_keys.json` and `external_backends.json` files, respectively, and update the application configuration accordingly.

#### 7. Graceful Shutdown:

The code includes a TODO comment indicating that a graceful shutdown mechanism should be implemented, which would include calling the `Stop` method on the config file handler to close the file watcher.



core/startup/startup.go
```
package startup

imports:
- fmt
- os
- github.com/mudler/LocalAI/core
- github.com/mudler/LocalAI/core/backend
- github.com/mudler/LocalAI/core/config
- github.com/mudler/LocalAI/core/services
- github.com/mudler/LocalAI/internal
- github.com/mudler/LocalAI/pkg/assets
- github.com/mudler/LocalAI/pkg/library
- github.com/mudler/LocalAI/pkg/model
- github.com/mudler/LocalAI/pkg/startup
- github.com/mudler/LocalAI/pkg/xsysinfo
- github.com/rs/zerolog/log

external data:
- config.AppOption
- config.BackendConfigLoader
- model.ModelLoader
- config.ApplicationConfig
- config.LoadOptionDebug
- config.LoadOptionThreads
- config.LoadOptionContextSize
- config.LoadOptionF16
- config.ModelPath
- model.Option
- backend.ModelOptions
- backend.EmbeddingsBackendService
- backend.ImageGenerationBackendService
- backend.LLMBackendService
- backend.TranscriptionBackendService
- backend.TextToSpeechBackendService
- services.BackendMonitorService
- services.GalleryService
- services.OpenAIService
- services.LocalAIMetricsService

summary:
Startup function initializes the LocalAI application by loading models, configuring backends, and setting up services. It takes an optional list of configuration options and returns a BackendConfigLoader, ModelLoader, ApplicationConfig, and an error.

1. It first logs the number of threads and the model path being used.
2. It then retrieves CPU and GPU information and logs it.
3. It creates the necessary directories for models, images, audio, and uploads.
4. It installs models from the specified galleries, model library URL, and model path.
5. It loads backend configurations from the model path and a specified configuration file.
6. It preloads models from JSON and file paths if provided.
7. It extracts backend assets from the embedded file system if a destination is specified.
8. It loads external libraries from a specified path.
9. It starts a goroutine to shut down all GRPC backends when the context is canceled.
10. If a watchdog is enabled, it starts a watchdog goroutine to monitor the application.
11. It loads models into memory if a list of model names is provided.
12. It starts a watcher to monitor the dynamic configuration directory.
13. Finally, it creates an Application instance and returns it.

The Startup function is responsible for initializing the LocalAI application and setting up all the necessary components for it to function correctly.

pkg/startup/model_preload.go
## Package: startup

### Imports:

```
errors
fmt
os
path/filepath
strings

github.com/mudler/LocalAI/core/config
github.com/mudler/LocalAI/core/gallery
github.com/mudler/LocalAI/embedded
github.com/mudler/LocalAI/pkg/downloader
github.com/mudler/LocalAI/pkg/utils
github.com/rs/zerolog/log
```

### External Data, Input Sources:

1. Galleries: A list of config.Gallery objects, which represent model galleries.
2. Model Library URL: A string representing the URL of a remote library that contains model configurations.
3. Model Path: A string representing the directory where models will be installed.
4. Enforce Scan: A boolean flag indicating whether to enforce model scanning.
5. Download Status: A function that takes four arguments: filename, current, total, and percent, and is used to display download progress.
6. Models: A slice of strings representing the names of models to install.

### Summary:

#### InstallModels Function:

The InstallModels function takes a list of galleries, a model library URL, a model path, an enforce scan flag, a download status function, and a slice of model names as input. It iterates through the model names and attempts to install each model.

1. It first tries to resolve the model from the remote library, if a model library URL is provided. If the model is found in the library, it downloads the model configuration and saves it to the specified model path.

2. If the model is not found in the remote library, it checks if the model is an OCI image. If it is, it downloads the OCI image and saves it to the specified model path.

3. If the model is not an OCI image, it checks if the model is a URL. If it is, it downloads the model from the URL and saves it to the specified model path.

4. If the model is not a URL, it checks if it's a local file. If it is, it copies the model to the specified model path.

5. If none of the above conditions are met, it calls the installModel function to attempt to install the model from a gallery.

6. Finally, it returns an error if any errors occurred during the installation process.

#### installModel Function:

The installModel function takes a list of galleries, a model name, a model path, a download status function, and an enforce scan flag as input. It first retrieves a list of available models from the galleries and finds the specified model. If the model is found, it installs the model from the gallery using the provided download status function and enforce scan flag.

7. It returns an error if any errors occurred during the installation process.

]

