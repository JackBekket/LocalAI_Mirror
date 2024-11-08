## Package: gallery

This package provides functionality for installing and managing models from a gallery. It includes functions for finding available models, installing models, and scanning models for potential vulnerabilities.

### Imports:

- errors
- fmt
- os
- filepath
- strings
- dario.cat/mergo
- github.com/mudler/LocalAI/core/config
- github.com/mudler/LocalAI/pkg/downloader
- github.com/mudler/LocalAI/pkg/utils
- github.com/rs/zerolog/log
- gopkg.in/yaml.v2

### External Data, Input Sources:

- config.Gallery: A struct containing information about a model gallery, such as the URL and name.
- basePath: The base path where models will be installed.
- req: A struct containing information about the model to be installed, such as the name and overrides.
- downloadStatus: A function that is called to update the progress of the download.
- enforceScan: A boolean flag that determines whether to perform a safety scan on the model.

### Major Code Parts:

1. InstallModelFromGallery: This function installs a model from the gallery. It takes a list of galleries, the model name, the base path, the request schema, a download status function, and an enforce scan flag as input. It first finds the model in the galleries and then applies the model configuration.

2. FindModel: This function finds a model in a list of models based on the given name. It handles cases where the model name contains an "@" symbol, which indicates a model from a specific gallery.

3. AvailableGalleryModels: This function retrieves a list of models from the given galleries. It iterates through each gallery and calls the getGalleryModels function to retrieve the models from the gallery.

4. getGalleryModels: This function retrieves a list of models from a given gallery. It first checks if the gallery URL is a reference URL and if so, it resolves the reference URL to the actual gallery URL. Then, it downloads the gallery file and unmarshals the YAML data to create a list of models.

5. SafetyScanGalleryModels: This function scans all installed models for potential vulnerabilities. It iterates through each model and calls the SafetyScanGalleryModel function to perform the scan.

6. SafetyScanGalleryModel: This function scans a single model for potential vulnerabilities. It iterates through the additional files associated with the model and calls the HuggingFaceScan function to check for unsafe files.

7. DeleteModelFromSystem: This function deletes a model from the system. It first reads the model configuration and then removes all files associated with the model, including the model configuration file and any additional files.

8. ReadConfigFile: This function reads a YAML configuration file and returns a Config struct.

9. galleryFileName: This function generates the filename for a gallery based on the model name.

10. galleryFileName: This function generates the filename for a gallery based on the model name.



core/gallery/models.go
## Package: gallery

### Imports:

```
errors
fmt
os
path/filepath

dario.cat/mergo
github.com/mudler/LocalAI/core/config
github.com/mudler/LocalAI/pkg/downloader
github.com/mudler/LocalAI/pkg/utils

github.com/rs/zerolog/log
gopkg.in/yaml.v2
```

### External Data, Input Sources:

1. Config file: The package reads a YAML file containing model details, such as description, icon, license, URLs, name, config file, files, and prompt templates.

2. Downloadable files: The package downloads files specified in the config file, verifying their SHA256 checksums.

3. Prompt templates: The package reads and writes prompt templates to separate files.

4. Config overrides: The package accepts a map of config overrides to modify the model configuration.

### Major Code Parts:

#### 1. Config Structure:

The package defines a `Config` struct to store model details, including description, icon, license, URLs, name, config file, files, and prompt templates.

#### 2. File Structure:

The package defines a `File` struct to store file details, such as filename, SHA256 checksum, and URI.

#### 3. Prompt Template Structure:

The package defines a `PromptTemplate` struct to store prompt template details, including name and content.

#### 4. GetGalleryConfigFromURL Function:

This function downloads the model configuration from a given URL and unmarshals the YAML data into a `Config` struct.

#### 5. ReadConfigFile Function:

This function reads a YAML file containing model details and unmarshals the YAML data into a `Config` struct.

#### 6. InstallModel Function:

This function downloads files, verifies their SHA256 checksums, writes prompt templates to separate files, and writes the model configuration to a YAML file. It also accepts config overrides and enforces a scan for unsafe files.

#### 7. galleryFileName Function:

This function generates a filename for the model gallery file based on the model name.

core/gallery/op.go
package gallery

imports:
- github.com/mudler/LocalAI/core/config

external data:
- config.Gallery

summary:
### GalleryOp struct
The GalleryOp struct is used to represent an operation on a gallery. It contains the following fields:
- Id: A unique identifier for the operation.
- GalleryModelName: The name of the gallery model to be used for the operation.
- ConfigURL: The URL of the configuration file for the gallery model.
- Delete: A boolean flag indicating whether the operation is a deletion.
- Req: A GalleryModel struct containing the gallery model data.
- Galleries: A slice of config.Gallery structs containing the gallery data.

### GalleryOpStatus struct
The GalleryOpStatus struct is used to represent the status of a gallery operation. It contains the following fields:
- Deletion: A boolean flag indicating whether the operation is a deletion.
- FileName: The name of the file associated with the operation.
- Error: An error object containing any errors that occurred during the operation.
- Processed: A boolean flag indicating whether the operation has been processed.
- Message: A string containing a message about the operation.
- Progress: A float64 value representing the progress of the operation.
- TotalFileSize: A string containing the total file size.
- DownloadedFileSize: A string containing the downloaded file size.
- GalleryModelName: The name of the gallery model used for the operation.



core/gallery/request.go
package gallery

imports:
- fmt
- strings
- github.com/mudler/LocalAI/core/config

external data:
- config.Gallery

summary:
The code defines a struct called GalleryModel, which represents a model in a gallery. It includes fields for the model's URL, name, description, license, URLs, icon, tags, config file, overrides, additional files, gallery, and installed status.

The GalleryModel struct has a method called ID() that returns a string representation of the model's ID, which is a combination of the gallery name and the model name.

The code also defines a type called GalleryModels, which is a slice of GalleryModel structs. It has two methods: Search() and FindByName().

Search() takes a search term as input and returns a new slice of GalleryModel structs that match the search term in their name, description, gallery name, or tags.

FindByName() takes a model name as input and returns a pointer to the GalleryModel struct that matches the name, or nil if no match is found.

]

%!s(MISSING)
%!s(MISSING)