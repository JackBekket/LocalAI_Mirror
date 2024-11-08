# pkg/model/filters.go  
## Package: model  
  
### Imports:  
- process "github.com/mudler/go-processmanager"  
  
### External Data and Input Sources:  
- None  
  
### GRPCProcessFilter:  
- GRPCProcessFilter is a type alias for a function that takes a process ID (string) and a process object (process.Process) as input and returns a boolean value.  
  
### all:  
- This function takes a string and a process object as input and returns true for all processes.  
  
### allExcept:  
- This function takes a string as input and returns a GRPCProcessFilter that excludes the specified process ID from the filter.  
  
# pkg/model/initializers.go  
```  
package model  
  
import (  
	"context"  
	"errors"  
	"fmt"  
	"os"  
	"path/filepath"  
	"slices"  
	"strings"  
	"time"  
  
	"github.com/klauspost/cpuid/v2"  
	grpc "github.com/mudler/LocalAI/pkg/grpc"  
	"github.com/mudler/LocalAI/pkg/library"  
	"github.com/mudler/LocalAI/pkg/utils"  
	"github.com/mudler/LocalAI/pkg/xsysinfo"  
	"github.com/rs/zerolog/log"  
  
	"github.com/elliotchance/orderedmap/v2"  
)  
  
var Aliases map[string]string = map[string]string{  
	"go-llama":              LLamaCPP,  
	"llama":                 LLamaCPP,  
	"embedded-store":        LocalStoreBackend,  
	"langchain-huggingface": LCHuggingFaceBackend,  
}  
  
var AutoDetect = os.Getenv("DISABLE_AUTODETECT") != "true"  
  
const (  
	LlamaGGML = "llama-ggml"  
  
	LLamaCPP = "llama-cpp"  
  
	LLamaCPPAVX2     = "llama-cpp-avx2"  
	LLamaCPPAVX      = "llama-cpp-avx"  
	LLamaCPPFallback = "llama-cpp-fallback"  
	LLamaCPPCUDA     = "llama-cpp-cuda"  
	LLamaCPPHipblas  = "llama-cpp-hipblas"  
	LLamaCPPSycl16   = "llama-cpp-sycl_16"  
	LLamaCPPSycl32   = "llama-cpp-sycl_32"  
  
	LLamaCPPGRPC = "llama-cpp-grpc"  
  
	BertEmbeddingsBackend  = "bert-embeddings"  
	RwkvBackend            = "rwkv"  
	WhisperBackend         = "whisper"  
	StableDiffusionBackend = "stablediffusion"  
	TinyDreamBackend       = "tinydream"  
	PiperBackend           = "piper"  
	LCHuggingFaceBackend   = "huggingface"  
  
	LocalStoreBackend = "local-store"  
)  
  
func backendPath(assetDir, backend string) string {  
	return filepath.Join(assetDir, "backend-assets", "grpc", backend)  
}  
  
// backendsInAssetDir returns the list of backends in the asset directory  
// that should be loaded  
func backendsInAssetDir(assetDir string) (map[string][]string, error) {  
	// Exclude backends from automatic loading  
	excludeBackends := []string{LocalStoreBackend}  
	entry, err := os.ReadDir(backendPath(assetDir, ""))  
	if err != nil {  
		return nil, err  
	}  
	backends := make(map[string][]string)  
	for _, e := range entry {  
		for _, exclude := range excludeBackends {  
			if e.Name() == exclude {  
				continue  
			}  
		}  
		if e.IsDir() {  
			continue  
		}  
		if strings.HasSuffix(e.Name(), ".log") {  
			continue  
		}  
  
		// Skip the llama.cpp variants if we are autoDetecting  
		// But we always load the fallback variant if it exists  
		if strings.Contains(e.Name(), LLamaCPP) && !strings.Contains(e.Name(), LLamaCPPFallback) && AutoDetect {  
			continue  
		}  
  
		backends[e.Name()] = []string{}  
	}  
  
	return backends, nil  
}  
  
func orderBackends(backends map[string][]string) ([]string, error) {  
	// sets a priority list - first has more priority  
	priorityList := []string{  
		// First llama.cpp(variants) and llama-ggml to follow.  
		// We keep the fallback to prevent that if the llama.cpp variants  
		// that depends on shared libs if breaks have still a safety net.  
		LLamaCPP, LlamaGGML, LLamaCPPFallback,  
	}  
  
	toTheEnd := []string{  
		// last has to be huggingface  
		LCHuggingFaceBackend,  
		// then bert embeddings  
		BertEmbeddingsBackend,  
	}  
  
	// create an ordered map  
	orderedBackends := orderedmap.NewOrderedMap[string, any]()  
	// add priorityList first  
	for _, p := range priorityList {  
		if _, ok := backends[p]; ok {  
			orderedBackends.Set(p, backends[p])  
		}  
	}  
  
	for k, v := range backends {  
		if !slices.Contains(toTheEnd, k) {  
			if _, ok := orderedBackends.Get(k); !ok {  
				orderedBackends.Set(k, v)  
			}  
		}  
	}  
  
	for _, t := range toTheEnd {  
		if _, ok := backends[t]; ok {  
			orderedBackends.Set(t, backends[t])  
		}  
	}  
  
	return orderedBackends.Keys(), nil  
}  
  
// selectGRPCProcessByHostCapabilities selects the GRPC process to start based on system capabilities  
// Note: this is now relevant only for llama.cpp  
func selectGRPCProcessByHostCapabilities(backend, assetDir string, f16 bool) string {  
	foundCUDA := false  
	foundAMDGPU := false  
	foundIntelGPU := false  
	var grpcProcess string  
  
	// Select backend now just for llama.cpp  
	if backend != LLamaCPP {  
		return ""  
	}  
  
	// Check for GPU-binaries that are shipped with single binary releases  
	gpus, err := xsysinfo.GPUs()  
	if err == nil {  
		for _, gpu := range gpus {  
			if strings.Contains(gpu.String(), "nvidia") {  
				p := backendPath(assetDir, LLamaCPPCUDA)  
				if _, err := os.Stat(p); err == nil {  
					log.Info().Msgf("[%s] attempting to load with CUDA variant", backend)  
					grpcProcess = p  
					foundCUDA = true  
				} else {  
					log.Debug().Msgf("Nvidia GPU device found, no embedded CUDA variant found. You can ignore this message if you are using container with CUDA support")  
				}  
			}  
			if strings.Contains(gpu.String(), "amd") {  
				p := backendPath(assetDir, LLamaCPPHipblas)  
				if _, err := os.Stat(p); err == nil {  
					log.Info().Msgf("[%s] attempting to load with HIPBLAS variant", backend)  
					grpcProcess = p  
					foundAMDGPU = true  
				} else {  
					log.Debug().Msgf("AMD GPU device found, no embedded HIPBLAS variant found. You can ignore this message if you are using container with HIPBLAS support")  
				}  
			}  
			if strings.Contains(gpu.String(), "intel") {  
				backend := LLamaCPPSycl16  
				if !f16 {  
					backend = LLamaCPPSycl32  
				}  
				p := backendPath(assetDir, backend)  
				if _, err := os.Stat(p); err == nil {  
					log.Info().Msgf("[%s] attempting to load with Intel variant", backend)  
					grpcProcess = p  
					foundIntelGPU = true  
				} else {  
					log.Debug().Msgf("Intel GPU device found, no embedded SYCL variant found. You can ignore this message if you are using container with SYCL support")  
				}  
			}  
		}  
	}  
  
	if foundCUDA || foundAMDGPU || foundIntelGPU {  
		return grpcProcess  
	}  
  
	// IF we find any optimized binary, we use that  
	if xsysinfo.HasCPUCaps(cpuid.AVX2) {  
		p := backendPath(assetDir, LLamaCPPAVX2)  
		if _, err := os.Stat(p); err == nil {  
			log.Info().Msgf("[%s] attempting to load with AVX2 variant", backend)  
			grpcProcess = p  
		}  
	} else if xsysinfo.HasCPUCaps(cpuid.AVX) {  
		p := backendPath(assetDir, LLamaCPPAVX)  
		if _, err := os.Stat(p); err == nil {  
			log.Info().Msgf("[%s] attempting to load with AVX variant", backend)  
			grpcProcess = p  
		}  
	}  
  
	// Safety measure: check if the binary exists otherwise return empty string  
	if _, err := os.Stat(grpcProcess); err == nil {  
		return grpcProcess  
	}  
  
	return ""  
}  
  
// starts the grpcModelProcess for the backend, and returns a grpc client  
// It also loads the model  
func (ml *ModelLoader) grpcModel(backend string, autodetect bool, o *Options) func(string, string, string) (*Model, error) {  
	return func(modelID, modelName, modelFile string) (*Model, error) {  
  
		log.Debug().Msgf("Loading Model %s with gRPC (file: %s) (backend: %s): %+v", modelID, modelFile, backend, *o)  
  
		var client *Model  
  
		getFreeAddress := func() (string, error) {  
			port, err := freeport.GetFreePort()  
			if err != nil {  
				return "", fmt.Errorf("failed allocating free ports: %s", err.Error())  
			}  
			return fmt.Sprintf("127.0.0.1:%d", port), nil  
		}  
  
		// If no specific model path is set for transformers/HF, set it to the  
  
# pkg/model/loader.go  
## Package: model  
  
### Imports:  
  
```  
"context"  
"fmt"  
"os"  
"path/filepath"  
"strings"  
"sync"  
"time"  
  
