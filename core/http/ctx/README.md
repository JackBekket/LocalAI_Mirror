# Package: fiberContext

### Imports:

- fmt
- strings
- github.com/gofiber/fiber/v2
- github.com/mudler/LocalAI/core/config
- github.com/mudler/LocalAI/core/services
- github.com/mudler/LocalAI/pkg/model
- github.com/rs/zerolog/log

### External Data, Input Sources:

- `ctx`: fiber.Ctx object containing request context
- `cl`: config.BackendConfigLoader object for loading backend configuration
- `loader`: model.ModelLoader object for loading models
- `modelInput`: string representing the model name requested by the user
- `firstModel`: boolean indicating whether to use the first available model if no model is specified

### ModelFromContext Function:

This function takes a fiber.Ctx object, a config.BackendConfigLoader object, a model.ModelLoader object, a string representing the model name requested by the user, and a boolean indicating whether to use the first available model if no model is specified. It returns the resolved model name and an error if any.

The function first checks if a model is specified in the request parameters or query parameters. If a model is found, it is used as the input model. Otherwise, it checks if a model is available in the bearer token. If a model is found in the bearer token, it is used as the input model.

If no model is specified in the request parameters, query parameters, or bearer token, and the `firstModel` flag is true, the function lists all available models and uses the first one as the input model. If no models are available, it returns an error.

Finally, the function returns the resolved model name and an error if any.

