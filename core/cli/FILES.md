# core/cli/cli.go  
## Package: cli  
  
### Imports:  
- cliContext "github.com/mudler/LocalAI/core/cli/context"  
- "github.com/mudler/LocalAI/core/cli/worker"  
  
### External Data and Input Sources:  
- None  
  
### CLI Structure:  
The code defines a struct named `CLI` which contains various command-line interface (CLI) components. These components are grouped under different subcommands, each with its own help text and default behavior.  
  
#### Run:  
- Command: `local-ai run`  
- Description: Runs LocalAI in the default mode.  
- Help: `Run 'local-ai run --help' for more information`  
- Default: `withargs`  
  
#### Federated:  
- Command: `local-ai federated`  
- Description: Runs LocalAI in federated mode.  
  
#### Models:  
- Command: `local-ai models`  
- Description: Manages LocalAI models and definitions.  
  
#### TTS:  
- Command: `local-ai tts`  
- Description: Converts text to speech.  
  
#### Sound Generation:  
- Command: `local-ai sound-generation`  
- Description: Generates audio files from text or audio.  
  
#### Transcript:  
- Command: `local-ai transcript`  
- Description: Converts audio to text.  
  
#### Worker:  
- Command: `local-ai worker`  
- Description: Runs workers to distribute workload (llama.cpp-only).  
  
#### Util:  
- Command: `local-ai util`  
- Description: Utility commands.  
  
#### Explorer:  
- Command: `local-ai explorer`  
- Description: Runs p2p explorer.  
  
  
  
# core/cli/explorer.go  
## Package: cli  
  
### Imports:  
  
- context  
- time  
- cliContext "github.com/mudler/LocalAI/core/cli/context"  
- explorer "github.com/mudler/LocalAI/core/explorer"  
- http "github.com/mudler/LocalAI/core/http"  
  
### External Data, Input Sources:  
  
- Environment variables:  
    - LOCALAI_ADDRESS: Bind address for the API server  
    - LOCALAI_POOL_DATABASE: Path to the pool database  
    - LOCALAI_CONNECTION_TIMEOUT: Connection timeout for the explorer  
    - LOCALAI_CONNECTION_ERROR_THRESHOLD: Connection failure threshold for the explorer  
    - LOCALAI_WITH_SYNC: Enable sync with the network  
    - LOCALAI_ONLY_SYNC: Only sync with the network  
  
### Code Summary:  
  
#### ExplorerCMD struct:  
  
This struct defines the command-line arguments for the explorer component. It includes fields for the bind address, pool database path, connection timeout, connection error threshold, and flags for enabling or only syncing with the network.  
  
#### Run method:  
  
This method is responsible for executing the explorer component. It first initializes a database using the provided pool database path. Then, it parses the connection timeout from the environment variable and sets up a discovery server if the `WithSync` flag is set. If the `OnlySync` flag is set, it starts the discovery server and returns the result of its execution. Otherwise, it creates an HTTP server using the explorer and starts listening on the specified address.  
  
# core/cli/federated.go  
## Package: cli  
  
### Imports:  
- context  
- cliContext "github.com/mudler/LocalAI/core/cli/context"  
- p2p "github.com/mudler/LocalAI/core/p2p"  
  
### External Data, Input Sources:  
- Environment variables:  
    - LOCALAI_ADDRESS: Bind address for the API server  
    - LOCALAI_P2P_TOKEN: Token for P2P mode  
    - LOCALAI_RANDOM_WORKER: Select a random worker from the pool  
    - LOCALAI_P2P_NETWORK_ID: Network ID for P2P mode  
    - LOCALAI_TARGET_WORKER: Target worker to run the federated server on  
  
### FederatedCLI struct:  
- Address: Bind address for the API server  
- Peer2PeerToken: Token for P2P mode  
- RandomWorker: Select a random worker from the pool  
- Peer2PeerNetworkID: Network ID for P2P mode  
- TargetWorker: Target worker to run the federated server on  
  
### Run method:  
- Takes a context from the cliContext package  
- Creates a new FederatedServer instance using the provided parameters  
- Starts the FederatedServer and returns any error encountered  
  
### Summary:  
The code defines a FederatedCLI struct and a Run method that creates and starts a federated server. The FederatedCLI struct takes several parameters from environment variables, including the bind address, P2P token, random worker selection, network ID, and target worker. The Run method creates a new FederatedServer instance using these parameters and starts the server, returning any errors encountered during the process.  
  
# core/cli/models.go  
## Package: cli  
  
### Imports:  
  
```  
encoding/json  
errors  
fmt  
github.com/mudler/LocalAI/core/cli/context  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/core/gallery  
github.com/mudler/LocalAI/pkg/downloader  
github.com/mudler/LocalAI/pkg/startup  
github.com/rs/zerolog/log  
github.com/schollz/progressbar/v3  
```  
  