"github.com/mudler/LocalAI/pkg/templates"  
  
"github.com/mudler/LocalAI/pkg/utils"  
  
"github.com/rs/zerolog/log"  
```  
  
### External Data, Input Sources:  
  
- `ModelPath`: Path to the directory containing the models.  
- `knownFilesToSkip`: List of file names to skip when listing models.  
- `knownModelsNameSuffixToSkip`: List of file name suffixes to skip when listing models.  
- `retryTimeout`: Timeout for retrying to shutdown a model.  
  
### ModelLoader:  
  
The `ModelLoader` struct is responsible for loading and managing models. It has the following methods:  
  
- `NewModelLoader(modelPath string)`: Creates a new `ModelLoader` instance with the given model path.  
- `SetWatchDog(wd *WatchDog)`: Sets the watch dog for the model loader.  
- `ExistsInModelPath(s string)`: Checks if a file exists in the model path.  
- `ListFilesInModelPath() ([]string, error)`: Lists all files in the model path, excluding known files to skip and directories.  
- `ListModels() []Model`: Returns a list of all loaded models.  
- `LoadModel(modelID, modelName string, loader func(string, string, string) (*Model, error)) (*Model, error)`: Loads a model with the given ID and name using the provided loader function.  
- `ShutdownModel(modelName string) error`: Shuts down a model with the given name.  
- `CheckIsLoaded(s string) *Model`: Checks if a model with the given ID is loaded and returns the model if it is.  
  
### Summary:  
  
The `model` package provides a `ModelLoader` struct that is responsible for loading and managing models. It has methods for creating a new instance, setting a watch dog, checking if a file exists in the model path, listing files in the model path, listing loaded models, loading a model, shutting down a model, and checking if a model is loaded.  
  
# pkg/model/loader_options.go  
## Package: model  
  
### Imports:  
- context  
- pb "github.com/mudler/LocalAI/pkg/grpc/proto"  
  
### External Data, Input Sources:  
- `externalBackends`: A map of string keys (backend names) to string values (backend URIs).  
  
### Options:  
- `backendString`: A string representing the backend to use.  
- `model`: A string representing the path to the model file.  
- `modelID`: A string representing the model ID.  
- `assetDir`: A string representing the directory containing assets.  
- `context`: A context.Context object.  
- `gRPCOptions`: A pointer to a pb.ModelOptions object.  
- `grpcAttempts`: An integer representing the number of attempts to make when connecting to a backend.  
- `grpcAttemptsDelay`: An integer representing the delay in seconds between attempts to connect to a backend.  
- `singleActiveBackend`: A boolean indicating whether to use only one active backend.  
- `parallelRequests`: A boolean indicating whether to make parallel requests to backends.  
  
### Functions:  
- `NewOptions(opts ...Option)`: Creates a new Options object with the given options.  
  
### Option Functions:  
- `EnableParallelRequests`: Sets the `parallelRequests` field to true.  
- `WithExternalBackend(name string, uri string)`: Adds an external backend to the `externalBackends` map.  
- `WithGRPCAttempts(attempts int)`: Sets the `grpcAttempts` field to the given value.  
- `WithGRPCAttemptsDelay(delay int)`: Sets the `grpcAttemptsDelay` field to the given value.  
- `WithBackendString(backend string)`: Sets the `backendString` field to the given value.  
- `WithModel(modelFile string)`: Sets the `model` field to the given value.  
- `WithLoadGRPCLoadModelOpts(opts *pb.ModelOptions)`: Sets the `gRPCOptions` field to the given value.  
- `WithAssetDir(assetDir string)`: Sets the `assetDir` field to the given value.  
- `WithContext(ctx context.Context)`: Sets the `context` field to the given value.  
- `WithSingleActiveBackend()`: Sets the `singleActiveBackend` field to true.  
- `WithModelID(id string)`: Sets the `modelID` field to the given value.  
  
# pkg/model/model.go  
## Package: model  
  
### Imports:  
  
- `sync`  
- `github.com/mudler/LocalAI/pkg/grpc`  
- `github.com/mudler/go-processmanager`  
  
### External Data, Input Sources:  
  
- `process.Process`: Represents a running process.  
- `WatchDog`: An object that monitors the health of the process.  
  
### Model:  
  
The `Model` struct represents a model that can be used for machine learning tasks. It has the following fields:  
  
- `ID`: A unique identifier for the model.  
- `address`: The address of the model's gRPC server.  
- `client`: A gRPC client that can be used to communicate with the model.  
- `process`: A `process.Process` object that represents the running process for the model.  
- `sync.Mutex`: A mutex to protect the shared state of the model.  
  
### NewModel:  
  
The `NewModel` function creates a new `Model` instance. It takes the model's ID, address, and a `process.Process` object as input and returns a new `Model` instance.  
  
### Process:  
  
The `Process` method returns the `process.Process` object associated with the model.  
  
### GRPC:  
  
The `GRPC` method returns a gRPC client that can be used to communicate with the model. It takes two arguments:  
  
- `parallel`: A boolean flag that indicates whether the gRPC client should be used in parallel.  
- `wd`: A `WatchDog` object that monitors the health of the process.  
  
If a client is already available, it will be returned. Otherwise, a new client will be created and returned. The new client will be created with the specified parallel flag and WatchDog object.  
  
# pkg/model/process.go  
## Package: model  
  
### Imports:  
  
```  
errors  
fmt  
os  
os/signal  
path/filepath  
strconv  
strings  
syscall  
  
