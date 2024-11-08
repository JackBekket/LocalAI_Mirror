# pkg/stablediffusion/generate.go  
## Package: stablediffusion  
  
### Imports:  
- stableDiffusion "github.com/mudler/go-stable-diffusion"  
  
### External Data, Input Sources:  
- height, width, mode, step, seed: integers  
- positive_prompt, negative_prompt: strings  
- dst: string  
- asset_dir: string  
  
### GenerateImage Function:  
This function generates an image using the Stable Diffusion model. It takes several parameters, including the desired height and width of the image, the generation mode, the number of steps to take, a random seed, positive and negative prompts, the destination path for the generated image, and the directory containing any assets to be used.  
  
If the desired height or width is greater than 512, the function calls the `GenerateImageUpscaled` function from the `stableDiffusion` package to generate the image at a higher resolution. Otherwise, it calls the `GenerateImage` function from the same package to generate the image at the specified resolution.  
  
The `GenerateImageUpscaled` function takes the same parameters as the `GenerateImage` function, but it also takes an additional parameter for the upscaling method. The `GenerateImage` function takes the same parameters as the `GenerateImageUpscaled` function, but it does not take an upscaling method parameter.  
  
# pkg/stablediffusion/generate_unsupported.go  
Package: stablediffusion  
  
Imports:  
- fmt  
  
External data, input sources:  
- height: int  
- width: int  
- mode: string  
- step: int  
- seed: int  
- positive_prompt: string  
- negative_prompt: string  
- dst: string  
- asset_dir: string  
  
### GenerateImage function  
The GenerateImage function is responsible for generating an image based on the provided parameters. It takes the following arguments: height, width, mode, step, seed, positive_prompt, negative_prompt, dst, and asset_dir. The function returns an error if the stablediffusion tag is not present in the build.  
  
# pkg/stablediffusion/stablediffusion.go  
package stablediffusion  
  
imports:  
- os  
  
external data, input sources:  
- assetDir: string  
  
### New function  
The New function takes an asset directory as input and returns a new StableDiffusion instance. It first checks if the asset directory exists using os.Stat. If the directory exists, it creates a new StableDiffusion instance with the provided asset directory and returns it. Otherwise, it returns an error.  
  
### GenerateImage function  
The GenerateImage function takes several parameters: height, width, mode, step, seed, positive_prompt, negative_prompt, dst, and assetDir. It calls the GenerateImage function with these parameters and the asset directory from the StableDiffusion instance. The function returns an error if there is an issue with the image generation process.  
  
  
  
