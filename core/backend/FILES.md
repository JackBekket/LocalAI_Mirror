# core/backend/embeddings.go  
## Package: backend  
  
### Imports:  
  
- fmt  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/pkg/grpc  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
- s (string)  
- tokens (slice of integers)  
- loader (model.ModelLoader)  
- backendConfig (config.BackendConfig)  
- appConfig (config.ApplicationConfig)  
  
### ModelEmbedding Function:  
  
The ModelEmbedding function takes a string `s`, a slice of integers `tokens`, a model loader `loader`, a backend configuration `backendConfig`, and an application configuration `appConfig` as input. It returns a function that can be called to get the embeddings for the given input and an error if any.  
  
1. It initializes an interface variable `inferenceModel` and an error variable `err`.  
  
2. It creates a slice of model options `opts` based on the backend configuration and application configuration.  
  
3. If the backend configuration is empty, it uses the greedy loader to load the model. Otherwise, it uses the backend loader with the specified backend string.  
  
4. If there is an error during model loading, it returns an error.  
  
5. It creates a function `fn` based on the type of the loaded model.  
  
6. If the loaded model is a gRPC backend, it creates a function that takes the application context, prediction options, and embeddings as input and returns the embeddings.  
  
7. If the loaded model is not a gRPC backend, it returns an error indicating that embeddings are not supported by the backend.  
  
8. The returned function calls the `fn` to get the embeddings and removes any trailing 0s from the result.  
  
9. Finally, it returns the function that can be called to get the embeddings and an error if any.  
  
# core/backend/image.go  
## Package: backend  
  
### Imports:  
  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/pkg/grpc/proto  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
- `height`, `width`, `mode`, `step`, `seed`: integer parameters for image generation.  
- `positive_prompt`, `negative_prompt`: strings for positive and negative prompts.  
- `src`, `dst`: strings for source and destination paths.  
- `loader`: pointer to a model loader.  
- `backendConfig`: config.BackendConfig object containing backend configuration.  
- `appConfig`: config.ApplicationConfig object containing application configuration.  
  
### ImageGeneration Function:  
  
The `ImageGeneration` function takes various parameters and returns a function that can be called to generate an image and an error if any.  
  
1. It initializes `opts` with ModelOptions based on the provided backend and application configurations.  
  
2. It calls the `BackendLoader` method of the model loader, passing the options and any additional model options. If an error occurs during this process, it returns an error.  
  
3. It defines a nested function `fn` that performs the actual image generation. This function takes the application context, a GenerateImageRequest proto message, and the model. It calls the `GenerateImage` method of the model, passing the request and the model. If an error occurs during this process, it returns the error.  
  
4. Finally, it returns the `fn` function and nil error.  
  
# core/backend/llm.go  
## Package: backend  
  
### Imports:  
  
```  
context  
encoding/json  
fmt  
os  
regexp  
strings  
sync  
unicode/utf8  
  
github.com/rs/zerolog/log  
  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/core/schema  
  
github.com/mudler/LocalAI/core/gallery  
github.com/mudler/LocalAI/pkg/grpc  
github.com/mudler/LocalAI/pkg/grpc/proto  
model "github.com/mudler/LocalAI/pkg/model"  
github.com/mudler/LocalAI/pkg/utils  
```  
  
### External Data, Input Sources:  
  
1. `config.BackendConfig`: Configuration for the backend, including model file, backend type, and other settings.  
2. `schema.Message`: A message containing the prompt and content for the model.  
3. `images, videos, audios`: Lists of image, video, and audio file paths.  
4. `loader`: A model loader that can load models from various sources.  
5. `o`: Application configuration.  
6. `tokenCallback`: A function that can be used to track token usage and other metrics.  
  
### ModelInference Function:  
  
This function takes a context, prompt string, messages, images, videos, audios, model loader, backend configuration, application configuration, and a token callback function as input. It returns a function that can be used to perform model inference and an error.  
  
1. It first checks if the model file exists. If not, it tries to download the model from the gallery.  
2. It then loads the model using the specified backend or greedy loader.  
3. If the tokenizer template is used, it converts the messages to proto messages.  
4. It then performs model inference using the loaded model and the provided input data.  
5. If a token callback function is provided, it tracks token usage and returns the response along with the token usage information.  
6. Otherwise, it returns the response without token usage information.  
  
