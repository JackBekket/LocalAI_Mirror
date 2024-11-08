# core/cli/worker/worker.go  
## Package: worker  
  
### Imports:  
  
- P2P  
- LLamaCPP  
  
### External Data, Input Sources:  
  
- Environment variables:  
    - LOCALAI_BACKEND_ASSETS_PATH  
    - BACKEND_ASSETS_PATH  
    - LOCALAI_EXTRA_LLAMA_CPP_ARGS  
    - EXTRA_LLAMA_CPP_ARGS  
  
### Code Summary:  
  
#### WorkerFlags:  
  
This struct defines the flags for the worker component. It includes two fields:  
  
- BackendAssetsPath: This field specifies the path to the directory where libraries required by some backends will be extracted. It can be set using the environment variables LOCALAI_BACKEND_ASSETS_PATH or BACKEND_ASSETS_PATH, or by default, it will be set to "/tmp/localai/backend_data".  
  
- ExtraLLamaCPPArgs: This field allows you to pass extra arguments to the llama-cpp-rpc-server. It can be set using the environment variables LOCALAI_EXTRA_LLAMA_CPP_ARGS or EXTRA_LLAMA_CPP_ARGS.  
  
#### Worker:  
  
This struct defines the worker component and includes two fields:  
  
- P2P: This field represents a P2P llama.cpp worker that requires a token. It can be started using the command "p2p-llama-cpp-rpc".  
  
- LLamaCPP: This field represents a standalone llama.cpp worker. It can be started using the command "llama-cpp-rpc".  
  
  
  
# core/cli/worker/worker_llamacpp.go  
## Package: worker  
  
### Imports:  
  
- fmt  
- os  
- strings  
- syscall  
- cliContext "github.com/mudler/LocalAI/core/cli/context"  
- assets "github.com/mudler/LocalAI/pkg/assets"  
- library "github.com/mudler/LocalAI/pkg/library"  
- log "github.com/rs/zerolog/log"  
  
### External Data, Input Sources:  
  
- BackendAssetsPath: Path to the backend assets directory.  
- ExtraLLamaCPPArgs: Additional arguments for the Llama-CPP RPC server.  
  
### Code Summary:  
  
#### LLamaCPP struct:  
  
- The LLamaCPP struct embeds the WorkerFlags struct, which likely contains configuration options for the worker.  
  
#### Run method:  
  
1. Extracts files from the embedded FS to the specified BackendAssetsPath.  
2. Checks if the number of command-line arguments is at least 4. If not, it returns an error with usage instructions.  
3. Resolves the path to the Llama-CPP RPC server executable within the BackendAssetsPath.  
4. Splits the ExtraLLamaCPPArgs string into individual arguments.  
5. Loads any necessary shared libraries (LDSO) using the library package, updating the arguments and the executable path.  
6. Appends the resolved executable path to the arguments.  
7. Executes the Llama-CPP RPC server using syscall.Exec, passing the executable path and arguments, along with the environment variables.  
  
  
  
# core/cli/worker/worker_nop2p.go  
Package: worker  
  
Imports:  
- fmt  
- cliContext "github.com/mudler/LocalAI/core/cli/context"  
  
External data, input sources:  
- cliContext.Context  
  
Summary:  
### P2P struct and Run method  
  
The code defines a struct named P2P and a method called Run. The Run method takes a cliContext.Context as input and returns an error. The method returns an error indicating that p2p mode is not enabled in this build.  
  
# core/cli/worker/worker_p2p.go  
## Package: worker  
  
### Imports:  
  
```  
context  
fmt  
os  
os/exec  
strings  
time  
  
cliContext "github.com/mudler/LocalAI/core/cli/context"  
p2p "github.com/mudler/LocalAI/core/p2p"  
assets "github.com/mudler/LocalAI/pkg/assets"  
library "github.com/mudler/LocalAI/pkg/library"  
freeport "github.com/phayes/freeport"  
zerolog/log "github.com/rs/zerolog/log"  
```  
  
### External Data, Input Sources:  
  
- Backend assets path: `r.BackendAssetsPath`  
- Token: `r.Token`  
- No runner flag: `r.NoRunner`  
- Runner address: `r.RunnerAddress`  
- Runner port: `r.RunnerPort`  
- Peer2PeerNetworkID: `r.Peer2PeerNetworkID`  
- ExtraLLamaCPPArgs: `r.ExtraLLamaCPPArgs`  
  
### Summary of Code:  
  
The `worker` package contains a `P2P` struct that manages the peer-to-peer functionality of the LocalAI application. The `Run` method of the `P2P` struct is responsible for starting the peer-to-peer network and exposing the worker service.  
  
1. The `Run` method first extracts the backend assets from the embedded file system and stores them in the specified path.  
  
2. It then checks if the token is set and returns an error if it's not.  
  
3. The method then finds a free port and sets the address and port for the worker service.  
  
4. If the `NoRunner` flag is set, the method exposes the worker service using the provided address and port, and the user is instructed to start the llama-cpp-rpc-server separately.  
  
5. If the `NoRunner` flag is not set, the method starts the llama-cpp-rpc-server directly from the pre-packaged version and exposes the worker service using the address and port of the running server.  
  
6. Finally, the method enters an infinite loop and waits for any errors or events.  
  
The `worker` package provides a way to run the LocalAI application in a peer-to-peer mode, allowing multiple instances to communicate with each other and share resources.  
  