github.com/hpcloud/tail  
github.com/mudler/go-processmanager  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
- GRPCProcessFilter: A function that takes a process ID and a process object as input and returns a boolean value. This function is used to filter which processes should be stopped.  
  
### Summary of Code:  
  
#### ModelLoader:  
  
The ModelLoader struct is responsible for managing and controlling the lifecycle of GRPC processes. It provides methods for starting, stopping, and retrieving information about these processes.  
  
#### deleteProcess:  
  
This method takes a process ID as input and removes the corresponding process from the ModelLoader's internal state. It first checks if the process exists and if it has a process object associated with it. If both conditions are true, it attempts to stop the process and removes it from the ModelLoader's internal state.  
  
#### StopGRPC:  
  
This method takes a GRPCProcessFilter function as input and iterates through the ModelLoader's internal state. For each process that matches the filter criteria, it calls the ShutdownModel method to stop the process.  
  
#### StopAllGRPC:  
  
This method calls the StopGRPC method with a default filter that includes all processes.  
  
#### GetGRPCPID:  
  
This method takes a process ID as input and returns the process ID of the corresponding GRPC process. It first checks if the process exists and if it has a process object associated with it. If both conditions are true, it returns the process ID as an integer.  
  
#### startProcess:  
  
This method takes the path to the GRPC process executable, a process ID, a server address, and an array of arguments as input. It first ensures that the process executable is executable and then creates a new process object using the provided information. It then starts the process and sets up monitoring for its standard output and standard error streams.  
  