### Finetune Function:  
  
This function takes a backend configuration, input, and prediction string as input. It performs various transformations on the prediction string based on the configuration, such as trimming, removing specific substrings, and extracting results from the response. Finally, it returns the transformed prediction string.  
  
### Summary:  
  
The backend package provides functions for model inference and fine-tuning. The ModelInference function loads a model, performs inference, and returns the response along with token usage information if a token callback function is provided. The Finetune function takes a prediction string and applies various transformations based on the configuration, such as trimming, removing substrings, and extracting results.  
  
# core/backend/options.go  
## Package: backend  
  
### Imports:  
  
```  
math/rand  
os  
path/filepath  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/pkg/grpc/proto  
github.com/mudler/LocalAI/pkg/model  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
1. Configuration data from `config.BackendConfig` and `config.ApplicationConfig` structs.  
2. Model options provided as a slice of `model.Option` structs.  
  
### Summary of Major Code Parts:  
  
#### ModelOptions function:  
  
This function takes a `config.BackendConfig` struct, an `config.ApplicationConfig` struct, and a slice of `model.Option` structs as input. It combines these inputs to create a new slice of `model.Option` structs that are used to configure the model.  
  
1. It first sets the default model options based on the provided configuration data.  
2. It then sets the number of threads to use for model loading based on the configuration data.  
3. It creates a `pb.ModelOptions` struct based on the configuration data and appends it to the default model options.  
4. It adds additional model options based on the application configuration data, such as whether to use a single active backend or enable parallel backend requests.  
5. It adds options for handling GRPC attempts, sleep time, and external GRPC backends based on the configuration data.  
6. Finally, it returns the combined slice of model options.  
  
#### getSeed function:  
  
This function takes a `config.BackendConfig` struct as input and returns an integer representing the random seed to use for model loading.  
  
1. It first sets the default seed value to `config.RAND_SEED`.  
2. It then checks if a seed value is provided in the configuration data and updates the seed value accordingly.  
3. If no seed value is provided, it generates a random seed value using `rand.Int31()`.  
4. Finally, it returns the seed value.  
  
#### grpcModelOpts function:  
  
This function takes a `config.BackendConfig` struct as input and returns a `pb.ModelOptions` struct that is used to configure the model.  
  
1. It sets various model options based on the configuration data, such as CUDA, scheduler type, pipeline type, CFG scale, Lora adapter, Lora scale, Lora adapters, Lora scales, F16 memory, Lora base, IMG2IMG, CLIP model, CLIP subfolder, CLIP skip, ControlNet, context size, seed, batch size, NoMulMatQ, draft model, audio path, quantization, load format, GPU memory utilization, trust remote code, enforce eager, swap space, max model length, tensor parallel size, MMProj, flash attention, no KV offload, yarn ext factor, yarn attn factor, yarn beta fast, yarn beta slow, NGQA, RMSNormEps, MLock, rope freq base, rope scaling, type, rope freq scale, NUMA, embeddings, low VRAM, NGPU layers, MMap, main GPU, threads, and tensor split.  
2. It also sets options for AutoGPTQ and RWKV based on the configuration data.  
3. Finally, it returns the `pb.ModelOptions` struct.  
  
#### gRPCPredictOpts function:  
  
This function takes a `config.BackendConfig` struct and a model path as input and returns a `pb.PredictOptions` struct that is used to configure the model for prediction.  
  
1. It first checks if a prompt cache path is provided in the configuration data and creates the prompt cache folder if necessary.  
2. It then sets various prediction options based on the configuration data, such as temperature, topP, NDraft, topK, tokens, threads, prompt cache all, prompt cache RO, prompt cache path, F16KV, debug mode, grammar, negative prompt scale, rope freq base, rope freq scale, negative prompt, Mirostat, Mirostat ETA, Mirostat TAU, debug, stop prompts, repeat, frequency penalty, presence penalty, penalty, NKeep, batch, ignore EOS, seed, MLock, MMap, main GPU, tensor split, tail free sampling Z, and typical P.  
3. Finally, it returns the `pb.PredictOptions` struct.  
  
  
  
# core/backend/rerank.go  
## Package: backend  
  
### Imports:  
- context  
- fmt  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/pkg/grpc/proto  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
- modelFile (string)  
- request (proto.RerankRequest)  
- loader (model.ModelLoader)  
- appConfig (config.ApplicationConfig)  
- backendConfig (config.BackendConfig)  
  
### Rerank Function:  
The `Rerank` function takes a model file, a rerank request, a model loader, an application configuration, and a backend configuration as input. It first creates a set of model options based on the backend and application configurations, as well as any additional options provided. Then, it uses the model loader to load the rerank model based on the provided options. If the model is successfully loaded, it calls the `Rerank` method of the loaded model with the provided request and returns the result along with any errors encountered. If the model is not loaded successfully, it returns an error indicating that the rerank model could not be loaded.  
  
# core/backend/soundgeneration.go  
## Package: backend  
  
### Imports:  
  
```  
context  
fmt  
os  
path/filepath  
  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/pkg/grpc/proto  
