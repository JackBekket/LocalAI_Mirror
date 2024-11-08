## Package: cli

### Imports:

- context
- fmt
- os
- path/filepath
- strings

- github.com/mudler/LocalAI/core/backend
- github.com/mudler/LocalAI/core/cli/context
- github.com/mudler/LocalAI/core/config
- github.com/mudler/LocalAI/pkg/model
- github.com/rs/zerolog/log

### External Data, Input Sources:

- `LOCALAI_MODELS_PATH` or `MODELS_PATH` environment variable: Path containing models used for inferencing.
- `LOCALAI_BACKEND_ASSETS_PATH` or `BACKEND_ASSETS_PATH` environment variable: Path used to extract libraries that are required by some of the backends in runtime.

### Code Summary:

#### TTSCMD struct:

This struct defines the command-line arguments for the text-to-speech (TTS) functionality. It includes fields for the text to be converted, the backend to use, the model to run, the voice to use, the language to use, the output file path, and paths for models and backend assets.

#### Run method:

This method is responsible for executing the TTS process. It first determines the output directory based on the provided output file path or the backend assets path. Then, it joins the text input into a single string.

Next, it creates an `ApplicationConfig` object with the specified model path, context, audio directory, and assets destination. It also initializes a model loader using the provided model path.

The method then calls the `ModelTTS` function from the `backend` package to perform the TTS conversion. This function takes the backend, text, model, voice, language, model loader, application configuration, and backend configuration as input. It returns the file path of the generated audio file, the model name, and any error encountered.

Finally, the method renames the generated audio file to the specified output file path if provided, or prints the file path of the generated audio file otherwise. It also ensures that all GRPC processes are stopped when the method exits.

