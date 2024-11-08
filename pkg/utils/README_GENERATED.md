## Package: utils

### Imports:

- encoding/base64
- fmt
- io
- net/http
- strings
- time

### External Data, Input Sources:

- The function `GetContentURIAsBase64` takes a string as input, which can be either a URL or a string containing base64 encoded data.

### Summary:

#### Function `GetContentURIAsBase64`:

This function takes a string as input and checks if it's a URL. If it is, it downloads the content from the URL, reads it into memory, encodes it in base64, and returns the base64 string. If the input string is not a URL, it checks if it's prefixed with "data:image/...;base64," and if so, it removes the prefix and returns the remaining string. Otherwise, it returns an error.

The function uses a custom `http.Client` with a timeout of 30 seconds for downloading the content from the URL. It also uses the `base64.StdEncoding` package to encode the downloaded content in base64.



pkg/utils/config.go
## Package: utils

### Imports:

- encoding/json
- os
- path/filepath
- github.com/rs/zerolog/log

### External Data, Input Sources:

- filePath: Path to the directory where the configuration files are stored.
- fileName: Name of the configuration file.
- obj: Any type of object that needs to be saved or loaded as JSON.

### SaveConfig Function:

This function takes the filePath, fileName, and an object as input. It first marshals the object into a JSON string using json.MarshalIndent. If there is an error during marshaling, it logs an error message.

Next, it constructs the absolute path to the configuration file by joining the filePath and fileName using filepath.Join. It then writes the JSON string to the file using os.WriteFile with the file permissions set to 0600. If there is an error during writing, it logs an error message with the filepath.

### LoadConfig Function:

This function takes the filePath, fileName, and an interface as input. It first constructs the absolute path to the configuration file using filepath.Join.

Then, it checks if the file exists using os.Stat. If the file does not exist, it logs a debug message and returns. If the file exists, it reads the file contents using os.ReadFile. If there is an error during reading, it logs an error message with the filepath.

If the file is read successfully, it unmarshals the JSON data into the provided interface using json.Unmarshal. If there is an error during unmarshalling, it logs an error message with the filepath.



pkg/utils/ffmpeg.go
## Package: utils

### Imports:

- fmt
- os
- os/exec
- strings

### External Data, Input Sources:

- src: Input audio file path
- dst: Output audio file path
- format: Desired output audio format (e.g., "opus", "mp3", "aac", "flac")

### Code Summary:

#### ffmpegCommand:

This function executes an ffmpeg command with the given arguments and returns the output and any errors encountered. It sets the environment variables to the current system's environment and uses the CombinedOutput method to capture both the standard output and standard error of the command.

#### AudioToWav:

This function converts an audio file to a WAV format for transcription. It uses the ffmpegCommand function to execute a command that converts the input audio file to a WAV file with specific parameters (16kHz sampling rate, 1 channel, and 16-bit signed integer encoding). It returns an error if the conversion fails.

#### AudioConvert:

This function converts a generated WAV file from text-to-speech (TTS) to other output formats. It first determines the file extension based on the desired format and then executes an ffmpeg command to convert the WAV file to the target format. It returns the output file path and any errors encountered during the conversion process.

pkg/utils/hash.go
## Package: utils

### Imports:
- crypto/md5
- fmt

### External Data and Input Sources:
- String input (s)

### MD5 Function:
This function takes a string as input and returns its MD5 hash as a hexadecimal string. It uses the crypto/md5 package to calculate the hash and the fmt package to format the output. The function first converts the input string to a byte slice using []byte(s), then calculates the MD5 hash using md5.Sum() on the byte slice. Finally, it formats the hash as a hexadecimal string using fmt.Sprintf("%x", ...).

pkg/utils/json.go
## Package: utils

### Imports:
- "regexp"

### External Data, Input Sources:
- None

### Code Summary:
#### EscapeNewLines Function:
This function takes a string as input and returns a new string with newline characters replaced by "\\n". It uses regular expressions to achieve this. First, it finds all occurrences of double-quoted strings within the input string. Then, for each double-quoted string, it replaces any newline characters with "\\n". Finally, it returns the modified string.

pkg/utils/logging.go
## Package: utils

### Imports:

- time
- github.com/rs/zerolog/log

