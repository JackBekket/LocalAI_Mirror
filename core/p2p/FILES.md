# core/p2p/federated.go  
## Package: p2p  
  
### Imports:  
  
- fmt  
- math/rand/v2  
- sync  
- github.com/rs/zerolog/log  
  
### External Data, Input Sources:  
  
- FederatedID: "federated"  
- GetAvailableNodes(service string): Returns a list of available nodes for the given service.  
  
### Summary:  
  
#### FederatedServer:  
  
The FederatedServer struct represents a server that manages a group of nodes and distributes requests among them. It has the following fields:  
  
- listenAddr: The address to listen on for incoming connections.  
- service: The service name for the server.  
- p2ptoken: The p2p token for the server.  
- requestTable: A map that stores the number of requests served by each node.  
- loadBalanced: A boolean flag indicating whether the server should load balance requests among nodes.  
- workerTarget: The target worker for the server.  
  
#### NetworkID:  
  
The NetworkID function takes two strings, networkID and serviceID, and returns a string that combines them. If networkID is not empty, it returns a string in the format "networkID_serviceID". Otherwise, it returns the serviceID.  
  
#### NewFederatedServer:  
  
The NewFederatedServer function creates a new FederatedServer instance with the given parameters.  
  
#### RandomServer:  
  
The RandomServer method returns a random online node from the available nodes for the given service. It iterates through the available nodes, checks if they are online, and appends their IDs to a list. If there are no online nodes, it returns an empty string.  
  
#### syncTableStatus:  
  
The syncTableStatus method synchronizes the requestTable with the available nodes for the given service. It iterates through the available nodes, checks if they are online, and updates the requestTable accordingly.  
  
#### SelectLeastUsedServer:  
  
The SelectLeastUsedServer method selects the node with the lowest request count from the requestTable. If there are multiple nodes with the same request count, it selects one randomly. If there are no entries in the requestTable, it returns an empty string.  
  
#### RecordRequest:  
  
The RecordRequest method increments the request count for the given nodeID in the requestTable.  
  
#### ensureRecordExist:  
  
The ensureRecordExist method adds a new entry to the requestTable for the given nodeID with a counter of 0 if it doesn't already exist.  
  
  
  
# core/p2p/federated_server.go  
## Package: p2p  
  
### Imports:  
  
```  
context  
errors  
fmt  
io  
net  
  
github.com/mudler/edgevpn/pkg/node  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
- `p2ptoken`: A token used for creating a new node.  
- `service`: A string representing the service being allocated.  
- `listenAddr`: A string representing the local port for listening.  
- `workerTarget`: A string representing the target worker node.  
- `loadBalanced`: A boolean indicating whether load balancing should be used.  
  
### Summary:  
  
#### FederatedServer.Start:  
  
This function initializes a new node using the provided `p2ptoken` and starts it. It then discovers other nodes using the `ServiceDiscoverer` function and returns an error if any issues occur. Finally, it calls the `proxy` function to handle incoming connections.  
  
#### FederatedServer.proxy:  
  
This function handles incoming connections by listening on the specified `listenAddr`. It then forwards the connection to a selected node based on the `workerTarget`, `loadBalanced`, or random selection. The selected node is identified using the `GetNode` function, and the connection is proxied using the `proxyP2PConnection` function. If load balancing is enabled, the `RecordRequest` function is called to track the request.  
  
#### sendHTMLResponse:  
  
This function sends a basic HTML response with a status code and a message to the client connection. The HTML content is defined separately for easier maintenance.  
  
#### getHTTPStatusText:  
  
This function returns a textual representation of HTTP status codes. It supports status codes 503, 404, and 200, and returns "Unknown Status" for any other status code.  
  
  
  
# core/p2p/node.go  
## Package: p2p  
  
### Imports:  
- sync  
- time  
  
### External Data, Input Sources:  
- None  
  
### NodeData Struct:  
- Represents a node in the peer-to-peer network.  
- Contains the following fields:  
    - Name: The name of the node.  
    - ID: The unique identifier of the node.  
    - TunnelAddress: The address of the node's tunnel.  
    - ServiceID: The ID of the service the node belongs to.  
    - LastSeen: The last time the node was seen.  
  
### IsOnline() Method:  
- Checks if a node is online based on the last seen time.  
- Returns true if the node was seen in the last 40 seconds, false otherwise.  
  
### Nodes Map:  
- A global map that stores all nodes in the network.  
- The map is keyed by service ID, and the value is a map of node IDs to NodeData structs.  
  
### GetAvailableNodes() Function:  
- Returns a list of available nodes for a given service ID.  
- If no service ID is provided, it defaults to the default service ID.  
- Iterates through the nodes map and returns all nodes for the specified service ID.  
  
### GetNode() Function:  
- Returns a NodeData struct for a given service ID and node ID.  
- If the service ID is not provided, it defaults to the default service ID.  
- Returns a boolean indicating whether the node exists in the nodes map.  
  
### AddNode() Function:  
- Adds a new node to the nodes map.  
- If the service ID is not provided, it defaults to the default service ID.  
- Locks the nodes map to ensure thread safety.  
  
  
  
# core/p2p/p2p.go  
```  
Package: p2p  
  
