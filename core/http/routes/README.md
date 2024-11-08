## Package: routes

### Imports:

- github.com/gofiber/fiber/v2
- github.com/mudler/LocalAI/core/config
- github.com/mudler/LocalAI/core/http/endpoints/localai
- github.com/mudler/LocalAI/core/http/endpoints/openai
- github.com/mudler/LocalAI/core/p2p
- github.com/mudler/LocalAI/core/services
- github.com/mudler/LocalAI/internal
- github.com/mudler/LocalAI/pkg/model
- github.com/mudler/LocalAI/pkg/xsync
- github.com/rs/zerolog/log

### External Data, Input Sources:

- cl: BackendConfigLoader
- ml: ModelLoader
- appConfig: ApplicationConfig
- galleryService: GalleryService

### Summary:

This package contains the routes for the LocalAI application. It registers various endpoints for different functionalities, such as chat, text-to-image generation, and model management.

### RegisterUIRoutes Function:

This function registers the UI routes for the application. It takes the Fiber app, BackendConfigLoader, ModelLoader, ApplicationConfig, and GalleryService as input.

1. It registers a route for the home page, which displays the available models and their status.
2. It registers routes for chat functionality, allowing users to interact with different models.
3. It registers routes for text-to-image generation, enabling users to generate images based on text prompts.
4. It registers routes for managing models, including installing, deleting, and viewing model details.
5. It registers routes for the P2P dashboard, which displays information about the P2P network and nodes.

### Edge Cases:

- If no models are available, the application redirects to the home page, suggesting how to install models.
- If the P2P network is not enabled, the P2P dashboard routes are not registered.

### File Structure:

- routes/elevenlabs.go
- routes/explorer.go
- routes/health.go
- routes/jina.go
- routes/localai.go
- routes/openai.go
- routes/ui.go

