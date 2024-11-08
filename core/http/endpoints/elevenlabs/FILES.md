# core/http/endpoints/elevenlabs/soundgeneration.go  
## Package: elevenlabs  
  
### Imports:  
  
- github.com/gofiber/fiber/v2  
- github.com/mudler/LocalAI/core/backend  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/http/ctx  
- github.com/mudler/LocalAI/core/schema  
- github.com/mudler/LocalAI/pkg/model  
- github.com/rs/zerolog/log  
  
### External Data, Input Sources:  
  
- The code uses a configuration file to load backend configuration.  
- It also uses a model loader to load models.  
- The input data is taken from the request body, which is expected to be a schema.ElevenLabsSoundGenerationRequest object.  
  
### SoundGenerationEndpoint:  
  
This function handles the ElevenLabs SoundGeneration endpoint. It takes three arguments:  
  
1. cl: A config.BackendConfigLoader object.  
2. ml: A model.ModelLoader object.  
3. appConfig: A config.ApplicationConfig object.  
  
The function returns a closure that takes a fiber.Ctx object as input and returns an error.  
  
Inside the closure, the input data is parsed from the request body. Then, the model file is retrieved from the context or from the input data. The backend configuration is loaded based on the model file and the application configuration.  
  
The SoundGeneration function from the backend package is called with the model file, input text, duration, temperature, and other parameters. The function returns a file path to the generated audio file. Finally, the audio file is downloaded to the client.  
  
  
  
# core/http/endpoints/elevenlabs/tts.go  
## Package: elevenlabs  
  
### Imports:  
  
```  
github.com/mudler/LocalAI/core/backend  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/core/http/ctx  
github.com/mudler/LocalAI/pkg/model  
github.com/gofiber/fiber/v2  
github.com/mudler/LocalAI/core/schema  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
1. `cl`: BackendConfigLoader  
2. `ml`: ModelLoader  
3. `appConfig`: ApplicationConfig  
4. Request body: schema.ElevenLabsTTSRequest  
  
### Code Summary:  
  
#### TTSEndpoint:  
  
This function handles the OpenAI Speech API endpoint for text-to-speech conversion. It takes the following parameters:  
  
1. `cl`: BackendConfigLoader  
2. `ml`: ModelLoader  
3. `appConfig`: ApplicationConfig  
  
The function first parses the input data from the request body and extracts the voice ID from the URL parameters. It then retrieves the model file based on the input model ID and the context. If the model is not found in the context, it uses the input model ID as the model file.  
  
Next, the function loads the backend configuration file for the specified model and voice ID. If the model is not found in the context, it uses the input model ID as the model file.  
  
Finally, the function calls the backend.ModelTTS function to generate the audio file and returns the download link for the generated audio file.  
  
