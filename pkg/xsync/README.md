## Package: xsync

### Imports:

```
import (
	"sync"
)
```

### External Data, Input Sources:

None

### SyncedMap:

The `SyncedMap` struct is a thread-safe map implementation that uses a `sync.RWMutex` to protect the underlying map from concurrent access. It provides methods for accessing, modifying, and iterating over the map in a thread-safe manner.

#### Methods:

1. `NewSyncedMap[K comparable, V any]() *SyncedMap[K, V]`: Creates a new `SyncedMap` with an empty map.

2. `Map() map[K]V`: Returns a read-only view of the underlying map.

3. `Get(key K) V`: Retrieves the value associated with the given key.

4. `Keys() []K`: Returns a slice containing all keys in the map.

5. `Values() []V`: Returns a slice containing all values in the map.

6. `Len() int`: Returns the number of key-value pairs in the map.

7. `Iterate(f func(key K, value V) bool)`: Iterates over the map, calling the provided function for each key-value pair.

8. `Set(key K, value V)`: Sets the value associated with the given key.

9. `Delete(key K)`: Removes the key-value pair associated with the given key.

10. `Exists(key K) bool`: Checks if a key exists in the map.



File structure:

- map.go
- map_test.go
- sync_suite_test.go
 pkg/xsync/map.go

