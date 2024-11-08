# tinydream

### Imports
```
fmt
path/filepath
tinyDream "github.com/M0Rf30/go-tiny-dream"
```

### External Data, Input Sources
- `height`: integer
- `width`: integer
- `step`: integer
- `seed`: integer
- `positive_prompt`: string
- `negative_prompt`: string
- `dst`: string
- `asset_dir`: string

### GenerateImage Function
The `GenerateImage` function takes several parameters, including height, width, step, seed, positive_prompt, negative_prompt, dst, and asset_dir. It first checks if the height or width is greater than 512. If so, it calls the `tinyDream.GenerateImage` function with a value of 1 for the first parameter. Otherwise, it calls the same function with a value of 0 for the first parameter. The function returns an error if there is an issue with the image generation process.

## tinydream

### Imports:
- os

### External Data, Input Sources:
- assetDir: A string representing the directory containing the assets.

### Summary:
#### TinyDream struct:
The TinyDream struct represents a tiny dream generator. It has a single field, assetDir, which stores the path to the directory containing the assets.

#### New function:
The New function creates a new TinyDream instance. It takes the asset directory as input and checks if the directory exists. If the directory exists, it returns a new TinyDream instance with the provided asset directory. Otherwise, it returns an error.

#### GenerateImage function:
The GenerateImage function generates an image based on the provided parameters. It takes the following parameters:
- height: The height of the image.
- width: The width of the image.
- step: The step size for the image generation.
- seed: The random seed for the image generation.
- positive_prompt: A string containing the positive prompt for the image generation.
- negative_prompt: A string containing the negative prompt for the image generation.
- dst: The destination path for the generated image.

The function calls the GenerateImage function from another package, passing the provided parameters and the asset directory from the TinyDream instance.

```
tinydream/
├── generate.go
├── generate_unsupported.go
└── tinydream.go
```

