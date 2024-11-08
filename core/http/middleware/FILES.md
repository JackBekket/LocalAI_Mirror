# core/http/middleware/auth.go  
## Package: middleware  
  
This package contains the configuration generators and handler functions that are used along with the fiber/keyauth middleware. Currently, this requires an upstream patch - and feature patches are no longer accepted to v2. Therefore, `dave-gray101/v2keyauth` contains the v2 backport of the middleware until v3 stabilizes and we migrate.  
  
### Imports:  
  
- crypto/subtle  
- errors  
- github.com/dave-gray101/v2keyauth  
- github.com/gofiber/fiber/v2  
- github.com/gofiber/fiber/v2/middleware/keyauth  
- github.com/microcosm-cc/bluemonday  
- github.com/mudler/LocalAI/core/config  
  
### External Data, Input Sources:  
  
- config.ApplicationConfig: This struct contains the application configuration, including API keys, opaque errors, and other settings related to the API key authentication.  
  
### Code Summary:  
  
#### GetKeyAuthConfig:  
  
This function takes an instance of the `config.ApplicationConfig` struct and returns a `v2keyauth.Config` struct, which is used to configure the key authentication middleware. It sets up a custom key lookup function that checks for API keys in the "Authorization", "x-api-key", and "xi-api-key" headers. It also sets up the validator, error handler, and auth scheme for the middleware.  
  
#### getApiKeyErrorHandler:  
  
This function is the error handler for the key authentication middleware. It checks if the error is due to a missing or malformed API key. If so, it returns a 403 Forbidden status code if the application configuration specifies opaque errors. Otherwise, it returns a 403 Forbidden status code with a sanitized error message.  
  
#### getApiKeyValidationFunction:  
  
This function is the API key validation function for the key authentication middleware. It takes an instance of the `fiber.Ctx` struct and an API key as input. If the application configuration specifies the use of subtle key comparison, it uses the `subtle.ConstantTimeCompare` function to compare the provided API key with the valid API keys stored in the application configuration. Otherwise, it performs a simple string comparison.  
  
#### getApiKeyRequiredFilterFunction:  
  
This function is the API key requirement filter function for the key authentication middleware. It checks if the HTTP method is GET and if the request path matches any of the exempted endpoints specified in the application configuration. If so, it returns true, indicating that the API key is not required for this request. Otherwise, it returns false, indicating that the API key is required.  
  
  
  