github.com/mudler/LocalAI/pkg/model  
github.com/mudler/LocalAI/pkg/utils  
```  
  
### External Data, Input Sources:  
  
1. `modelFile`: Path to the model file.  
2. `text`: Text to be converted into sound.  
3. `duration`: Desired duration of the sound in seconds.  
4. `temperature`: Temperature parameter for text-to-speech generation.  
5. `doSample`: Boolean flag to indicate whether to sample the sound.  
6. `sourceFile`: Path to a source audio file (optional).  
7. `sourceDivisor`: Divisor for the source audio file (optional).  
8. `loader`: Model loader instance.  
9. `appConfig`: Application configuration.  
10. `backendConfig`: Backend configuration.  
  
### Sound Generation Function:  
  
The `SoundGeneration` function takes the above inputs and performs the following steps:  
  
1. Creates a list of model options based on the backend and application configurations.  
2. Loads the sound generation model using the provided model loader and options.  
3. Creates a directory for storing audio files if it doesn't exist.  
4. Generates a unique filename for the output audio file.  
5. Calls the sound generation method of the loaded model, passing the text, model file, output file path, sampling flag, duration, temperature, source file, and source divisor.  
6. Checks if the RPC call was successful and returns an error if not.  
7. Returns the path to the generated audio file, the RPC result, and any errors encountered.  
  
# core/backend/stores.go  
## Package: backend  
  
### Imports:  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/pkg/grpc  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
- model.ModelLoader  
- config.ApplicationConfig  
- storeName (string)  
  
### StoreBackend Function:  
This function takes a model loader, application configuration, and store name as input. It initializes a slice of model options, including the backend type, asset directory, and model name. It then calls the BackendLoader method of the model loader, passing the options, and returns the backend and any potential error. If the store name is empty, it defaults to "default".  
  
# core/backend/token_metrics.go  
## Package: backend  
  
### Imports:  
- context  
- fmt  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/pkg/grpc/proto  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
- modelFile (string)  
- loader (model.ModelLoader)  
- appConfig (config.ApplicationConfig)  
- backendConfig (config.BackendConfig)  
  
### TokenMetrics Function:  
The TokenMetrics function takes five arguments: modelFile, loader, appConfig, and backendConfig. It first creates a list of options for the model using the ModelOptions function, which takes the backendConfig, appConfig, and a list of model options. The model is then loaded using the loader's BackendLoader function with the provided options. If the model is not loaded successfully, an error is returned.  
  
If the model is loaded successfully, the GetTokenMetrics method of the model is called with a context and a MetricsRequest object. The result of this call is returned as a MetricsResponse object along with any errors that occurred during the process.  
  
# core/backend/tokenize.go  
## Package: backend  
  
### Imports:  
  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/schema  
- github.com/mudler/LocalAI/pkg/grpc  
- model "github.com/mudler/LocalAI/pkg/model"  
  
### External Data, Input Sources:  
  
- backendConfig: config.BackendConfig  
- appConfig: config.ApplicationConfig  
- loader: model.ModelLoader  
- s: string  
  
### ModelTokenize Function:  
  
The ModelTokenize function takes a string input (s), a model loader (loader), backend configuration (backendConfig), and application configuration (appConfig) as input. It first determines the model file path from the backend configuration. Then, it initializes a backend model (inferenceModel) based on the backend configuration and application configuration. The function uses the GreedyLoader or BackendLoader methods of the model loader to load the model based on the backend configuration.  
  
Once the model is loaded, the function creates a predictOptions object with the backend configuration and model path. The prompt is set to the input string (s). The function then calls the TokenizeString method of the loaded model to tokenize the input string. The result is returned as a schema.TokenizeResponse object containing the tokens.  
  
### Summary:  
  
The backend package provides a ModelTokenize function that takes a string input and tokenizes it using a loaded model. The function uses a model loader to load the model based on the backend configuration and application configuration. The loaded model is then used to tokenize the input string, and the result is returned as a schema.TokenizeResponse object.  
  
# core/backend/transcript.go  
## Package: backend  
  
### Imports:  
  
* context  
* fmt  
* time  
* github.com/mudler/LocalAI/core/config  
* github.com/mudler/LocalAI/core/schema  
* github.com/mudler/LocalAI/pkg/grpc/proto  
* github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
* audio (string)  
* language (string)  
* translate (bool)  
* ml (model.ModelLoader)  
* backendConfig (config.BackendConfig)  
* appConfig (config.ApplicationConfig)  
  
### ModelTranscription Function:  
  
The ModelTranscription function takes audio, language, translate, ml, backendConfig, and appConfig as input. It first checks if the backend is set in the backendConfig. If not, it defaults to model.WhisperBackend. Then, it creates ModelOptions using the backendConfig, appConfig, and an empty slice of model.Option.  
  
Next, it loads the transcription model using the ml.BackendLoader function with the created ModelOptions. If the transcription model is nil, it returns an error. Otherwise, it calls the AudioTranscription method of the transcription model with the provided audio, language, translate, and threads from the backendConfig.  
  
If the AudioTranscription method returns an error, it returns nil and the error. Otherwise, it creates a schema.TranscriptionResult object with the text from the response and iterates through the segments in the response. For each segment, it creates a schema.Segment object with the text, id, start, end, and tokens from the response. Finally, it returns the schema.TranscriptionResult object and any error.  
  
# core/backend/tts.go  
## Package: backend  
  
### Imports:  
  
```  
context  
fmt  
os  
path/filepath  
  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/pkg/grpc/proto  
