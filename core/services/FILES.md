# core/services/backend_monitor.go  
## services/backend_monitor.go  
  
This file contains the implementation of the `BackendMonitorService` which is responsible for monitoring the status and performance of local backend models.  
  
### Imports  
  
```go  
import (  
	"context"  
	"fmt"  
	"strings"  
  
	"github.com/mudler/LocalAI/core/config"  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/grpc/proto"  
	"github.com/mudler/LocalAI/pkg/model"  
  
	"github.com/rs/zerolog/log"  
  
	gopsutil "github.com/shirou/gopsutil/v3/process"  
)  
```  
  
### External Data, Input Sources  
  
The `BackendMonitorService` relies on the following external data and input sources:  
  
1. `config.BackendConfigLoader`: This interface is used to retrieve backend configuration data for a given model name.  
2. `model.ModelLoader`: This interface is used to load and manage backend models. It provides methods for checking if a model is loaded, getting the GRPC address of a loaded model, and shutting down a model.  
3. `config.ApplicationConfig`: This struct contains application-level configuration options, which may be used to inspect external GRPC backends.  
  
### Major Code Parts  
  
1. `BackendMonitorService` struct: This struct holds the necessary dependencies for the service, including the `backendConfigLoader`, `modelLoader`, and `options`.  
  
2. `NewBackendMonitorService` function: This function creates a new instance of the `BackendMonitorService` by initializing the dependencies.  
  
3. `getModelLoaderIDFromModelName` function: This function takes a model name as input and returns the corresponding backend ID. It first checks if a backend configuration exists for the given model name. If it does, it uses the backend ID from the configuration. Otherwise, it uses the model name as the backend ID.  
  
4. `SampleLocalBackendProcess` function: This function takes a model name as input and returns a `schema.BackendMonitorResponse` containing memory and CPU usage information for the corresponding backend process. It first retrieves the backend ID using the `getModelLoaderIDFromModelName` function. Then, it uses the `modelLoader` to get the GRPC PID of the backend process. It then uses the `gopsutil` library to retrieve memory and CPU usage information for the process.  
  
5. `CheckAndSample` function: This function takes a model name as input and returns a `proto.StatusResponse` containing the status of the backend model. It first retrieves the backend ID using the `getModelLoaderIDFromModelName` function. Then, it checks if the model is loaded using the `modelLoader`. If the model is loaded, it retrieves the status information using the GRPC API. If the model is not loaded or the status retrieval fails, it calls the `SampleLocalBackendProcess` function to get local process information and returns a `proto.StatusResponse` with the status set to `ERROR`.  
  
6. `ShutdownModel` function: This function takes a model name as input and shuts down the corresponding backend model using the `modelLoader`.  
  
  
  
# core/services/gallery.go  
## services/gallery.go  
  
This file contains the implementation of the GalleryService, which is responsible for managing and updating models in the LocalAI system.  
  
### Imports  
  
* context  
* encoding/json  
* fmt  
* os  
* path/filepath  
* sync  
* github.com/mudler/LocalAI/core/config  
* github.com/mudler/LocalAI/core/gallery  
* github.com/mudler/LocalAI/pkg/startup  
* github.com/mudler/LocalAI/pkg/utils  
* gopkg.in/yaml.v2  
  
### External Data, Input Sources  
  
* config.ApplicationConfig: This struct contains the application configuration, including the model path and other relevant settings.  
* galleries: A list of config.Gallery structs, which represent the available models in the system.  
* requests: A slice of galleryModel structs, which contain the information about the models to be installed or updated.  
  
### Code Summary  
  
1. GalleryService: This struct represents the service responsible for managing the gallery models. It has a mutex to ensure thread safety, a channel to receive gallery operations, and a map to store the status of each operation.  
  
2. NewGalleryService: This function creates a new instance of the GalleryService, initializing the mutex, channel, and status map.  
  
3. prepareModel: This function prepares a model for installation by downloading the necessary files and configuring the model parameters. It takes the model path, the gallery model, a download status function, and an enforce scan flag as input.  
  
4. UpdateStatus, GetStatus, GetAllStatus: These functions allow for updating, retrieving, and retrieving all the status of the gallery operations.  
  
5. Start: This function starts the gallery service and processes the gallery operations received through the channel. It updates the status of each operation and handles any errors that may occur during the process.  
  
6. galleryModel: This struct represents a model to be installed or updated. It contains the GalleryModel struct and an ID field.  
  
7. processRequests: This function processes a list of gallery models, installing or updating them based on the provided information.  
  
8. ApplyGalleryFromFile, ApplyGalleryFromString: These functions apply gallery models from a file or a string, respectively. They parse the input data, process the models, and install or update them in the system.  
  
# core/services/list_models.go  
## Package: services  
  
### Imports:  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
- bcl: BackendConfigLoader  
- ml: ModelLoader  
- filter: config.BackendConfigFilterFn  
- looseFilePolicy: LooseFilePolicy  
  
### ListModels Function:  
The ListModels function takes a BackendConfigLoader, ModelLoader, BackendConfigFilterFn, and LooseFilePolicy as input. It returns a list of model names and an error.  
  
The function first initializes a map called skipMap to store model names that should be skipped. It then initializes an empty slice called dataModels to store the list of model names.  
  
The function then iterates through the backend configurations filtered by the provided filter function. For each configuration, it checks the looseFilePolicy and updates the skipMap accordingly. If the looseFilePolicy is SKIP_IF_CONFIGURED or LOOSE_ONLY, the model name is added to the skipMap. If the looseFilePolicy is not LOOSE_ONLY, the model name is appended to the dataModels slice.  
  
Next, the function iterates through the loose files in the model path using the ModelLoader. For each loose file, it checks if the file should be skipped based on the skipMap and the filter function. If the file should not be skipped, it is appended to the dataModels slice.  
  
Finally, the function returns the dataModels slice and nil error.  
  
# core/services/metrics.go  
## Package: services  
  
### Imports:  
  
```  
"context"  
"github.com/rs/zerolog/log"  
"go.opentelemetry.io/otel/attribute"  
"go.opentelemetry.io/otel/exporters/prometheus"  
"go.opentelemetry.io/otel/metric"  
"go.opentelemetry.io/otel/sdk/metric"  
```  
  
### External Data and Input Sources:  
  
- Prometheus exporter for OpenTelemetry metrics.  
  
### Code Summary:  
  
#### LocalAIMetricsService:  
  
This struct is responsible for collecting and exporting metrics related to API calls. It has two fields:  
  
1. Meter: A metric.Meter object from OpenTelemetry SDK, used for creating and managing metrics.  
2. ApiTimeMetric: A metric.Float64Histogram object, used to track the duration of API calls.  
  
#### ObserveAPICall:  
  
This method is used to record the duration of an API call. It takes the method, path, and duration of the call as input. It then uses the ApiTimeMetric to record the duration with the specified attributes.  
  
#### NewLocalAIMetricsService:  
  
This function creates a new instance of LocalAIMetricsService. It first creates a Prometheus exporter for OpenTelemetry metrics. Then, it creates a meter provider using the exporter and a meter with the name "github.com/mudler/LocalAI". Finally, it creates a float64 histogram metric called "api_call" and returns a new LocalAIMetricsService instance with the meter and the api_call metric.  
  
#### Shutdown:  
  
This method is intended to shut down the OpenTelemetry pipeline, but the actual implementation is not yet available. It logs a warning message indicating that the shutdown functionality is not yet implemented.  
  
  
  
