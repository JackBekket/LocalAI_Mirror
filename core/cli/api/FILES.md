# core/cli/api/p2p.go  
## Package: cli_api  
  
### Imports:  
  
- context  
- fmt  
- net  
- os  
- strings  
- github.com/mudler/LocalAI/core/p2p  
- github.com/mudler/edgevpn/pkg/node  
- github.com/rs/zerolog/log  
  
### External Data, Input Sources:  
  
- address (string)  
- token (string)  
- networkID (string)  
- federated (bool)  
  
### Summary:  
  
#### StartP2PStack function:  
  
This function is responsible for starting the P2P stack based on the provided parameters. It first checks if the federated mode is enabled. If it is, it creates a federated node and exposes a service to the local instance running at the provided address. The function then starts the service discovery for the node.  
  
If the federated mode is not enabled, the function checks if a token is provided. If a token is provided, it creates a new node and starts the service discovery. The service discovery attaches a ServiceDiscoverer to the p2p node, which is responsible for discovering available nodes and setting the LLAMACPP_GRPC_SERVERS environment variable to the tunnel addresses of the available nodes.  
  
Finally, the function returns an error if any of the steps fail.  
  
  
  
