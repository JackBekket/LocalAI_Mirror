# pkg/templates/cache.go  
## Package: templates  
  
### Imports:  
  
* bytes  
* fmt  
* os  
* path/filepath  
* sync  
* text/template  
* github.com/mudler/LocalAI/pkg/utils  
* github.com/Masterminds/sprig/v3  
  
### External Data, Input Sources:  
  
* templatesPath: A string representing the path to the directory containing the template files.  
  
### Summary:  
  
#### TemplateCache:  
  
The `TemplateCache` struct is responsible for managing a cache of templates. It has a mutex to ensure thread safety, a templatesPath string to store the path to the directory containing the template files, and a map of templates. The map is keyed by TemplateType and maps to another map where the keys are template names and the values are pointers to template.Template objects.  
  
#### NewTemplateCache:  
  
The `NewTemplateCache` function creates a new instance of the `TemplateCache` struct and initializes the templatesPath field with the provided value.  
  
#### initializeTemplateMapKey:  
  
The `initializeTemplateMapKey` function checks if a map for the given TemplateType exists in the templates map. If not, it creates a new map for that TemplateType.  
  
#### EvaluateTemplate:  
  
The `EvaluateTemplate` function takes a TemplateType, templateName, and an interface as input. It locks the mutex, initializes the template map key, and checks if the template exists in the cache. If not, it calls the `loadTemplateIfExists` function to load the template. If the template is found, it executes the template with the provided input and returns the output as a string.  
  
#### loadTemplateIfExists:  
  
The `loadTemplateIfExists` function checks if the template already exists in the cache. If not, it constructs the path to the template file, verifies the path, and reads the template content from the file or uses the provided templateName as the template content. It then parses the template and stores it in the cache.  
  
#### Summary:  
  
The `templates` package provides a way to manage and evaluate templates. It uses a TemplateCache struct to store templates and their corresponding data. The package includes functions to create a new TemplateCache, initialize template map keys, evaluate templates, and load templates from files or strings.  
  
# pkg/templates/multimodal.go  
## Package: templates  
  
### Imports:  
- bytes  
- text/template  
- github.com/Masterminds/sprig/v3  
  
### External Data, Input Sources:  
- MultiModalOptions struct:  
    - TotalImages: int  
    - TotalAudios: int  
    - TotalVideos: int  
    - ImagesInMessage: int  
    - AudiosInMessage: int  
    - VideosInMessage: int  
- MultimodalContent struct:  
    - ID: int  
- templateString: string  
- text: string  
  
### Summary:  
#### TemplateMultiModal function:  
This function takes a template string, MultiModalOptions struct, and text as input. It first checks if the template string is empty, and if so, it uses a default template. Then, it compiles the template using the sprig library.  
  
Next, it creates three slices of MultimodalContent structs: audios, images, and videos. These slices are populated with IDs based on the values in the MultiModalOptions struct.  
  
A new bytes.Buffer is created to store the output. The compiled template is executed with the provided data, and the output is written to the buffer. Finally, the function returns the output as a string and any error encountered during the process.  
  