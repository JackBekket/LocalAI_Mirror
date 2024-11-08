## Package: oci

### Imports:

* context
* fmt
* io
* os
* ocispec "github.com/opencontainers/image-spec/specs-go/v1"
* oras "oras.land/oras-go/v2"
* "oras.land/oras-go/v2/registry/remote"

### External Data, Input Sources:

* `r`: Remote repository URL
* `reference`: Image reference (e.g., "library/image:tag")
* `dst`: Output file path
* `statusReader`: Function to write status information to a writer

### Function: FetchImageBlob

The `FetchImageBlob` function downloads an image blob from a remote repository and writes it to a file. It takes the following arguments:

1. `r`: Remote repository URL
2. `reference`: Image reference (e.g., "library/image:tag")
3. `dst`: Output file path
4. `statusReader`: Function to write status information to a writer

The function first creates a file store for the output file. Then, it connects to the remote repository and fetches the image blob using the `oras.Fetch` function. If a `statusReader` is provided, the function writes the blob to the file store and the status reader. Otherwise, it only writes the blob to the file store. Finally, the function returns an error if any of the steps fail.

pkg/oci/image.go
## Package: oci

### Imports:

```
"context"
"errors"
"fmt"
"io"
"net/http"
"runtime"
"strings"
"syscall"
"time"

"github.com/containerd/containerd/archive"
registrytypes "github.com/docker/docker/api/types/registry"
"github.com/google/go-containerregistry/pkg/authn"
"github.com/google/go-containerregistry/pkg/logs"
"github.com/google/go-containerregistry/pkg/name"
v1 "github.com/google/go-containerregistry/pkg/v1"
"github.com/google/go-containerregistry/pkg/v1/mutate"
"github.com/google/go-containerregistry/pkg/v1/remote"
"github.com/google/go-containerregistry/pkg/v1/remote/transport"
```

### External Data, Input Sources:

- `targetImage`: The image to extract.
- `targetDestination`: The directory to extract the image to.
- `targetPlatform`: The platform to use for the image.
- `auth`: An optional authentication configuration.
- `t`: An optional HTTP round-tripper.

### Code Summary:

#### ExtractOCIImage:

This function extracts a given targetImage into a given targetDestination. It uses the mutate.Extract function to create a reader for the image, and then uses the archive.Apply function to extract the image to the target destination.

#### ParseImageParts:

This function parses an image string and returns the tag, repository, and image parts. It first checks if the image string contains a colon, which indicates a tag. If so, it splits the string by the colon and assigns the first part to the repository and the second part to the tag. If the image string contains a slash, it splits the string by the slash and assigns the first part to the repository and the second part to the image.

#### GetImage:

This function returns the proper image to pull with transport and auth. It first tries to use the local daemon, and if that fails, it falls back to a remote image. It takes the targetImage, targetPlatform, auth, and t as input parameters. It parses the targetImage and targetPlatform, and then uses the remote.Image function to get the image.

#### GetOCIImageSize:

This function returns the size of the given image. It first calls the GetImage function to get the image, and then iterates over the layers of the image to calculate the total size.

pkg/oci/ollama.go
## Package: oci

### Imports:

- encoding/json
- fmt
- io
- net/http
- ocispec

### External Data, Input Sources:

- The package uses the `registry.ollama.ai` URL to fetch the Ollama model manifest and blob.
- It also uses the `ParseImageParts` function to parse the image string and extract the repository, tag, and image parts.

### Code Summary:

#### OllamaModelManifest Function:

- This function takes an image string as input and returns a Manifest struct containing the model manifest data.
- It first parses the repository, tag, and image name from the input image string.
- Then, it constructs a GET request to the Ollama registry to fetch the manifest for the specified image.
- The response is parsed as JSON, and the resulting Manifest struct is returned.

#### OllamaModelBlob Function:

- This function takes an image string as input and returns the digest of the layer containing the Ollama model blob.
- It first calls the OllamaModelManifest function to get the manifest data.
- Then, it iterates through the layers in the manifest and returns the digest of the layer with the media type "application/vnd.ollama.image.model".

#### OllamaFetchModel Function:

- This function takes an image string, output path, and a status writer function as input and returns an error if any.
- It first calls the OllamaModelBlob function to get the digest of the layer containing the Ollama model blob.
- Then, it calls the FetchImageBlob function to download the blob to the specified output path, using the status writer function to update the progress.