github.com/mudler/LocalAI/pkg/model  
github.com/mudler/LocalAI/pkg/utils  
```  
  
### External Data, Input Sources:  
  
1. `backend`: String representing the backend to use for TTS.  
2. `text`: String containing the text to be converted to speech.  
3. `modelFile`: String representing the path to the model file.  
4. `voice`: String representing the voice to use for TTS.  
5. `language`: String representing the language to use for TTS.  
6. `loader`: A pointer to a model.ModelLoader instance.  
7. `appConfig`: A pointer to a config.ApplicationConfig instance.  
8. `backendConfig`: A config.BackendConfig instance.  
  
### ModelTTS Function:  
  
The ModelTTS function takes the above inputs and performs the following steps:  
  
1. Sets the backend to use for TTS. If the `backend` input is empty, it defaults to `model.PiperBackend`.  
2. Creates a list of options for the TTS model, including the backend and model file.  
3. Loads the TTS model using the provided options and the `loader` instance.  
4. Creates a directory for storing the generated audio files if it doesn't exist.  
5. Generates a unique filename for the output audio file.  
6. Constructs the full path to the output audio file.  
7. If the `modelFile` input is not empty, it checks if the model file exists and is within the allowed path. If so, it uses the full path to the model file; otherwise, it uses the `modelFile` input directly.  
8. Calls the TTS method of the loaded model, passing the text, model path, voice, output file path, and language as arguments.  
9. Checks if the RPC call was successful and returns an error if not.  
10. Returns the path to the generated audio file, the result of the TTS call, and any errors encountered.  
  
  
  
