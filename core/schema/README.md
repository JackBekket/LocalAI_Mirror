# schema

## Package: schema

### Imports:

```
import (
	"context"
	"time"

	functions "github.com/mudler/LocalAI/pkg/functions"
)
```

### External Data, Input Sources:

- OpenAI API
- LocalAI/pkg/functions package

### APIError
- Provides error information returned by the OpenAI API.
- Fields:
    - Code: any
    - Message: string
    - Param: *string
    - Type: string

### ErrorResponse
- Contains an APIError.

### OpenAIUsage
- Represents usage information for OpenAI API calls.
- Fields:
    - PromptTokens: int
    - CompletionTokens: int
    - TotalTokens: int

### Item
- Represents an item in a list.
- Fields:
    - Embedding: []float32
    - Index: int
    - Object: string
    - URL: string
    - B64JSON: string

### OpenAIResponse
- Represents a response from the OpenAI API.
- Fields:
    - Created: int
    - Object: string
    - ID: string
    - Model: string
    - Choices: []Choice
    - Data: []Item
    - Usage: OpenAIUsage

### Choice
- Represents a choice in a list of choices.
- Fields:
    - Index: int
    - FinishReason: string
    - Message: *Message
    - Delta: *Message
    - Text: string

### Content
- Represents a piece of content.
- Fields:
    - Type: string
    - Text: string
    - ImageURL: ContentURL
    - AudioURL: ContentURL
    - VideoURL: ContentURL

### ContentURL
- Represents a URL for content.
- Field:
    - URL: string

### Message
- Represents a message.
- Fields:
    - Role: string
    - Name: string
    - Content: interface{}
    - StringContent: string
    - StringImages: []string
    - StringVideos: []string
    - StringAudios: []string
    - FunctionCall: interface{}
    - ToolCalls: []ToolCall

### ToolCall
- Represents a tool call.
- Fields:
    - Index: int
    - ID: string
    - Type: string
    - FunctionCall: FunctionCall

### FunctionCall
- Represents a function call.
- Fields:
    - Name: string
    - Arguments: string

### OpenAIModel
- Represents an OpenAI model.
- Fields:
    - ID: string
    - Object: string

### DeleteAssistantResponse
- Represents a response from deleting an assistant.
- Fields:
    - ID: string
    - Object: string
    - Deleted: bool

### File
- Represents a file object from the OpenAI API.
- Fields:
    - ID: string
    - Object: string
    - Bytes: int
    - CreatedAt: time.Time
    - Filename: string
    - Purpose: string

### ListFiles
- Represents a list of files.
- Fields:
    - Data: []File
    - Object: string

### AssistantFileRequest
- Represents a request to delete an assistant file.
- Field:
    - FileID: string

### DeleteAssistantFileResponse
- Represents a response from deleting an assistant file.
- Fields:
    - ID: string
    - Object: string
    - Deleted: bool

### ImageGenerationResponseFormat
- Represents the format of an image generation response.

### ChatCompletionResponseFormatType
- Represents the type of a chat completion response format.

### ChatCompletionResponseFormat
- Represents a chat completion response format.
- Field:
    - Type: ChatCompletionResponseFormatType

### JsonSchemaRequest
- Represents a request to get a JSON schema.
- Fields:
    - Type: string
    - JsonSchema: JsonSchema

### JsonSchema
- Represents a JSON schema.
- Fields:
    - Name: string
    - Strict: bool
    - Schema: functions.Item

### OpenAIRequest
- Represents a request to the OpenAI API.
- Fields:
    - PredictionOptions
    - Context: context.Context
    - Cancel: context.CancelFunc
    - File: string
    - ResponseFormat: interface{}
    - Size: string
    - Prompt: interface{}
    - Instruction: string
    - Input: interface{}
    - Stop: interface{}
    - Messages: []Message
    - Functions: functions.Functions
    - FunctionCall: interface{}
    - Tools: []functions.Tool
    - ToolsChoice: interface{}
    - Stream: bool
    - Mode: int
    - Step: int
    - Grammar: string
    - JSONFunctionGrammarObject: *functions.JSONFunctionStructure
    - Backend: string
    - ModelBaseName: string

### ModelsDataResponse
- Represents a response containing a list of OpenAI models.
- Fields:
    - Object: string
    - Data: []OpenAIModel



core/schema/prediction.go
## Package: schema

### Imports:

None

### External Data, Input Sources:

None

### PredictionOptions struct:

The `PredictionOptions` struct defines the parameters for making predictions using various AI models. It includes both standard options from the OpenAI API and custom parameters specific to this package.

#### Standard OpenAI API options:

- `Model`: The name of the AI model to use.
- `Language`: The language of the input and output.
- `Translate`: Whether to translate the input text to another language (only for audio transcription).
- `N`: The number of results to return.
- `TopP`: The top-p sampling parameter.
- `TopK`: The top-k sampling parameter.
- `Temperature`: The temperature parameter for sampling.
- `Maxtokens`: The maximum number of tokens to generate.
- `Echo`: Whether to echo the input text in the output.

#### Custom parameters:

- `Batch`: The batch size for processing multiple inputs.
- `IgnoreEOS`: Whether to ignore the end-of-sequence token.
- `RepeatPenalty`: The penalty for repeating tokens.
- `RepeatLastN`: The number of previous tokens to consider when calculating the repeat penalty.
- `Keep`: The number of top-k tokens to keep.
- `FrequencyPenalty`: The penalty for using frequent tokens.
- `PresencePenalty`: The penalty for using tokens that are already present in the output.
- `TFZ`: The TFZ parameter for controlling the generation process.
- `TypicalP`: The typical probability parameter.
- `Seed`: The random seed for reproducibility.
- `NegativePrompt`: A string containing tokens to avoid in the output.
- `RopeFreqBase`: The base frequency for rope sampling.
- `RopeFreqScale`: The scale factor for rope sampling.
- `NegativePromptScale`: The scale factor for negative prompt tokens.
- `UseFastTokenizer`: Whether to use a fast tokenizer.
- `ClipSkip`: The clip skip parameter for diffusion models.
- `Tokenizer`: The tokenizer to use for text processing.



core/schema/tokenize.go
## Package: schema

### Imports:

- json

### External Data, Input Sources:

- TokenizeRequest:
    - Content: string
    - Model: string
- TokenizeResponse:
    - Tokens: []int32

### Code Summary:

#### TokenizeRequest:

This struct represents the input for the Tokenize function. It contains two fields: Content and Model. Content is a string that represents the text to be tokenized, and Model is a string that specifies the tokenizer model to be used.

#### TokenizeResponse:

This struct represents the output of the Tokenize function. It contains a single field: Tokens, which is a slice of int32 values representing the tokenized text.



core/schema/transcription.go
## Package: schema

### Imports:

- time

### External Data and Input Sources:

- None

### Code Summary:

#### Segment Struct:

The `Segment` struct represents a segment of a transcription. It has the following fields:

- `Id`: An integer representing the unique identifier of the segment.
- `Start`: A `time.Duration` representing the start time of the segment.
- `End`: A `time.Duration` representing the end time of the segment.
- `Text`: A string representing the text content of the segment.
- `Tokens`: A slice of integers representing the token indices corresponding to the text content of the segment.

#### TranscriptionResult Struct:

The `TranscriptionResult` struct represents the overall transcription result. It has the following fields:

- `Segments`: A slice of `Segment` structs representing the individual segments of the transcription.
- `Text`: A string representing the complete text content of the transcription.

