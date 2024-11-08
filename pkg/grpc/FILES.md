# pkg/grpc/backend.go  
## Package: grpc  
  
### Imports:  
  
- `context`  
- `pb "github.com/mudler/LocalAI/pkg/grpc/proto"`  
- `google.golang.org/grpc`  
  
### External Data, Input Sources:  
  
- `embeds`: A map that stores backend instances based on their address.  
  
### Code Summary:  
  
#### `Provide` function:  
  
- Takes an address and an LLM instance as input.  
- Stores the backend instance in the `embeds` map based on the provided address.  
  
#### `NewClient` function:  
  
- Takes an address, parallel flag, WatchDog instance, and enableWatchDog flag as input.  
- If a backend instance exists for the given address in the `embeds` map, it returns that instance.  
- Otherwise, it calls the `buildClient` function to create a new client instance.  
  
#### `buildClient` function:  
  
- Takes an address, parallel flag, WatchDog instance, and enableWatchDog flag as input.  
- If `enableWatchDog` is false, the WatchDog instance is set to nil.  
- Returns a new Client instance with the provided address, parallel flag, and WatchDog instance.  
  
#### `Backend` interface:  
  
- Defines methods for interacting with the backend, such as:  
    - `IsBusy()`: Checks if the backend is busy.  
    - `HealthCheck(ctx context.Context)`: Performs a health check on the backend.  
    - `Embeddings(ctx context.Context, in *pb.PredictOptions, opts ...grpc.CallOption)`: Generates embeddings for the given input.  
    - `Predict(ctx context.Context, in *pb.PredictOptions, opts ...grpc.CallOption)`: Performs prediction on the given input.  
    - `LoadModel(ctx context.Context, in *pb.ModelOptions, opts ...grpc.CallOption)`: Loads a new model.  
    - `PredictStream(ctx context.Context, in *pb.PredictOptions, f func(s []byte), opts ...grpc.CallOption)`: Performs prediction in a streaming manner.  
    - `GenerateImage(ctx context.Context, in *pb.GenerateImageRequest, opts ...grpc.CallOption)`: Generates an image.  
    - `TTS(ctx context.Context, in *pb.TTSRequest, opts ...grpc.CallOption)`: Performs text-to-speech conversion.  
    - `SoundGeneration(ctx context.Context, in *pb.SoundGenerationRequest, opts ...grpc.CallOption)`: Generates sound.  
    - `AudioTranscription(ctx context.Context, in *pb.TranscriptRequest, opts ...grpc.CallOption)`: Performs audio transcription.  
    - `TokenizeString(ctx context.Context, in *pb.PredictOptions, opts ...grpc.CallOption)`: Tokenizes a string.  
    - `Status(ctx context.Context)`: Gets the status of the backend.  
    - `StoresSet(ctx context.Context, in *pb.StoresSetOptions, opts ...grpc.CallOption)`: Sets data in the backend's storage.  
    - `StoresDelete(ctx context.Context, in *pb.StoresDeleteOptions, opts ...grpc.CallOption)`: Deletes data from the backend's storage.  
    - `StoresGet(ctx context.Context, in *pb.StoresGetOptions, opts ...grpc.CallOption)`: Retrieves data from the backend's storage.  
    - `StoresFind(ctx context.Context, in *pb.StoresFindOptions, opts ...grpc.CallOption)`: Searches for data in the backend's storage.  
    - `Rerank(ctx context.Context, in *pb.RerankRequest, opts ...grpc.CallOption)`: Reranks results.  
    - `GetTokenMetrics(ctx context.Context, in *pb.MetricsRequest, opts ...grpc.CallOption)`: Retrieves token metrics.  
  
  
  
# pkg/grpc/client.go  
  
  
# pkg/grpc/embed.go  
## Package: grpc  
  
### Imports:  
  
```  
"context"  
"github.com/mudler/LocalAI/pkg/grpc/proto"  
"google.golang.org/grpc"  
"google.golang.org/grpc/metadata"  
```  
  
### External Data, Input Sources:  
  
- `pb.Backend_PredictStreamServer`: Interface for handling prediction stream requests.  
- `pb.PredictOptions`: Structure containing input data for prediction requests.  
- `pb.EmbeddingResult`: Structure containing embedding results.  
- `pb.Reply`: Structure containing reply data for prediction requests.  
- `pb.ModelOptions`: Structure containing model loading options.  
- `pb.Result`: Structure containing results for various operations.  
- `pb.GenerateImageRequest`: Structure containing input data for image generation requests.  
- `pb.TTSRequest`: Structure containing input data for text-to-speech requests.  
- `pb.SoundGenerationRequest`: Structure containing input data for sound generation requests.  
- `pb.TranscriptRequest`: Structure containing input data for audio transcription requests.  
- `pb.TokenizationResponse`: Structure containing tokenization results.  
- `pb.StatusResponse`: Structure containing status information.  
- `pb.StoresSetOptions`: Structure containing options for setting stores.  
- `pb.StoresDeleteOptions`: Structure containing options for deleting stores.  
- `pb.StoresGetOptions`: Structure containing options for getting stores.  
- `pb.StoresFindOptions`: Structure containing options for finding stores.  
- `pb.RerankRequest`: Structure containing input data for reranking requests.  
- `pb.MetricsRequest`: Structure containing input data for getting token metrics.  
- `pb.MetricsResponse`: Structure containing token metrics.  
  
### Code Summary:  
  
1. **Backend Interface:** The code defines an interface `Backend` which is implemented by the `embedBackend` struct. This interface provides methods for various operations such as embedding, prediction, model loading, and more.  
  
