# core/explorer/database.go  
## Package: explorer  
  
This package provides a simple JSON database for storing and retrieving p2p network tokens, along with their names and descriptions.  
  
### Imports:  
  
- encoding/json  
- os  
- sort  
- sync  
- github.com/gofrs/flock  
  
### External Data, Input Sources:  
  
- The package uses a JSON file to store and retrieve data. The file path is provided as an argument to the `NewDatabase` function.  
  
### Code Summary:  
  
#### Database Structure:  
  
The `Database` struct is responsible for managing the data and providing methods for interacting with the JSON file. It contains the following fields:  
  
- `path`: The path to the JSON file.  
- `data`: A map that stores the token data, where the key is the token and the value is a `TokenData` struct.  
- `flock`: A flock object used for file locking to ensure that multiple processes can access the file safely.  
- `sync.Mutex`: A mutex to protect the data from concurrent access.  
  
#### TokenData Structure:  
  
The `TokenData` struct represents a p2p network token and contains the following fields:  
  
- `Name`: The name of the token.  
- `Description`: A description of the token.  
- `Clusters`: A list of `ClusterData` structs, which contain information about the clusters associated with the token.  
- `Failures`: The number of failures associated with the token.  
  
#### ClusterData Structure:  
  
The `ClusterData` struct represents a cluster and contains the following fields:  
  
- `Workers`: A list of worker nodes in the cluster.  
- `Type`: The type of the cluster.  
- `NetworkID`: The network ID of the cluster.  
  
#### Database Methods:  
  
- `NewDatabase(path string) (*Database, error)`: Creates a new Database instance with the given path.  
- `Get(token string) (TokenData, bool)`: Retrieves a Token from the Database by its token.  
- `Set(token string, t TokenData) error`: Stores a Token in the Database by its token.  
- `Delete(token string) error`: Removes a Token from the Database by its token.  
- `TokenList() []string`: Returns a sorted list of all tokens in the Database.  
  
#### Load and Save Methods:  
  
- `load() error`: Reads the Database from disk.  
- `save() error`: Writes the Database to disk.  
  
The package provides a simple and efficient way to store and retrieve p2p network token data using a JSON database. The use of file locking ensures that multiple processes can access the data safely, while the sorting of tokens in the `TokenList` method provides a convenient way to retrieve the tokens in a specific order.  
  
# core/explorer/discovery.go  
## Package: explorer  
  
### Imports:  
  
```  
"context"  
"fmt"  
"strings"  
"sync"  
"time"  
  
"github.com/rs/zerolog/log"  
  
"github.com/mudler/LocalAI/core/p2p"  
"github.com/mudler/edgevpn/pkg/blockchain"  
```  
  
### External Data, Input Sources:  
  
- Database: The code uses a Database to store and retrieve token information, network data, and connection failures.  
- Context: The code uses a context to manage the lifetime of the discovery server and its goroutines.  
- Blockchain: The code uses a blockchain library to interact with the network and retrieve network data.  
  
### Major Code Parts:  
  
#### DiscoveryServer:  
  
- The DiscoveryServer struct manages the discovery process and keeps the database in sync with the network state.  
- It has a mutex to protect the shared state, a database, a connection time, and an error threshold.  
- The NewDiscoveryServer function creates a new DiscoveryServer instance with the given database, connection time, and error threshold.  
  
#### Network:  
  
- The Network struct represents the network data, which includes a list of clusters.  
  
#### runBackground:  
  
- This function is responsible for running the discovery process in the background.  
- It iterates through the tokens in the database and attempts to connect to the network for each token.  
- It retrieves network data, updates the database with the latest information, and handles failed connections.  
  
#### failedToken:  
  
- This function is called when a token fails to connect to the network or retrieve network data.  
- It increments the failure count for the token in the database.  
  
#### deleteFailedConnections:  
  
- This function removes tokens from the database if they have exceeded the error threshold.  
  
#### retrieveNetworkData:  
  
- This function retrieves network data from the blockchain and stores it in a map.  
- It iterates through the data and extracts cluster information, including the type, network ID, and workers.  
  
#### Start:  
  
- This function starts the discovery server and runs the discovery process in a goroutine.  
- It takes a context and a boolean flag to control whether the server should keep running or not.  
  
  
  
# core/http/endpoints/explorer/dashboard.go  
## Package: explorer  
  
### Imports:  
- encoding/base64  
- sort  
- github.com/gofiber/fiber/v2  
- github.com/mudler/LocalAI/core/explorer  
- github.com/mudler/LocalAI/internal  
  
### External Data, Input Sources:  
- Database (explorer.Database)  
  
### Dashboard:  
This function returns a handler for the dashboard route. It creates a summary map with the title and version of the LocalAI API. Based on the client's request, it either returns a JSON response with the summary or renders the "explorer" view with the summary data.  
  
### AddNetworkRequest:  
This struct defines the request body for adding a new network. It includes fields for the token, name, and description of the network.  
  
### Network:  
This struct represents a network and includes the TokenData from the explorer package and the token string.  
  
### ShowNetworks:  
This function returns a handler for the route that shows all networks. It iterates through the token list in the database, retrieves the corresponding network data, and checks if the network has any workers. If both conditions are met, the network is added to the results list. The results are then sorted by the number of clusters in descending order. Finally, the sorted list of networks is returned as a JSON response.  
  
### AddNetwork:  
This function returns a handler for the route that adds a new network. It parses the request body, validates the required fields (token, name, and description), and checks if the token is valid. If the token is valid and doesn't already exist in the database, it adds the new network to the database and returns a success message. Otherwise, it returns an error message.  
  
  
  
