# pkg/assets/extract.go  
## Package: assets  
  
### Imports:  
  
```  
embed  
fmt  
io/fs  
os  
path/filepath  
github.com/mudler/LocalAI/pkg/library  
```  
  
### External Data, Input Sources:  
  
1. `embed.FS`: Embedded file system containing the assets to be extracted.  
2. `extractDir`: String representing the target directory where the assets will be extracted.  
  
### Code Summary:  
  
#### `ResolvePath` function:  
  
This function takes a base directory and a list of paths as input and returns a resolved path by joining the base directory, "backend-assets", and the provided paths.  
  
#### `ExtractFiles` function:  
  
This function takes an embedded file system (`content`) and an extraction directory (`extractDir`) as input. It first creates the target directory if it doesn't exist. Then, it walks through the embedded file system and extracts each file to the target directory, reconstructing the directory structure as needed.  
  
After extracting all files, the function calls `library.LoadExtractedLibs` to set the `LD_LIBRARY_PATH` environment variable to include the extracted libraries, allowing the system to find and load them.  
  
Finally, the function returns an error if any issues occur during the extraction process.  
  
  
  
# pkg/assets/list.go  
## Package: assets  
  
### Imports:  
  
- embed  
- io/fs  
- github.com/rs/zerolog/log  
  
### External Data, Input Sources:  
  
- embed.FS: An embedded filesystem that contains the assets.  
  
### Summary:  
  
#### ListFiles function:  
  
This function takes an embedded filesystem (embed.FS) as input and returns a list of files within the filesystem. It uses the fs.WalkDir function to iterate through the filesystem and collect the paths of all files. The function first checks if the current entry is a directory. If it is, it skips the directory and continues to the next entry. If the current entry is a file, the function appends the file path to the `files` slice. Finally, the function returns the list of files.  
  
