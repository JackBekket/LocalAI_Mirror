## Package: functions

### Imports:

- github.com/mudler/LocalAI/pkg/functions
- github.com/onsi/ginkgo/v2
- github.com/onsi/gomega

### External Data and Input Sources:

- Functions: []Function

### Summary:

The package provides functions for parsing function calls from various sources, such as text, JSON, and regex. It also includes functions for extracting notes from LLM results and converting single-quoted key-value pairs to double-quoted values.

#### ToJSONStructure()

This function converts a list of functions to a JSON structure that can be parsed to a grammar. It takes two arguments: the key for the function name and the key for the function arguments. The function returns a JSON structure with a list of one-of elements, where each element represents a function. The function name and arguments are extracted from the JSON structure and compared to the expected values.

#### Select()

This function selects one of the functions from a list of functions based on the provided function name. It returns a new list containing only the selected function. The function name is compared to the expected value.

#### ParseFunctionCall()

This function parses a function call from a given input string and returns a list of parsed function calls. It takes two arguments: the input string and a configuration object containing various options, such as the function name key, the function arguments key, and regex patterns for matching the function call.

#### ParseTextContent()

This function extracts notes from the LLM result based on the provided regex pattern. It takes two arguments: the input string and a configuration object containing the regex pattern.

#### ParseJSON()

This function parses a JSON string and returns a list of parsed JSON objects. It takes one argument: the input JSON string.

#### Edge Cases:

- When the input string is empty or invalid, the function should return an empty list or an error, respectively.
- When the input string contains multiple function calls, the function should parse each call and return a list of parsed function calls.
- When the input string contains invalid JSON, the function should return an error.

#### Environment Variables:

- None

#### Flags:

- None

#### Command-line Arguments:

- None

#### File Paths:

- None

#### Relations between Code Entities:

- The `ParseFunctionCall()` function is responsible for parsing function calls from various sources, such as text, JSON, and regex.
- The `ParseTextContent()` function extracts notes from the LLM result based on the provided regex pattern.
- The `ParseJSON()` function parses a JSON string and returns a list of parsed JSON objects.

#### Dead Code:

- None