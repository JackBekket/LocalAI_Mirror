# Package: oci_test

### Imports:

- os
- github.com/mudler/LocalAI/pkg/oci
- github.com/onsi/ginkgo/v2
- github.com/onsi/gomega

### External Data, Input Sources:

- registry.ollama.ai/library/gemma
- sha256:c1864a5eb19305c40519da12cc543519e48a0697ecd30e15d5ac228644957d12

### Summary:

#### Pulling Images:

- The code defines a test suite for the OCI package, specifically focusing on image pulling functionality.
- It uses the `Describe` and `Context` functions from the `github.com/onsi/ginkgo/v2` package to structure the test suite.
- The `It` function is used to define individual test cases.
- In the test case "should fetch blobs correctly", the code creates a temporary file and attempts to fetch an image blob from the specified registry and digest.
- The `FetchImageBlob` function from the `github.com/mudler/LocalAI/pkg/oci` package is used to perform the image blob retrieval.
- The test case asserts that the blob retrieval is successful by checking for errors using the `Expect` function from the `github.com/onsi/gomega` package.



pkg/oci/image_test.go
## Package: oci_test

### Imports:

- os
- runtime
- github.com/mudler/LocalAI/pkg/oci
- github.com/onsi/ginkgo/v2
- github.com/onsi/gomega

### External Data, Input Sources:

- The code uses the `GetImage` function from the `oci` package to retrieve an image based on the provided image name, which is "alpine" in this case.
- The code also uses the `GetOCIImageSize` function to get the size of the image.

### Summary:

#### Context: when template is loaded successfully

- The code first checks if the current operating system is Darwin (macOS). If it is, the test is skipped.
- It then retrieves an image using the `GetImage` function with the image name "alpine".
- The code then gets the size of the image using the `GetOCIImageSize` function.
- It then creates a temporary directory using `os.MkdirTemp` and extracts the contents of the image to this directory using the `ExtractOCIImage` function.

#### It: should evaluate the template correctly

- The code checks if the size of the extracted image is not equal to 0, which indicates that the image was successfully extracted.

#### End of output:



pkg/oci/oci_suite_test.go
Package: oci_test

Imports:
- testing
- github.com/onsi/ginkgo/v2
- github.com/onsi/gomega

External data, input sources:
- None

Summary:
### TestOCI function
The TestOCI function is the entry point for the OCI test suite. It registers the Fail handler and runs the test suite using the RunSpecs function. The test suite is named "OCI test suite".

pkg/oci/ollama_test.go
## Package: oci_test

### Imports:

- os
- github.com/mudler/LocalAI/pkg/oci
- github.com/onsi/ginkgo/v2
- github.com/onsi/gomega

### External Data, Input Sources:

- None

### Summary:

#### Describe("OCI", func() {

This section describes the OCI package and its functionality.

#### Context("ollama", func() {

This context focuses on the "ollama" model and its related operations.

#### It("pulls model files", func() {

This test case checks if the OllamaFetchModel function successfully downloads the model files for the "gemma:2b" model. It creates a temporary file, calls the OllamaFetchModel function to download the model files, and then removes the temporary file.