### External Data, Input Sources:  
  
- JSON list of galleries (LOCALAI_GALLERIES or GALLERIES environment variable)  
- Path containing models used for inferencing (LOCALAI_MODELS_PATH or MODELS_PATH environment variable)  
- Model configuration URLs to load (arg: models)  
- DisablePredownloadScan flag (LOCALAI_DISABLE_PREDOWNLOAD_SCAN environment variable)  
  
### ModelsList:  
  
- This struct contains the ModelsCMDFlags and is used for listing available models.  
- The Run method of this struct takes a cliContext.Context as input and returns an error if any.  
- It unmarshals the JSON string of galleries from the Galleries field, then calls the AvailableGalleryModels function to get a list of available models.  
- It iterates through the models and prints the name and gallery of each model, indicating whether it's installed or not.  
  
### ModelsInstall:  
  
- This struct contains the ModelsCMDFlags and additional fields for installing models.  
- The Run method of this struct takes a cliContext.Context as input and returns an error if any.  
- It unmarshals the JSON string of galleries from the Galleries field, then iterates through the provided model names.  
- For each model, it creates a progress bar, finds the model in the available models, and performs a safety scan if necessary.  
- It then calls the InstallModels function to install the model, passing the galleries, model path, disablePredownloadScan flag, progress callback, and model name as arguments.  
  
### Summary:  
  
The provided code defines two structs, ModelsList and ModelsInstall, which are used for listing and installing models, respectively. Both structs take a cliContext.Context as input and return an error if any. The ModelsList struct iterates through the available models and prints their names and galleries, while the ModelsInstall struct iterates through the provided model names, installs each model, and handles progress updates. The code also includes functions for finding models, performing safety scans, and installing models.  
  
# core/cli/run.go  
  
  
# core/cli/soundgeneration.go  
## Package: cli  
  
### Imports:  
  
```  
"context"  
"fmt"  
"os"  
"path/filepath"  
"strconv"  
"strings"  
  
"github.com/mudler/LocalAI/core/backend"  
"github.com/mudler/LocalAI/core/cli/context"  
"github.com/mudler/LocalAI/core/config"  
"github.com/mudler/LocalAI/pkg/model"  
"github.com/rs/zerolog/log"  
```  
  
### External Data, Input Sources:  
  
- `Text`: A slice of strings representing the text to be used as input for the SoundGeneration model.  
- `Backend`: A string representing the backend to run the SoundGeneration model.  
- `Model`: A string representing the model name to run the SoundGeneration model.  
- `Duration`: A string representing the length of audio to generate in seconds.  
- `Temperature`: A string representing the temperature of the generation.  
- `InputFile`: A string representing the path to an input file to condition generation upon.  
- `InputFileSampleDivisor`: A string representing the divisor to use when sampling from the input file.  
- `DoSample`: A boolean flag indicating whether to enable sampling from the model.  
- `OutputFile`: A string representing the path to write the output wav file.  
- `ModelsPath`: A string representing the path containing models used for inferencing.  
- `BackendAssetsPath`: A string representing the path used to extract libraries that are required by some of the backends in runtime.  
- `ExternalGRPCBackends`: A list of strings representing external grpc backends.  
  
### SoundGenerationCMD:  
  
- This struct defines the command-line arguments for the SoundGeneration command.  
- It includes fields for the text input, backend, model, duration, temperature, input file, input file sample divisor, do sample flag, output file, models path, backend assets path, and external grpc backends.  
  
### Run Method:  
  
- This method is responsible for executing the SoundGeneration command.  
- It first parses the command-line arguments and sets up the necessary configuration options.  
- It then creates a model loader and initializes the backend configuration.  
- The SoundGeneration function is called with the specified parameters, and the resulting file path is returned.  
- If an output file is specified, the generated file is renamed to the output file path.  
- Finally, the method returns an error if any issues occur during the process.  
  
### Summary:  
  
The provided code defines a command-line tool for generating sound from text using a specified model and backend. It takes various command-line arguments to configure the generation process, including the text input, backend, model, duration, temperature, input file, and output file. The code also handles external grpc backends and provides options for sampling from the model and specifying the path to the models and backend assets. The SoundGeneration function is responsible for executing the actual sound generation process, and the resulting file path is returned.  
  
# core/cli/transcript.go  
## Package: cli  
  
### Imports:  
  
