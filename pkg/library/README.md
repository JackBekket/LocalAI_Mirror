# Package: library

### Imports:

```
errors
fmt
os
path/filepath
runtime
github.com/rs/zerolog/log
```

### External Data, Input Sources:

- `LOCALAI_SKIP_LIBRARY_PATH` environment variable: Determines whether to skip loading libraries from the asset directory.
- `dir` parameter in `LoadExtractedLibs` function: Specifies the directory containing the asset files.
- `args` parameter in `LoadLDSO` function: Contains the arguments for the grpc process.
- `grpcProcess` parameter in `LoadLDSO` function: Specifies the executable for the grpc process.
- `assetDir` parameter in `LoadLDSO` function: Specifies the directory containing the asset files.

### Summary:

#### LoadExtractedLibs:

This function loads the extracted libraries from the asset directory. It first checks if the `LOCALAI_SKIP_LIBRARY_PATH` environment variable is set. If it is, the function returns an error. Otherwise, it iterates through the specified directories and calls the `LoadExternal` function for each directory. The function returns an error if any of the calls to `LoadExternal` return an error.

#### LoadLDSO:

This function checks if there is a ld.so file in the asset directory and if so, prefixes the grpc process with it. This is done to ensure compatibility with different versions of ld.so. The function first checks if the `LOCALAI_SKIP_LIBRARY_PATH` environment variable is set. If it is, the function returns the original arguments and grpc process. Otherwise, it checks if the operating system is Linux. If it is, it checks if there is a ld.so file in the asset directory. If it is found, the function appends the ld.so file to the grpc process arguments and sets the `grpcProcess` variable to the path of the ld.so file.

#### LoadExternal:

This function sets the LD_LIBRARY_PATH environment variable to include the given directory. It first checks if the `LOCALAI_SKIP_LIBRARY_PATH` environment variable is set. If it is, the function returns an error. Otherwise, it checks if the directory exists. If it does, it retrieves the current value of the LD_LIBRARY_PATH environment variable and appends the given directory to it. The function then sets the LD_LIBRARY_PATH environment variable to the new value.



