## Package: localai

### Imports:

- github.com/gofiber/fiber/v2
- github.com/mudler/LocalAI/core/backend
- github.com/mudler/LocalAI/core/config
- github.com/mudler/LocalAI/core/http/ctx
- github.com/mudler/LocalAI/core/schema
- github.com/mudler/LocalAI/pkg/model
- github.com/rs/zerolog/log

### External Data, Input Sources:

- The code uses a configuration file to load backend settings, model paths, and other parameters.
- It also uses a model loader to load the model files.
- The input data is received from the request body in JSON format.

### Code Summary:

#### BackendMonitorEndpoint:

This function returns the status of the specified backend. It takes a pointer to a `services.BackendMonitorService` as input. The function first parses the request body into a `schema.BackendMonitorRequest` struct. Then, it calls the `CheckAndSample` method of the provided backend monitor service, passing the model from the input struct. The result of this method is returned as a JSON response.

#### BackendShutdownEndpoint:

This function shuts down the specified backend. It also takes a pointer to a `services.BackendMonitorService` as input. Similar to the previous function, it first parses the request body into a `schema.BackendMonitorRequest` struct. Then, it calls the `ShutdownModel` method of the provided backend monitor service, passing the model from the input struct. The result of this method is returned as a JSON response.

#### GetOpStatusEndpoint:

This function returns the status of a specific job. It takes a UUID as input and returns the corresponding job status.

#### GetAllStatusEndpoint:

This function returns the status of all jobs. It returns a map of UUIDs to job statuses.

#### ApplyModelGalleryEndpoint:

This function installs a new model to a LocalAI instance from the model gallery. It takes a GalleryModel as input and returns a schema.GalleryResponse containing the ID and status URL of the job.

#### DeleteModelGalleryEndpoint:

This function deletes a model from a LocalAI instance. It takes the model name as input and returns a schema.GalleryResponse containing the ID and status URL of the job.

#### ListModelFromGalleryEndpoint:

This function lists the available models for installation from the active galleries. It returns a list of gallery.GalleryModel objects.

#### ListModelGalleriesEndpoint:

This function lists the available galleries configured in LocalAI. It returns a list of config.Gallery objects.

#### AddModelGalleryEndpoint:

This function adds a new gallery to LocalAI. It takes a config.Gallery object as input and returns a list of config.Gallery objects.

#### RemoveModelGalleryEndpoint:

This function removes a gallery from LocalAI. It takes a config.Gallery object as input and returns a list of config.Gallery objects.

#### LocalAIMetricsEndpoint:

This function returns the metrics endpoint for LocalAI. It uses the promhttp.Handler() to provide the metrics endpoint.

#### apiMiddlewareConfig:

This struct defines the configuration for the API middleware. It has two fields: Filter and metricsService. The Filter field is a function that takes a fiber.Ctx as input and returns a boolean value. The metricsService field is a pointer to a services.LocalAIMetricsService.

#### LocalAIMetricsAPIMiddleware:

This function takes a pointer to a services.LocalAIMetricsService as input and returns a fiber.Handler. It creates an instance of the apiMiddlewareConfig struct and sets the Filter field to a function that checks if the current path is "/metrics". The function then returns a new fiber.Handler that wraps the next handler in the chain and observes the API call metrics.

#### ShowP2PNodes:

This function returns the available P2P nodes. It takes an instance of config.ApplicationConfig as input. The function returns a Fiber handler that, when called, returns a JSON response containing two arrays: one for the available nodes in the worker network and another for the available nodes in the federated network. The data for these arrays is obtained from the p2p package using the provided network ID and worker/federated ID.

#### ShowP2PToken:

This function returns the P2P token. It takes an instance of config.ApplicationConfig as input. The function returns a Fiber handler that, when called, sends the P2P token as a byte array in the response. The token is obtained from the config.ApplicationConfig struct.

#### StoresSetEndpoint:

This function handles the endpoint for setting values in a store. It takes a model loader and application configuration as input. The function parses the request body, which is expected to be a schema.StoresSet object. It then calls the backend.StoreBackend function to get a store backend object. Next, it converts the input values to byte arrays and calls the store.SetCols function to set the values in the store. Finally, it sends a response back to the client.

#### StoresDeleteEndpoint:

This function handles the endpoint for deleting values from a store. It takes a model loader and application configuration as input. The function parses the request body, which is expected to be a schema.StoresDelete object. It then calls the backend.StoreBackend function to get a store backend object. Next, it calls the store.DeleteCols function to delete the specified keys from the store. Finally, it sends a response back to the client.

#### StoresGetEndpoint:

This function handles the endpoint for retrieving values from a store. It takes a model loader and application configuration as input. The function parses the request body, which is expected to be a schema.StoresGet object. It then calls the backend.StoreBackend function to get a store backend object. Next, it calls the store.GetCols function to retrieve the values for the specified keys from the store. Finally, it sends a JSON response back to the client containing the retrieved keys and values.

#### StoresFindEndpoint:

This function handles the endpoint for finding similar values in a store. It takes a model loader and application configuration as input. The function parses the request body, which is expected to be a schema.StoresFind object. It then calls the backend.StoreBackend function to get a store backend object. Next, it calls the store.Find function to find the topk most similar values to the specified key in the store. Finally, it sends a JSON response back to the client containing the retrieved keys, values, and similarities.

#### SystemInformations:

This function returns the system information of the LocalAI instance. It takes two arguments: ml (Model loader object) and appConfig (Application configuration object).

1. It first lists the available backends using ml.ListAvailableBackends(appConfig.AssetsDestination).
2. Then, it lists the loaded models using ml.ListModels().
3. It appends the external GRPC backends from appConfig.ExternalGRPCBackends to the available backends.
4. Finally, it returns a JSON response containing the available backends and loaded models in a schema.SystemInformationResponse object.

#### TokenizeEndpoint:

This function exposes a REST API to tokenize the content. It takes three arguments:

1. `cl`: A pointer to a config.BackendConfigLoader.
2. `ml`: A pointer to a model.ModelLoader.
3. `appConfig`: A pointer to a config.ApplicationConfig.

The function first creates a new schema.TokenizeRequest object to store the input data. Then, it parses the request body and stores the input data in the schema.TokenizeRequest object.

Next, the function retrieves the model file from the context using the fiberContext.ModelFromContext function. If the model is not found in the context, it uses the input.Model field.

The function then loads the backend configuration file using the cl.LoadBackendConfigFileByName function. The configuration file is loaded with the specified options, such as debug mode, number of threads, context size, and F16 support.

After loading the configuration file, the function calls the backend.ModelTokenize function to tokenize the input content using the loaded model and configuration. The result is stored in a tokenResponse object.

Finally, the function returns the tokenResponse as a JSON response using the c.JSON function.

#### WelcomeEndpoint:

This function defines a welcome endpoint for the LocalAI API. It takes the application configuration, backend configuration loader, model loader, and a function to get model statuses as input. The function returns a function that can be used as a handler for the welcome endpoint.

The handler function first retrieves all backend configurations and then iterates through them to get the local model configuration for each backend. It also lists models without configuration and gets the model statuses to display in the UI.

Finally, it creates a summary map containing the title, version, models, models configurations, gallery configurations, P2P status, application configuration, processing models, and task types. Based on the client's content type, it either returns a JSON response with the summary or renders the index view with the summary data.

