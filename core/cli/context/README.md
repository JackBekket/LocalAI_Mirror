# cliContext

This package provides a CLI context for the application. It includes a Context struct that stores configuration settings for the CLI, such as debug mode and log level. The package also includes an embed.FS field for storing backend assets.

Environment variables:
- LOCALAI_DEBUG
- LOCALAI_LOG_LEVEL

Flags:
- --log-level

Files:
- context.go
- core/cli/context/context.go

Edge cases:
- The Debug field in the Context struct is deprecated and should be replaced with the --log-level=debug flag.

Project package structure:
- context.go
- core/cli/context/context.go

Relations between code entities:
- The Context struct is used to store the configuration for the CLI.
- The Debug field in the Context struct is deprecated and should be replaced with the --log-level=debug flag.
- The BackendAssets field in the Context struct is used to store backend assets.