#### GRPCProcessFilter:  
  
This is a function that takes a process ID and a process object as input and returns a boolean value. It is used to filter which processes should be stopped by the StopGRPC method.  
  
  
  
# pkg/model/template.go  
## Package: model  
  
### Imports:  
  
- fmt  
- github.com/mudler/LocalAI/pkg/functions  
- github.com/mudler/LocalAI/pkg/templates  
  
### External Data, Input Sources:  
  
- PromptTemplateData:  
    - SystemPrompt: string  
    - SuppressSystemPrompt: bool  
    - Input: string  
    - Instruction: string  
    - Functions: []functions.Function  
    - MessageIndex: int  
- ChatMessageTemplateData:  
    - SystemPrompt: string  
    - Role: string  
    - RoleName: string  
    - FunctionName: string  
    - Content: string  
    - MessageIndex: int  
    - Function: bool  
    - FunctionCall: interface{}  
    - LastMessage: bool  
- templates.TemplateType:  
    - ChatPromptTemplate  
    - ChatMessageTemplate  
    - CompletionPromptTemplate  
    - EditPromptTemplate  
    - FunctionsPromptTemplate  
  
### Code Summary:  
  
#### PromptTemplateData and ChatMessageTemplateData:  
  
These structs define the data that can be used in prompt templates. PromptTemplateData is used for general prompts, while ChatMessageTemplateData is specifically for chat messages. Both structs contain various fields that can be populated with data, such as system prompts, input, instructions, and function information.  
  