### External Data, Input Sources:

- fileName (string)
- current (string)
- total (string)
- percentage (float64)

### Code Summary:

#### ResetDownloadTimers:

This function resets the download timers by setting the lastProgress and startTime variables to the current time.

#### DisplayDownloadFunction:

This function displays the download progress information. It takes the fileName, current, total, and percentage as input. The function first calculates the elapsed time since the last progress update. If the elapsed time is greater than or equal to 5 seconds, it updates the lastProgress variable and calculates the estimated time of arrival (ETA) based on the percentage and elapsed time. Then, it logs the download progress information, including the fileName, current, total, percentage, and ETA.

pkg/utils/path.go
## Package: utils

### Imports:

- fmt
- os
- path/filepath
- strings

### External Data, Input Sources:

- None

### Code Summary:

#### ExistsInPath:

This function checks if a given file or directory exists in a specified path. It takes two arguments: the path and the filename/directory name. It uses os.Stat to check if the file or directory exists and returns true if it does, false otherwise.

#### InTrustedRoot:

This function checks if a given path is within a trusted root directory. It takes two arguments: the path and the trusted root directory. It iteratively traverses the directory structure of the given path, checking if it matches the trusted root directory. If it does, it returns nil, indicating that the path is within the trusted root. Otherwise, it returns an error indicating that the path is outside the trusted root.

#### VerifyPath:

This function verifies that a given path is based in a specified base path. It takes two arguments: the path and the base path. It first cleans the path using filepath.Clean and filepath.Join to ensure that it is a valid path. Then, it calls the InTrustedRoot function to check if the cleaned path is within the trusted root of the base path. If it is, it returns nil, indicating that the path is valid. Otherwise, it returns an error indicating that the path is not valid.

#### SanitizeFileName:

This function sanitizes a given filename by removing any potentially problematic characters. It takes one argument: the filename. It first cleans the path using filepath.Clean and filepath.Base to ensure that it only contains the final element of the path, not any directory path. Then, it replaces any remaining tricky characters that might have survived cleaning using strings.Replace. Finally, it returns the sanitized filename.

#### GenerateUniqueFileName:

This function generates a unique filename by appending a counter to the base name and extension. It takes three arguments: the directory, the base name, and the extension. It starts with a counter of 1 and creates a filename by combining the base name, counter, and extension. It then checks if the generated filename already exists in the directory. If it does, it increments the counter and creates a new filename. This process continues until a unique filename is found, which is then returned.

pkg/utils/strings.go
## Package: utils

### Imports:
- math/rand
- time

### External Data, Input Sources:
- None

### Code Summary:
#### RandString Function:
This function generates a random string of a given length. It first initializes a slice of runes with the lowercase and uppercase letters of the English alphabet. Then, it seeds the random number generator using the current time in nanoseconds. Finally, it iterates through the slice and fills it with random runes from the letterRunes slice, and returns the resulting string.

#### Unique Function:
This function takes a slice of strings as input and returns a new slice containing only the unique elements from the input slice. It uses a map to keep track of the unique elements encountered so far. For each element in the input slice, it checks if the element is already present in the map. If not, it adds the element to the map and appends it to the result slice. Finally, it returns the result slice containing only the unique elements.

pkg/utils/untar.go
## Package: utils

### Imports:

- fmt
- os
- github.com/mholt/archiver/v3

### External Data, Input Sources:

- The package uses the `archiver` package to handle archive files.

### Code Summary:

#### IsArchive Function:

This function takes a file path as input and returns a boolean value indicating whether the file is an archive or not. It uses the `archiver.ByExtension` function to determine the archive format based on the file extension. If the format is recognized as an archive, it returns true; otherwise, it returns false.

#### ExtractArchive Function:

This function takes two string arguments: the archive file path and the destination directory path. It first determines the archive format using `archiver.ByExtension` and then extracts the contents of the archive to the specified destination directory.

The function first checks if the archive format is supported by the `archiver` package. If not, it returns an error. Then, it creates a new `archiver.Tar` object with specific settings for handling the archive. The function then iterates through the archive files and extracts them to the destination directory. If any errors occur during the extraction process, the function returns an error. Otherwise, it returns nil, indicating successful extraction.

