## Package: embedded

### Imports:

* embed
* fmt
* slices
* strings
* github.com/mudler/LocalAI/pkg/downloader
* github.com/rs/zerolog/log
* github.com/mudler/LocalAI/pkg/assets
* gopkg.in/yaml.v3

### External Data, Input Sources:

* model_library.yaml (embedded)
* models/* (embedded)

### Code Summary:

The embedded package provides functions for managing and accessing a library of models. It uses a YAML file named `model_library.yaml` to store information about the models, including their short URLs. The package also includes a directory named `models` that contains YAML files for each model, providing additional details about the model.

#### Model Short URL:

This function takes a model name as input and returns its short URL. It first checks if the model name already has a short URL stored in the `modelShorteners` map. If it does, it returns the short URL. Otherwise, it returns the original model name.

#### Model Shortener Initialization:

This function initializes the `modelShorteners` map by unmarshalling the data from the embedded `model_library.yaml` file. If there is an error during unmarshalling, it logs an error message.

#### Get Remote Library Shorteners:

This function downloads a remote YAML file containing model short URLs and returns a map of model names to short URLs. It takes the URL of the remote library and a base path as input. It downloads the file using the `downloader.URI` and `DownloadWithCallback` functions, and then unmarshals the YAML data into the `remoteLibrary` map. If there is an error during downloading or unmarshalling, it returns an error.

#### Exists In Models Library:

This function checks if a model exists in the embedded models library. It takes a model name as input and returns a boolean value indicating whether the model exists. It first constructs the expected file name for the model in the embedded library and then checks if the file exists in the embedded models library using the `assets.ListFiles` function.

#### Resolve Content:

This function returns the content of a model from the embedded models library. It takes a model name as input and returns the content as a byte slice. If the model exists in the embedded library, it reads the content from the corresponding YAML file. Otherwise, it returns an error.