Imports:  
- context  
- errors  
- fmt  
- io  
- net  
- os  
- sync  
- time  
- github.com/ipfs/go-log  
- github.com/libp2p/go-libp2p/core/peer  
- pkg/utils  
- pkg/config  
- pkg/node  
- pkg/protocol  
- pkg/services  
- pkg/types  
- pkg/utils  
- github.com/phayes/freeport  
- github.com/rs/zerolog/log  
- pkg/logger  
  
External data, input sources:  
- os.Hostname()  
- os.Getenv("LOCALAI_P2P_DISABLE_DHT")  
- os.Getenv("LOCALAI_P2P_ENABLE_LIMITS")  
- os.Getenv("LOCALAI_LIBP2P_LOGLEVEL")  
  
Summary:  
The p2p package provides functionality for exposing and discovering services using a peer-to-peer network. It includes functions for generating new connection data, generating tokens, checking if P2P is enabled, generating node IDs, announcing nodes, and proxying connections between services.  
  
1. generateNewConnectionData: Generates a new connection data structure with random values for various fields, such as room name, rendezvous, and MDNS.  
  
2. GenerateToken: Generates a new connection data structure and returns its base64-encoded representation.  
  
3. IsP2PEnabled: Returns true if P2P is enabled, otherwise false.  
  
4. nodeID: Generates a unique node ID based on a given string and the hostname.  
  
5. nodeAnnounce: Announces a node to the ledger, updating its information and ensuring it's accepted by other nodes.  
  
6. proxyP2PConnection: Handles incoming connections, redirects them to the appropriate service, and closes the connection when done.  
  
7. allocateLocalService: Allocates a local port for listening, starts a goroutine to handle incoming connections, and forwards them to the appropriate service.  
  
8. ServiceDiscoverer: Continuously discovers new services, allocates them, and updates the ledger with their information.  
  
9. discoveryTunnels: Generates a channel of NodeData, which contains information about discovered services and their addresses.  
  
10. ensureService: Ensures that a service is started and running, and updates its information in the ledger.  
  
11. ExposeService: Creates a new node, registers the service, and starts it.  
  
12. NewNode: Creates a new node with the given token.  
  
13. newNodeOpts: Parses configuration options and returns a list of node options.  
  
14. copyStream: Copies data between two streams, closing the connection when done.  
  
```  
  
# core/p2p/p2p_common.go  
## Package: p2p  
  
### Imports:  
- os  
- strings  
  
### External Data and Input Sources:  
- os.Getenv("LOCALAI_P2P_LOGLEVEL") - This environment variable determines the logging level for the package.  
  
### Code Summary:  
#### Logging Level Configuration:  
The code defines a variable `logLevel` which determines the logging level for the package. It is initialized with the value of the environment variable `LOCALAI_P2P_LOGLEVEL`. If the environment variable is not set, the default logging level is set to `logLevelInfo`. The code also defines two constants, `logLevelDebug` and `logLevelInfo`, which represent the possible logging levels.  
  
#### Initialization:  
The `init()` function is called when the package is first loaded. It checks if the `logLevel` variable is empty. If it is, the default logging level, `logLevelInfo`, is assigned to the variable.  
  
# core/p2p/p2p_disabled.go  
## Package: p2p  
  
### Imports:  
- context  
- fmt  
- github.com/mudler/edgevpn/pkg/node  
  
### External Data, Input Sources:  
- DHTInterval (int)  
- OTPInterval (int)  
- ctx (context.Context)  
- node (node.Node)  
- token (string)  
- servicesID (string)  
- fn (func(string, NodeData))  
- allocate (bool)  
- host (string)  
- port (string)  
  
### Summary:  
#### GenerateToken:  
This function is responsible for generating a token based on the provided DHTInterval and OTPInterval. However, it is currently not implemented and returns "not implemented" as a placeholder.  
  
#### FederatedServer:  
This struct likely represents a federated server. The Start method is intended to start the server, but it is not implemented and returns an error indicating that it is not implemented.  
  
#### ServiceDiscoverer:  
This function is designed to discover services within a given context. It takes a context, a node, a token, a servicesID, a function to handle discovered services, and a boolean flag to indicate whether to allocate resources. However, it is not implemented and returns an error indicating that it is not implemented.  
  
#### ExposeService:  
This function is intended to expose a service by providing a host, port, token, and servicesID. It should return a node and an error, but it is not implemented and returns an error indicating that it is not implemented.  
  
#### IsP2PEnabled:  
This function checks if P2P is enabled and returns false, indicating that P2P is not enabled in this context.  
  
#### NewNode:  
This function is responsible for creating a new node using a provided token. However, it is not implemented and returns an error indicating that it is not implemented.  
  
  
  
