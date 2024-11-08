# core/cli/context/context.go  
Package: cliContext  
  
Imports:  
- "embed"  
  
External data, input sources:  
- Environment variables: LOCALAI_DEBUG, LOCALAI_LOG_LEVEL  
  
### Context struct  
  
The Context struct is used to store the configuration for the CLI. It has three fields:  
  
- Debug: A boolean flag that controls whether debug logging is enabled. It is deprecated and should be replaced with the --log-level=debug flag.  
- LogLevel: A string flag that controls the level of logs to output. It can take values from the enum: error, warn, info, debug, trace.  
- BackendAssets: An embed.FS field that is not a command line argument/flag. It is excluded from the parsed CLI and is used to store backend assets.  
  
