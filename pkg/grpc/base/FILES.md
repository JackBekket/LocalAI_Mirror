# pkg/grpc/base/base.go  
## Package: base  
  
### Imports:  
  
- fmt  
- os  
- github.com/mudler/LocalAI/pkg/grpc/proto  
- github.com/shirou/gopsutil/v3/process  
  
### External Data, Input Sources:  
  
- The package uses the gopsutil library to get process memory information.  
- It also uses the proto package to define the gRPC service interface.  
  
### Summary:  
  
The package `base` provides a base class for all backends to implement. It includes methods for locking, unlocking, checking if the backend is busy, loading the model, predicting, generating embeddings, generating images, transcribing audio, generating speech, generating sound, tokenizing strings, and getting the status of the backend.  
  
The `Base` struct has several methods that are meant to be implemented by the specific backend types. These methods include:  
  
- `Locking()`: Returns a boolean indicating whether the backend is locking.  
- `Lock()`: Locks the backend.  
- `Unlock()`: Unlocks the backend.  
- `Busy()`: Returns a boolean indicating whether the backend is busy.  
- `Load()`: Loads the model.  
- `Predict()`: Predicts using the loaded model.  
- `PredictStream()`: Predicts using the loaded model in a streaming fashion.  
- `Embeddings()`: Generates embeddings for the input text.  
- `GenerateImage()`: Generates an image based on the input.  
- `AudioTranscription()`: Transcribes audio input.  
- `TTS()`: Generates speech from text input.  
- `SoundGeneration()`: Generates sound based on the input.  
- `TokenizeString()`: Tokenizes a string.  
- `Status()`: Returns the status of the backend, including memory usage.  
- `StoresSet()`: Sets the stores.  
- `StoresGet()`: Gets the stores.  
- `StoresDelete()`: Deletes the stores.  
- `StoresFind()`: Finds the stores.  
  
The `memoryUsage()` function is used to capture the gopsutil info and enhance it with additional memory usage details.  
  
# pkg/grpc/base/singlethread.go  
## Package: base  
  
### Imports:  
  
- `sync`  
- `pb "github.com/mudler/LocalAI/pkg/grpc/proto"`  
  
### External Data, Input Sources:  
  
- `memoryUsage()` function (not provided in the code, but assumed to be available)  
  
### Summary:  
  
#### SingleThread struct:  
  
The `SingleThread` struct is a type of backend that does not support multiple requests. It ensures that only one request is being served at a time, which is useful for models that are not thread-safe and cannot run multiple requests concurrently.  
  
#### Locking mechanism:  
  
The `SingleThread` struct implements a locking mechanism using a `sync.Mutex` called `backendBusy`. The `Locking()` method returns `true` to indicate that the backend needs to lock resources. The `Lock()` and `Unlock()` methods are used to acquire and release the lock, respectively.  
  
#### Busy() method:  
  
The `Busy()` method attempts to acquire the lock using `TryLock()`. If successful, it releases the lock immediately and returns `true`, indicating that the backend is busy. Otherwise, it returns `false`.  
  
#### Status() method:  
  
The `Status()` method captures the current state of the backend and returns a `pb.StatusResponse` object. It calls the `memoryUsage()` function to get memory usage details and sets the `State` field in the response based on whether the backend is busy or not. The `Memory` field in the response contains the memory usage details.  
  
  
  