#### Template Types:  
  
The code defines five template types: ChatPromptTemplate, ChatMessageTemplate, CompletionPromptTemplate, EditPromptTemplate, and FunctionsPromptTemplate. These types are used to identify the type of prompt being evaluated.  
  
#### EvaluateTemplateForPrompt and EvaluateTemplateForChatMessage:  
  
These functions are used to evaluate prompt templates. EvaluateTemplateForPrompt takes a template type and name, as well as a PromptTemplateData struct, and returns the evaluated template string and any errors. EvaluateTemplateForChatMessage takes a template name and a ChatMessageTemplateData struct, and returns the evaluated template string and any errors.  
  
#### ModelLoader:  
  
The code also includes a ModelLoader struct, which is responsible for evaluating and returning prompt templates. It has methods for evaluating templates for both general prompts and chat messages.  
  
# pkg/model/watchdog.go  
## Package: model  
  
### Imports:  
  
```  
sync  
time  
process "github.com/mudler/go-processmanager"  
"github.com/rs/zerolog/log"  
```  
  
### External Data, Input Sources:  
  
- ProcessManager interface: This interface is used to interact with the process manager, which is responsible for managing the backend processes. The ProcessManager should implement the ShutdownModel method, which takes a model name as input and returns an error if the shutdown fails.  
  
### WatchDog:  
  
The WatchDog struct is responsible for tracking the state of all backend processes and ensuring that they don't stay busy for too long. It does this by monitoring the time each process has been busy and killing the process if it exceeds a certain threshold.  
  
- The WatchDog struct has several fields, including a map to store the last time each process was marked as busy, a map to store the last time each process was marked as idle, and a map to store the process objects for each address. It also has a channel to receive shutdown signals and a boolean flag to indicate whether the watchdog should check for busy or idle processes.  
  
- The NewWatchDog function creates a new WatchDog instance with the specified timeout values for busy and idle processes, as well as the ProcessManager to interact with.  
  
- The Shutdown method is used to stop the watchdog by sending a signal to the stop channel.  
  
- The AddAddressModelMap method is used to associate a model name with a given address.  
  
- The Add method is used to add a process object to the address map.  
  
- The Mark method is used to mark a process as busy by updating the timetable and deleting the entry from the idleTime map.  
  
- The UnMark method is used to mark a process as idle by updating the idleTime map and deleting the entry from the timetable.  
  
- The Run method starts the watchdog loop, which continuously checks for busy and idle processes based on the timeout values and the busyCheck and idleCheck flags.  
  
- The checkBusy method iterates through the timetable and kills any process that has been busy for longer than the timeout value.  
  
- The checkIdle method iterates through the idleTime map and kills any process that has been idle for longer than the idletimeout value.  
  
### Summary:  
  
The model package contains a WatchDog struct that monitors the state of backend processes and ensures that they don't stay busy or idle for too long. It does this by tracking the last time each process was marked as busy or idle and killing the process if it exceeds a certain threshold. The WatchDog struct has several methods for adding and removing processes, marking them as busy or idle, and shutting down the watchdog. The package also includes an interface for interacting with the process manager, which is responsible for managing the backend processes.  
  
