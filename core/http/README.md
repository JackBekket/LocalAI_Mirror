## Package: http

### Imports:

```
embed
errors
fmt
net/http

github.com/dave-gray101/v2keyauth
github.com/mudler/LocalAI/pkg/utils

github.com/mudler/LocalAI/core/http/endpoints/localai
github.com/mudler/LocalAI/core/http/endpoints/openai
github.com/mudler/LocalAI/core/http/middleware
github.com/mudler/LocalAI/core/http/routes

github.com/mudler/LocalAI/core/config
github.com/mudler/LocalAI/core/schema
github.com/mudler/LocalAI/core/services
github.com/mudler/LocalAI/pkg/model

github.com/gofiber/contrib/fiberzerolog
github.com/gofiber/fiber/v2
github.com/gofiber/fiber/v2/middleware/cors
github.com/gofiber/fiber/v2/middleware/csrf
github.com/gofiber/fiber/v2/middleware/favicon
github.com/gofiber/fiber/v2/middleware/filesystem
github.com/gofiber/fiber/v2/middleware/recover

github.com/rs/zerolog/log
```

### External Data, Input Sources:

1. Config files:
    - `appConfig.UploadDir`
    - `appConfig.ConfigsDir`
2. Uploaded files:
    - `openai.UploadedFilesFile`
    - `openai.AssistantsConfigFile`
    - `openai.AssistantsFileConfigFile`

### Code Summary:

1. **App Initialization:**
    - Creates a new Fiber app instance with custom configuration, including error handling, body limit, and startup message handling.
    - Sets up a custom error handler to return JSON responses with error details or a blank 500 response if OpaqueErrors are enabled.
    - Registers a startup log message to inform the user when the API is listening.
    - Uses zerolog as the logger for the application.

2. **Middleware Configuration:**
    - Enables recover middleware for error handling.
    - Enables metrics middleware if metrics are enabled in the configuration.
    - Registers health check routes.
    - Creates a KeyAuth configuration and applies it to all endpoints.
    - Enables CORS middleware if enabled in the configuration.
    - Enables CSRF middleware if enabled in the configuration.

3. **Static File Handling:**
    - Loads static files from the embedded directory.
    - Serves static files using the filesystem middleware.

4. **Route Registration:**
    - Registers routes for ElevenLabs, LocalAI, OpenAI, and JINA services.
    - Registers UI routes if the web UI is enabled.

5. **Gallery Service:**
    - Creates a new gallery service and starts it.

6. **404 Handler:**
    - Defines a custom 404 handler to handle missing routes.

7. **Return App Instance:**
    - Returns the configured Fiber app instance.



