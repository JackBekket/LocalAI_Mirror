# Package: store

### Imports:

```
context
fmt
github.com/mudler/LocalAI/pkg/grpc
github.com/mudler/LocalAI/pkg/grpc/proto
```

### External Data, Input Sources:

- GRPC client (grpc.Backend)

### Code Summary:

The package provides a set of functions to interact with a key-value store using a GRPC client. The functions are designed to handle both single and multiple key-value pairs.

#### SetCols:

This function sets multiple key-value pairs in the store. It takes a context, a GRPC client, and two slices of data: keys and values. The keys and values are in columnar format, meaning that keys[i] is associated with values[i]. The function converts the input data into the required protobuf format and calls the StoresSet method of the GRPC client. If the operation is successful, it returns nil; otherwise, it returns an error message.

#### SetSingle:

This function sets a single key-value pair in the store. It takes a context, a GRPC client, a key, and a value. It calls the SetCols function with the provided key and value, effectively setting a single key-value pair.

#### DeleteCols:

This function deletes multiple key-value pairs from the store. It takes a context, a GRPC client, and a slice of keys. The keys are in columnar format, meaning that keys[i] is associated with values[i]. The function converts the input data into the required protobuf format and calls the StoresDelete method of the GRPC client. If the operation is successful, it returns nil; otherwise, it returns an error message.

#### DeleteSingle:

This function deletes a single key-value pair from the store. It takes a context, a GRPC client, and a key. It calls the DeleteCols function with the provided key, effectively deleting a single key-value pair.

#### GetCols:

This function gets multiple key-value pairs from the store. It takes a context, a GRPC client, and a slice of keys. The keys are in columnar format, meaning that keys[i] is associated with values[i]. The function converts the input data into the required protobuf format and calls the StoresGet method of the GRPC client. If the operation is successful, it returns the keys and values; otherwise, it returns an error message.

#### GetSingle:

This function gets a single key-value pair from the store. It takes a context, a GRPC client, and a key. It calls the GetCols function with the provided key, effectively getting a single key-value pair.

#### Find:

This function finds similar keys to the given key. It takes a context, a GRPC client, a key, and a topk value. The function converts the input data into the required protobuf format and calls the StoresFind method of the GRPC client. If the operation is successful, it returns the keys, values, and similarities; otherwise, it returns an error message.