```  
context  
errors  
fmt  
github.com/mudler/LocalAI/core/backend  
github.com/mudler/LocalAI/core/cli/context  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/pkg/model  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
- `Filename`: String argument representing the path to the audio file.  
- `Backend`: String argument specifying the backend to use for transcription (default: "whisper").  
- `Model`: String argument specifying the model name to use for transcription (required).  
- `Language`: String argument specifying the language of the audio file.  
- `Translate`: Boolean argument indicating whether to translate the transcription to English.  
- `Threads`: Integer argument specifying the number of threads to use for parallel computation (default: 1).  
- `ModelsPath`: String argument specifying the path to the directory containing the models (default: "${basepath}/models").  
- `BackendAssetsPath`: String argument specifying the path to the directory where backend assets will be extracted (default: "/tmp/localai/backend_data").  
  
### Summary:  
  
#### TranscriptCMD:  
  
The `TranscriptCMD` struct defines the command-line arguments for the transcript command. It includes fields for the filename, backend, model, language, translation, threads, models path, and backend assets path.  
  
#### Run:  
  
The `Run` method of the `TranscriptCMD` struct is responsible for executing the transcription process. It first initializes an `ApplicationConfig` object with the provided arguments and loads the backend configurations from the specified models path. It then retrieves the backend configuration for the specified model and sets the number of threads to use.  
  
The method then starts the transcription process using the `ModelTranscription` function from the `backend` package. This function takes the filename, language, translation flag, model loader, backend configuration, and application configuration as input. It returns a `TranscriptionResult` object containing the transcribed segments.  
  
Finally, the method iterates through the transcribed segments and prints the start time and text for each segment. It also ensures that all GRPC processes are stopped when the method exits.  
  
# core/cli/tts.go  
## Package: cli  
  
### Imports:  
  
```  
context  
fmt  
os  
path/filepath  
strings  
  
github.com/mudler/LocalAI/core/backend  
github.com/mudler/LocalAI/core/cli/context  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/pkg/model  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
- `LOCALAI_MODELS_PATH` or `MODELS_PATH` environment variable: Path containing models used for inferencing.  
- `LOCALAI_BACKEND_ASSETS_PATH` or `BACKEND_ASSETS_PATH` environment variable: Path used to extract libraries that are required by some of the backends in runtime.  
  
### Code Summary:  
  
#### TTSCMD struct:  
  
This struct defines the command-line arguments for the text-to-speech (TTS) functionality. It includes fields for the text to be converted, the backend to use, the model to run, the voice to use, the language to use, the output file path, and paths for models and backend assets.  
  
#### Run method:  
  
This method is responsible for executing the TTS process. It first determines the output directory based on the provided output file path or the backend assets path. Then, it joins the text input into a single string.  
  
Next, it creates an `ApplicationConfig` object with the specified model path, context, audio directory, and assets destination. It also initializes a model loader using the provided model path.  
  
The method then calls the `ModelTTS` function from the `backend` package to perform the TTS conversion. This function takes the backend, text, model, voice, language, model loader, application configuration, and backend configuration as input. It returns the file path of the generated audio file, the model name, and any error encountered.  
  
Finally, the method renames the generated audio file to the specified output file path if provided, or prints the file path of the generated audio file otherwise. It also ensures that all GRPC processes are stopped when the method exits.  
  
# core/cli/util.go  
## Package: cli  
  
### Imports:  
  
- encoding/json  
- errors  
- fmt  
- github.com/rs/zerolog/log  
- github.com/mudler/LocalAI/core/cli/context  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/gallery  
- github.com/mudler/LocalAI/pkg/downloader  
- github.com/thxcode/gguf-parser-go  
  
### External Data, Input Sources:  
  
- `LOCALAI_MODELS_PATH` environment variable: Path containing models used for inferencing.  
- `LOCALAI_GALLERIES` environment variable: JSON list of galleries.  
- `galleries` variable: JSON list of galleries.  
- `ModelsPath` argument: Path containing models used for inferencing.  
- `ConfigName` argument: The config file to check.  
- `ToScan` argument: List of URIs to scan for security issues.  
  
### Code Summary:  
  
#### UtilCMD:  
  
- This struct defines three subcommands: `gguf-info`, `hf-scan`, and `usecase-heuristic`.  
  
#### GGUFInfoCMD:  
  
- This subcommand takes a GGUF file as input and provides information about its tokenizer, architecture, and header metadata.  
- It parses the GGUF file using the `gguf` package and prints the relevant information.  
  
#### HFScanCMD:  
  
- This subcommand checks installed models for known security issues.  
- It can either scan all installed models or a specific list of URIs.  
- It uses the `downloader` package to scan the models and check for vulnerabilities.  
  
#### UsecaseHeuristicCMD:  
  
- This subcommand takes a config file as input and prints the usecases that LocalAI will offer for the specified model.  
- It loads the backend config from the specified path and checks if the model has any usecases defined.  
  
#### Run methods:  
  
- Each subcommand has a `Run` method that handles the actual execution of the command.  
- The `Run` methods for each subcommand are responsible for parsing the input, performing the necessary operations, and printing the results.  
  