2. **embedBackend Struct:** This struct implements the `Backend` interface and provides methods for handling various operations. It also has a field `s` which is a pointer to a `server` struct.  
  
3. **embedBackendServerStream Struct:** This struct is used for handling prediction stream requests. It has a field `ctx` for the context and a field `fn` which is a function that receives the reply data.  
  
4. **Methods for Various Operations:** The code provides methods for various operations such as embedding, prediction, model loading, image generation, text-to-speech, sound generation, audio transcription, tokenization, status, stores management, reranking, and getting token metrics. These methods are implemented in the `embedBackend` struct and use the `server` struct to perform the actual operations.  
  
5. **Prediction Stream Handling:** The `PredictStream` method in the `embedBackend` struct takes a `PredictOptions` struct and a function `f` as input. It creates an instance of the `embedBackendServerStream` struct and calls the `PredictStream` method on the `server` struct, passing the `PredictOptions` struct and the `embedBackendServerStream` instance as arguments.  
  
6. **Status and Stores Management:** The code also provides methods for checking the status of the backend and managing stores. These methods use the `server` struct to perform the actual operations.  
  
7. **Token Metrics Retrieval:** The `GetTokenMetrics` method in the `embedBackend` struct takes a `MetricsRequest` struct as input and returns a `MetricsResponse` struct containing the token metrics.  
  
8. **Error Handling:** The code returns errors for various operations, such as when a method fails to complete successfully or when an invalid input is provided.  
  
9. **Context Handling:** The code uses the `context` package to handle context information, such as cancellation and deadlines.  
  
10. **Metadata Handling:** The code uses the `google.golang.org/grpc/metadata` package to handle metadata, such as setting and retrieving headers for gRPC calls.  
  
  
  
# pkg/grpc/interface.go  
## Package: grpc  
  
### Imports:  
- github.com/mudler/LocalAI/pkg/grpc/proto  
  
### External Data, Input Sources:  
- pb.PredictOptions  
- pb.ModelOptions  
- pb.GenerateImageRequest  
- pb.TranscriptRequest  
- pb.TTSRequest  
- pb.SoundGenerationRequest  
- pb.PredictOptions  
- pb.TokenizationResponse  
- pb.StatusResponse  
- pb.StoresSetOptions  
- pb.StoresDeleteOptions  
- pb.StoresGetOptions  
- pb.StoresFindOptions  
  
### Code Summary:  
#### LLM Interface:  
The code defines an interface called LLM, which represents a large language model. This interface includes methods for various tasks such as predicting text, generating images, transcribing audio, generating sound, tokenizing strings, and managing stores.  
  
#### newReply Function:  
The newReply function takes a string as input and returns a new instance of the pb.Reply struct with the given message. This function is likely used for creating responses in the context of the LLM.  
  
  
  
# pkg/grpc/server.go  
## Package: grpc  
  
This package provides a gRPC server implementation for running LLM inference. It exposes various methods for interacting with the underlying LLM model, such as embedding, loading models, prediction, image generation, TTS, sound generation, audio transcription, and more.  
  
### Imports:  
  
- context  
- fmt  
- log  
- net  
- github.com/mudler/LocalAI/pkg/grpc/proto  
- google.golang.org/grpc  
  
### External Data, Input Sources:  
  
- The package relies on an external LLM model, which is passed as an argument to the `StartServer` and `RunServer` functions.  
  
### Code Summary:  
  
1. **Server Structure:**  
   - The `server` struct implements the `pb.UnimplementedBackendServer` interface and holds a reference to the LLM model.  
  
2. **Health Check:**  
   - The `Health` method returns a simple "OK" response, indicating that the server is up and running.  
  
3. **Embedding:**  
   - The `Embedding` method takes a `pb.PredictOptions` object as input and returns an `pb.EmbeddingResult` containing the embeddings generated by the LLM.  
  
4. **Model Loading:**  
   - The `LoadModel` method takes a `pb.ModelOptions` object as input and returns a `pb.Result` indicating whether the model was loaded successfully.  
  
5. **Prediction:**  
   - The `Predict` method takes a `pb.PredictOptions` object as input and returns a `pb.Reply` containing the prediction result.  
  
6. **Image Generation:**  
   - The `GenerateImage` method takes a `pb.GenerateImageRequest` object as input and returns a `pb.Result` indicating whether the image was generated successfully.  
  
7. **TTS and Sound Generation:**  
   - The `TTS` and `SoundGeneration` methods take `pb.TTSRequest` and `pb.SoundGenerationRequest` objects as input, respectively, and return a `pb.Result` indicating whether the audio was generated successfully.  
  
8. **Audio Transcription:**  
   - The `AudioTranscription` method takes a `pb.TranscriptRequest` object as input and returns a `pb.TranscriptResult` containing the transcribed text.  
  
9. **Prediction Stream:**  
   - The `PredictStream` method takes a `pb.PredictOptions` object and a `pb.Backend_PredictStreamServer` as input and returns an error if any.  
  
10. **Tokenization:**  
    - The `TokenizeString` method takes a `pb.PredictOptions` object as input and returns a `pb.TokenizationResponse` containing the tokenized string.  
  
11. **Status:**  
    - The `Status` method returns the current status of the LLM model.  
  
12. **Stores Management:**  
    - The `StoresSet`, `StoresDelete`, `StoresGet`, and `StoresFind` methods provide functionality for managing entries in a key-value store.  
  
13. **Server Start and Run Functions:**  
    - The `StartServer` and `RunServer` functions are responsible for starting and stopping the gRPC server, respectively. They take the address and the LLM model as input and return an error if any.  
  
  
  
