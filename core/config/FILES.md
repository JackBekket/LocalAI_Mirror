# core/config/application_config.go  
## config package  
  
This package contains the configuration for the application. It defines the ApplicationConfig struct, which holds all the necessary settings for the application to run. The package also provides various helper functions to set and modify the configuration options.  
  
### Imports  
  
* `context`  
* `embed`  
* `encoding/json`  
* `regexp`  
* `time`  
* `github.com/mudler/LocalAI/pkg/xsysinfo`  
* `github.com/rs/zerolog/log`  
  
### External Data, Input Sources  
  
* The package uses the `embed` package to embed assets, such as backend assets and galleries.  
* It also uses the `regexp` package to define regular expressions for exempting endpoints from the API key requirement.  
  
### Major Code Parts  
  
1. ApplicationConfig struct: This struct holds all the configuration options for the application. It includes settings for the context, file paths, upload limits, threads, context size, F16 support, debug mode, directories for images, audio, uploads, and configurations, as well as options for CORS, CSRF, preloading models, API keys, and more.  
  
2. AppOption type: This type represents an option for configuring the ApplicationConfig struct. It is used to set individual configuration options.  
  
3. NewApplicationConfig function: This function creates a new instance of the ApplicationConfig struct with default values and allows for customization using AppOption functions.  
  
4. Helper functions: The package provides various helper functions to set specific configuration options, such as WithModelsURL, WithModelPath, WithCors, WithP2PNetworkID, WithCsrf, WithP2PToken, WithModelLibraryURL, WithLibPath, and more.  
  
5. ConfigLoaderOption type: This type represents an option for configuring the ConfigLoader, which is responsible for loading configuration files for models. The ApplicationConfig struct's ToConfigLoaderOptions function returns a slice of ConfigLoaderOption, which can be used to set default values for models that don't specify their own configurations.  
  
6. WithMetrics function: This function allows for setting a metrics object for the application.  
  
7. WithHttpGetExemptedEndpoints function: This function allows for setting a list of regular expressions that exempt certain endpoints from the API key requirement.  
  
8. DisableMetricsEndpoint AppOption: This option disables the metrics endpoint.  
  
By providing these configuration options and helper functions, the package allows for flexible and customizable settings for the application, ensuring that it can be tailored to various use cases and environments.  
  
# core/config/backend_config.go  
```  
package config  
  
import (  
	"os"  
	"regexp"  
	"slices"  
	"strings"  
  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/downloader"  
	"github.com/mudler/LocalAI/pkg/functions"  
	"gopkg.in/yaml.v3"  
)  
  
const (  
	RAND_SEED = -1  
)  
  
type TTSConfig struct {  
	Voice string `yaml:"voice"`  
}  
  
type BackendConfig struct {  
	schema.PredictionOptions `yaml:"parameters"`  
	Name                     string `yaml:"name"`  
  
	F16                 *bool                  `yaml:"f16"`  
	Threads             *int                   `yaml:"threads"`  
	Debug               *bool                  `yaml:"debug"`  
	Roles               map[string]string      `yaml:"roles"`  
	Embeddings          *bool                  `yaml:"embeddings"`  
	Backend             string                 `yaml:"backend"`  
	TemplateConfig      TemplateConfig         `yaml:"template"`  
	KnownUsecaseStrings []string               `yaml:"known_usecases"`  
	KnownUsecases       *BackendConfigUsecases `yaml:"-"`  
  
	PromptStrings, InputStrings                []string               `yaml:"-"`  
	InputToken                                 [][]int                `yaml:"-"`  
	functionCallString, functionCallNameString string                 `yaml:"-"`  
	ResponseFormat                             string                 `yaml:"-"`  
	ResponseFormatMap                          map[string]interface{} `yaml:"-"`  
  
	FunctionsConfig functions.FunctionsConfig `yaml:"function"`  
  
	FeatureFlag FeatureFlag `yaml:"feature_flags"` // Feature Flag registry. We move fast, and features may break on a per model/backend basis.  
	// LLM configs (GPT4ALL, Llama.cpp, ...)  
	LLMConfig `yaml:",inline"`  
  
	// AutoGPTQ specifics  
	AutoGPTQ AutoGPTQ `yaml:"autogptq"`  
  
	// Diffusers  
	Diffusers Diffusers `yaml:"diffusers"`  
	Step      int       `yaml:"step"`  
  
	// GRPC Options  
	GRPC GRPC `yaml:"grpc"`  
  
	// TTS specifics  
	TTSConfig `yaml:"tts"`  
  
	// CUDA  
	CUDA bool `yaml:"cuda"`  
  
	DownloadFiles []File `yaml:"download_files"`  
  
	Description string `yaml:"description"`  
	Usage       string `yaml:"usage"`  
}  
  
type File struct {  
	Filename string         `yaml:"filename" json:"filename"`  
	SHA256   string         `yaml:"sha256" json:"sha256"`  
	URI      downloader.URI `yaml:"uri" json:"uri"`  
}  
  
type VallE struct {  
	AudioPath string `yaml:"audio_path"`  
}  
  
type FeatureFlag map[string]*bool  
  
func (ff FeatureFlag) Enabled(s string) bool {  
	v, exist := ff[s]  
	return exist && v != nil && *v  
}  
  
type GRPC struct {  
	Attempts          int `yaml:"attempts"`  
	AttemptsSleepTime int `yaml:"attempts_sleep_time"`  
}  
  
type Diffusers struct {  
	CUDA             bool    `yaml:"cuda"`  
	PipelineType     string  `yaml:"pipeline_type"`  
	SchedulerType    string  `yaml:"scheduler_type"`  
	EnableParameters string  `yaml:"enable_parameters"` // A list of comma separated parameters to specify  
	CFGScale         float32 `yaml:"cfg_scale"`         // Classifier-Free Guidance Scale  
	IMG2IMG          bool    `yaml:"img2img"`           // Image to Image Diffuser  
	ClipSkip         int     `yaml:"clip_skip"`         // Skip every N frames  
	ClipModel        string  `yaml:"clip_model"`        // Clip model to use  
	ClipSubFolder    string  `yaml:"clip_subfolder"`    // Subfolder to use for clip model  
	ControlNet       string  `yaml:"control_net"`  
}  
  
// LLMConfig is a struct that holds the configuration that are  
// generic for most of the LLM backends.  
type LLMConfig struct {  
	SystemPrompt    string   `yaml:"system_prompt"`  
	TensorSplit     string   `yaml:"tensor_split"`  
	MainGPU         string   `yaml:"main_gpu"`  
	RMSNormEps      float32  `yaml:"rms_norm_eps"`  
	NGQA            int32    `yaml:"ngqa"`  
	PromptCachePath string   `yaml:"prompt_cache_path"`  
	PromptCacheAll  bool     `yaml:"prompt_cache_all"`  
	PromptCacheRO   bool     `yaml:"prompt_cache_ro"`  
	MirostatETA     *float64 `yaml:"mirostat_eta"`  
	MirostatTAU     *float64 `yaml:"mirostat_tau"`  
	Mirostat        *int     `yaml:"mirostat"`  
	NGPULayers      *int     `yaml:"gpu_layers"`  
	MMap            *bool    `yaml:"mmap"`  
	MMlock          *bool    `yaml:"mmlock"`  
	LowVRAM         *bool    `yaml:"low_vram"`  
	Grammar         string   `yaml:"grammar"`  
	StopWords       []string `yaml:"stopwords"`  
	Cutstrings      []string `yaml:"cutstrings"`  
	ExtractRegex    []string `yaml:"extract_regex"`  
	TrimSpace       []string `yaml:"trimspace"`  
	TrimSuffix      []string `yaml:"trimsuffix"`  
  
	ContextSize          *int      `yaml:"context_size"`  
	NUMA                 bool      `yaml:"numa"`  
	LoraAdapter          string    `yaml:"lora_adapter"`  
	LoraBase             string    `yaml:"lora_base"`  
	LoraAdapters         []string  `yaml:"lora_adapters"`  
	LoraScales           []float32 `yaml:"lora_scales"`  
	LoraScale            float32   `yaml:"lora_scale"`  
	NoMulMatQ            bool      `yaml:"no_mulmatq"`  
	DraftModel           string    `yaml:"draft_model"`  
	NDraft               int32     `yaml:"n_draft"`  
	Quantization         string    `yaml:"quantization"`  
	LoadFormat           string    `yaml:"load_format"`  
	GPUMemoryUtilization float32   `yaml:"gpu_memory_utilization"` // vLLM  
	TrustRemoteCode      bool      `yaml:"trust_remote_code"`      // vLLM  
	EnforceEager         bool      `yaml:"enforce_eager"`          // vLLM  
	SwapSpace            int       `yaml:"swap_space"`             // vLLM  
	MaxModelLen          int       `yaml:"max_model_len"`          // vLLM  
	TensorParallelSize   int       `yaml:"tensor_parallel_size"`   // vLLM  
	MMProj               string    `yaml:"mmproj"`  
  
	FlashAttention bool `yaml:"flash_attention"`  
	NoKVOffloading bool `yaml:"no_kv_offloading"`  
  
	RopeScaling string `yaml:"rope_scaling"`  
	ModelType   string `yaml:"type"`  
  
	YarnExtFactor  float32 `yaml:"yarn_ext_factor"`  
	YarnAttnFactor float32 `yaml:"yarn_attn_factor"`  
	YarnBetaFast   float32 `yaml:"yarn_beta_fast"`  
	YarnBetaSlow   float32 `yaml:"yarn_beta_slow"`  
}  
  
// AutoGPTQ is a struct that holds the configuration specific to the AutoGPTQ backend  
type AutoGPTQ struct {  
	ModelBaseName    string `yaml:"model_base_name"`  
	Device           string `yaml:"device"`  
	Triton           bool   `yaml:"triton"`  
	UseFastTokenizer bool   `yaml:"use_fast_tokenizer"`  
}  
  
// TemplateConfig is a struct that holds the configuration of the templating system  
type TemplateConfig struct {  
	// Chat is the template used in the chat completion endpoint  
	Chat string `yaml:"chat"`  
  
	// ChatMessage is the template used for chat messages  
	ChatMessage string `yaml:"chat_message"`  
  
	// Completion is the template used for completion requests  
	Completion string `yaml:"completion"`  
  
	// Edit is the template used for edit completion requests  
	Edit string `yaml:"edit"`  
  
	// Functions is the template used when tools are present in the client requests  
	// Note: this is mostly consumed for backends such as vllm and transformers  
	// that can use the tokenizers specified in the JSON config files of the models  
	Functions string `yaml:"function"`  
  
	// UseTokenizerTemplate is a flag that indicates if the tokenizer template should be used.  
	// Note: this is mostly consumed for backends such as vllm and transformers  
	// that can use the tokenizers specified in the JSON config files of the models  
	UseTokenizerTemplate bool `yaml:"use_tokenizer_template"`  
  
	// JoinChatMessagesByCharacter is a string that will be used to join chat messages together.  
	// It defaults to \n  
	JoinChatMessagesByCharacter *string `yaml:"join_chat_messages_by_character"`  
  
	Multimodal string `yaml:"multimodal"`  
}  
  
func (c *BackendConfig) UnmarshalYAML(value *yaml.Node) error {  
	type BCAlias BackendConfig  
	var aux BCAlias  
	if err := value.Decode(&aux); err != nil {  
		return err  
	}  
	*c = BackendConfig(aux)  
	c.KnownUsecases = GetUsecasesFromYAML(c.KnownUsecaseStrings)  
	return nil  
}  
  
func (c *BackendConfig) SetFunctionCallString(s string) {  
	c.functionCallString = s  
}  
  
func (c *BackendConfig) SetFunctionCallNameString(s string) {  
	c.functionCallNameString = s  
}  
  
func (c *BackendConfig) ShouldUseFunctions() bool {  
	return ((c.functionCallString != "none" || c.functionCallString == "") || c.ShouldCallSpecificFunction())  
}  
  
func (c *BackendConfig) ShouldCallSpecificFunction() bool {  
	return len(c.functionCallNameString) > 0 && c.functionCallNameString != "none" && c.functionCallNameString != "auto"  
}  
  
// MMProjFileName returns the filename of the MMProj file  
// If the MMProj is a URL, it will return the MD5 of the URL which is the filename  
func (c *BackendConfig) MMProjFileName() string {  
	uri := downloader.URI(c.MMProj)  
	if uri.LooksLikeURL() {  
		f, _ := uri.FilenameFromUrl()  
		return f  
	}  
  
	return c.MMProj  
}  
  
func (c *BackendConfig) IsMMProjURL() bool {  
	uri := downloader.URI(c.MMProj)  
	return uri.LooksLikeURL()  
}  
  
func (c *BackendConfig) IsModelURL() bool {  
	uri := downloader.URI(c.Model)  
	return uri.LooksLikeURL()  
}  
  
// ModelFileName returns the filename of the model  
// If the model is a URL, it will return the MD5 of the URL which is the filename  
func (c *BackendConfig) ModelFileName() string {  
	uri := downloader.URI(c.Model)  
	if uri.LooksLikeURL() {  
		f, _ := uri.FilenameFromUrl()  
		return f  
	}  
  
	return c.Model  
}  
  
func (c *BackendConfig) FunctionToCall() string {  
	if c.functionCallNameString != "" &&  
		c.functionCallNameString != "none" && c.functionCallNameString != "auto" {  
		return c.functionCallNameString  
	}  
  
	return c.functionCallString  
}  
  
func (cfg *BackendConfig) SetDefaults(opts ...ConfigLoaderOption) {  
	lo := &LoadOptions{}  
	lo.Apply(opts...)  
  
	ctx := lo.ctxSize  
	threads := lo.threads  
	f16 := lo.f16  
	debug := lo.debug  
	// https://github.com/ggerganov/llama.cpp/blob/75cd4c77292034ecec587ecb401366f57338f7c0/common/sampling.h#L22  
	defaultTopP := 0.95  
	defaultTopK := 40  
	defaultTemp := 0.9  
	defaultMirostat := 2  
	defaultMirostatTAU := 5.0  
	defaultMirostatETA := 0.1  
	defaultTypicalP :=   
  
# core/config/backend_config_filter.go  
## Package: config  
  
### Imports:  
- "regexp"  
  
### External Data, Input Sources:  
- BackendConfigUsecases (enum)  
  
### Functions:  
#### BackendConfigFilterFn:  
This type represents a function that takes a string and a pointer to a BackendConfig struct as input and returns a boolean value. It is used to filter BackendConfig structs based on certain criteria.  
  
#### NoFilterFn:  
This function is a default implementation of BackendConfigFilterFn that always returns true, effectively filtering no BackendConfig structs.  
  
#### BuildNameFilterFn:  
This function takes a string as input and returns a BackendConfigFilterFn that filters BackendConfig structs based on the provided string. It first checks if the input string is empty. If it is, it returns NoFilterFn. Otherwise, it compiles a regular expression from the input string and returns a new BackendConfigFilterFn that uses this regular expression to filter BackendConfig structs.  
  
#### BuildUsecaseFilterFn:  
This function takes a BackendConfigUsecases value as input and returns a BackendConfigFilterFn that filters BackendConfig structs based on the provided usecases. If the input value is FLAG_ANY, it returns NoFilterFn. Otherwise, it returns a new BackendConfigFilterFn that checks if the BackendConfig struct has the specified usecases and returns true if it does, false otherwise.  
  
  
  
# core/config/backend_config_loader.go  
```  
package config  
  
// This file contains the code for loading and managing backend configurations for models.  
  
// Imports  
import (  
	"errors"  
	"fmt"  
	"io/fs"  
	"os"  
	"path/filepath"  
	"sort"  
	"strings"  
	"sync"  
  
	"github.com/charmbracelet/glamour"  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/downloader"  
	"github.com/mudler/LocalAI/pkg/utils"  
	"github.com/rs/zerolog/log"  
	"gopkg.in/yaml.v3"  
)  
  
// BackendConfigLoader is a struct that loads and manages backend configurations for models.  
type BackendConfigLoader struct {  
	configs   map[string]BackendConfig  
	modelPath string  
	sync.Mutex  
}  
  
// NewBackendConfigLoader creates a new BackendConfigLoader instance.  
func NewBackendConfigLoader(modelPath string) *BackendConfigLoader {  
	return &BackendConfigLoader{  
		configs:   make(map[string]BackendConfig),  
		modelPath: modelPath,  
	}  
}  
  
// LoadOptionDebug is a ConfigLoaderOption that enables debug mode.  
func LoadOptionDebug(debug bool) ConfigLoaderOption {  
	return func(o *LoadOptions) {  
		o.debug = debug  
	}  
}  
  
// LoadOptionThreads is a ConfigLoaderOption that sets the number of threads to use.  
func LoadOptionThreads(threads int) ConfigLoaderOption {  
	return func(o *LoadOptions) {  
		o.threads = threads  
	}  
}  
  
// LoadOptionContextSize is a ConfigLoaderOption that sets the context size.  
func LoadOptionContextSize(ctxSize int) ConfigLoaderOption {  
	return func(o *LoadOptions) {  
		o.ctxSize = ctxSize  
	}  
}  
  
// ModelPath is a ConfigLoaderOption that sets the model path.  
func ModelPath(modelPath string) ConfigLoaderOption {  
	return func(o *LoadOptions) {  
		o.modelPath = modelPath  
	}  
}  
  
// LoadOptionF16 is a ConfigLoaderOption that enables or disables F16 precision.  
func LoadOptionF16(f16 bool) ConfigLoaderOption {  
	return func(o *LoadOptions) {  
		o.f16 = f16  
	}  
}  
  
// ConfigLoaderOption is a type alias for a function that takes a LoadOptions pointer and modifies it.  
type ConfigLoaderOption func(*LoadOptions)  
  
// LoadOptions is a struct that holds the options for loading configurations.  
type LoadOptions struct {  
	modelPath        string  
	debug            bool  
	threads, ctxSize int  
	f16              bool  
}  
  
// Apply applies the given ConfigLoaderOptions to the LoadOptions struct.  
func (lo *LoadOptions) Apply(options ...ConfigLoaderOption) {  
	for _, l := range options {  
		l(lo)  
	}  
}  
  
// readMultipleBackendConfigsFromFile reads multiple backend configurations from a YAML file.  
func readMultipleBackendConfigsFromFile(file string, opts ...ConfigLoaderOption) ([]*BackendConfig, error) {  
	c := &[]*BackendConfig{}  
	f, err := os.ReadFile(file)  
	if err != nil {  
		return nil, fmt.Errorf("cannot read config file: %w", err)  
	}  
	if err := yaml.Unmarshal(f, c); err != nil {  
		return nil, fmt.Errorf("cannot unmarshal config file: %w", err)  
	}  
  
	for _, cc := range *c {  
		cc.SetDefaults(opts...)  
	}  
  
	return *c, nil  
}  
  
// readBackendConfigFromFile reads a single backend configuration from a YAML file.  
func readBackendConfigFromFile(file string, opts ...ConfigLoaderOption) (*BackendConfig, error) {  
	lo := &LoadOptions{}  
	lo.Apply(opts...)  
  
	c := &BackendConfig{}  
	f, err := os.ReadFile(file)  
	if err != nil {  
		return nil, fmt.Errorf("cannot read config file: %w", err)  
	}  
	if err := yaml.Unmarshal(f, c); err != nil {  
		return nil, fmt.Errorf("cannot unmarshal config file: %w", err)  
	}  
  
	c.SetDefaults(opts...)  
	return c, nil  
}  
  
// LoadBackendConfigFileByName loads a backend configuration file for a model.  
func (bcl *BackendConfigLoader) LoadBackendConfigFileByName(modelName, modelPath string, opts ...ConfigLoaderOption) (*BackendConfig, error) {  
	cfg := &BackendConfig{  
		PredictionOptions: schema.PredictionOptions{  
			Model: modelName,  
		},  
	}  
  
	cfgExisting, exists := bcl.GetBackendConfig(modelName)  
	if exists {  
		cfg = &cfgExisting  
	} else {  
		// Try loading a model config file  
		modelConfig := filepath.Join(modelPath, modelName+".yaml")  
		if _, err := os.Stat(modelConfig); err == nil {  
			if err := bcl.LoadBackendConfig(modelConfig, opts...); err != nil {  
				return nil, fmt.Errorf("failed loading model config (%s) %s", modelConfig, err.Error())  
			}  
			cfgExisting, exists = bcl.GetBackendConfig(modelName)  
			if exists {  
				cfg = &cfgExisting  
			}  
		}  
	}  
  
	cfg.SetDefaults(opts...)  
  
	return cfg, nil  
}  
  
// LoadMultipleBackendConfigsSingleFile loads multiple backend configurations from a single YAML file.  
func (bcl *BackendConfigLoader) LoadMultipleBackendConfigsSingleFile(file string, opts ...ConfigLoaderOption) error {  
	bcl.Lock()  
	defer bcl.Unlock()  
	c, err := readMultipleBackendConfigsFromFile(file, opts...)  
	if err != nil {  
		return fmt.Errorf("cannot load config file: %w", err)  
	}  
  
	for _, cc := range c {  
		if cc.Validate() {  
			bcl.configs[cc.Name] = *cc  
		}  
	}  
	return nil  
}  
  
// LoadBackendConfig loads a backend configuration from a YAML file.  
func (bcl *BackendConfigLoader) LoadBackendConfig(file string, opts ...ConfigLoaderOption) error {  
	bcl.Lock()  
	defer bcl.Unlock()  
	c, err := readBackendConfigFromFile(file, opts...)  
	if err != nil {  
		return fmt.Errorf("cannot read config file: %w", err)  
	}  
  
	if c.Validate() {  
		bcl.configs[c.Name] = *c  
	} else {  
		return fmt.Errorf("config is not valid")  
	}  
  
	return nil  
}  
  
// GetBackendConfig returns a backend configuration by name.  
func (bcl *BackendConfigLoader) GetBackendConfig(m string) (BackendConfig, bool) {  
	bcl.Lock()  
	defer bcl.Unlock()  
	v, exists := bcl.configs[m]  
	return v, exists  
}  
  
// GetAllBackendConfigs returns all backend configurations.  
func (bcl *BackendConfigLoader) GetAllBackendConfigs() []BackendConfig {  
	bcl.Lock()  
	defer bcl.Unlock()  
	var res []BackendConfig  
	for _, v := range bcl.configs {  
		res = append(res, v)  
	}  
  
	sort.SliceStable(res, func(i, j int) bool {  
		return res[i].Name < res[j].Name  
	})  
  
	return res  
}  
  
// GetBackendConfigsByFilter returns backend configurations that match the given filter.  
func (bcl *BackendConfigLoader) GetBackendConfigsByFilter(filter BackendConfigFilterFn) []BackendConfig {  
	bcl.Lock()  
	defer bcl.Unlock()  
	var res []BackendConfig  
  
	if filter == nil {  
		filter = NoFilterFn  
	}  
  
	for n, v := range bcl.configs {  
		if filter(n, &v) {  
			res = append(res, v)  
		}  
	}  
  
	return res  
}  
  
// RemoveBackendConfig removes a backend configuration by name.  
func (bcl *BackendConfigLoader) RemoveBackendConfig(m string) {  
	bcl.Lock()  
	defer bcl.Unlock()  
	delete(bcl.configs, m)  
}  
  
// Preload prepares models if they are not local but URL or huggingface repositories.  
func (bcl *BackendConfigLoader) Preload(modelPath string) error {  
	bcl.Lock()  
	defer bcl.Unlock()  
  
	status := func(fileName, current, total string, percent float64) {  
		utils.DisplayDownloadFunction(fileName, current, total, percent)  
	}  
  
	log.Info().Msgf("Preloading models from %s", modelPath)  
  
	renderMode := "dark"  
	if os.Getenv("COLOR") != "" {  
		renderMode = os.Getenv("COLOR")  
	}  
  
	glamText := func(t string) {  
		out, err := glamour.Render(t, renderMode)  
		if err == nil && os.Getenv("NO_COLOR") == "" {  
			fmt.Println(out)  
		} else {  
			fmt.Println(t)  
		}  
	}  
  
	for i, config := range bcl.configs {  
		// Download files and verify their SHA  
		for i, file := range config.DownloadFiles {  
			log.Debug().Msgf("Checking %q exists and matches SHA", file.Filename)  
  
			if err := utils.VerifyPath(file.Filename, modelPath); err != nil {  
				return err  
			}  
			// Create file path  
			filePath := filepath.Join(modelPath, file.Filename)  
  
			if err := file.URI.DownloadFile(filePath, file.SHA256, i, len(config.DownloadFiles), status); err != nil {  
				return err  
			}  
		}  
  
		// If the model is an URL, expand it, and download the file  
		if config.IsModelURL() {  
			modelFileName := config.ModelFileName()  
			uri := downloader.URI(config.Model)  
			// check if file exists  
			if _, err := os.Stat(filepath.Join(modelPath, modelFileName)); errors.Is(err, os.ErrNotExist) {  
				err := uri.DownloadFile(filepath.Join(modelPath, modelFileName), "", 0, 0, status)  
				if err != nil {  
					return err  
				}  
			}  
  
			cc := bcl.configs[i]  
			c := &cc  
			c.PredictionOptions.Model = modelFileName  
			bcl.configs[i] = *c  
		}  
  
		if config.IsMMProjURL() {  
			modelFileName := config.MMProjFileName()  
			uri := downloader.URI(config.MMProj)  
			// check if file exists  
			if _, err := os.Stat(filepath.Join(modelPath, modelFileName)); errors.Is(err, os.ErrNotExist) {  
				err := uri.DownloadFile(filepath.Join(modelPath, modelFileName), "", 0, 0, status)  
				if err != nil {  
					return err  
				}  
			}  
  
			cc := bcl.configs[i]  
			c := &cc  
			c.MMProj = modelFileName  
			bcl.configs[i] = *c  
		}  
  
		if bcl.configs[i].Name != "" {  
			glamText(fmt.Sprintf("**Model name**: _%s_", bcl.configs[i].Name))  
		}  
		if bcl.configs[i].Description != "" {  
			//glamText("**Description**")  
			glamText(bcl.configs[i].Description)  
		}  
		if bcl.configs[i].Usage != "" {  
			//glamText("**Usage**")  
			glamText(bcl.configs[i].Usage)  
		}  
	}  
	return nil  
}  
  
// LoadBackendConfigsFromPath reads all the configurations of the models from a path (non-recursive).  
func (bcl *BackendConfigLoader) LoadBackendConfigsFromPath(path string, opts ...ConfigLoaderOption) error {  
	bcl.Lock()  
	defer bcl.Unlock()  
	entries, err := os.ReadDir(path)  
	if err != nil {  
		return fmt.Errorf("cannot read directory '%s': %w", path, err)  
	}  
	files := make([]fs.FileInfo, 0, len(entries))  
	for _, entry := range entries {  
		info, err := entry.Info()  
		if err != nil {  
			return err  
		}  
		files = append(files, info)  
	}  
	for _, file := range files {  
		// Skip templates, YAML and .keep files  
		if !strings.Contains(file.Name(), ".yaml") && !strings.Contains(file.Name(), ".yml") ||  
			strings.HasPrefix(file.Name(), ".") {  
			continue  
		}  
		c, err := readBackendConfigFromFile(filepath.Join(path, file.Name()), opts...)  
		if err != nil {  
			log.Error().Err(err).Msgf("cannot read config file: %s", file.Name())  
			continue  
		}  
		if c.Validate() {  
			bcl.configs[c.Name] = *c  
		} else {  
			log.Error().Err(err).Msgf("config is not valid")  
		}  
	}  
  
	return nil  
}  
```  
  
# core/config/backend_config_test.go  
## Package: config  
  
### Imports:  
  
- io  
- net/http  
- os  
- github.com/onsi/ginkgo/v2  
- github.com/onsi/gomega  
  
### External Data, Input Sources:  
  
- config.yaml file (used for testing)  
- https://raw.githubusercontent.com/mudler/LocalAI/master/embedded/models/hermes-2-pro-mistral.yaml (used for testing)  
  
### Summary of Code:  
  
#### Test cases for config related functions:  
  
This section contains test cases for config related functions, specifically focusing on reading and validating configuration files.  
  
#### Test Read configuration functions:  
  
This subsection contains two test cases, both of which involve creating a temporary file named "config.yaml" and writing configuration data to it. The first test case demonstrates the validation process for a configuration file with incorrect data, while the second test case demonstrates the validation process for a configuration file with correct data.  
  
#### Properly handles backend usecase matching:  
  
This section tests the functionality of the `HasUsecases` method, which determines whether a backend configuration has specific use cases. It covers various scenarios with different backend configurations and use cases, ensuring that the method correctly identifies the presence or absence of specific use cases.  
  
# core/config/config_test.go  
## Package: config  
  
### Imports:  
  
- os  
- github.com/onsi/ginkgo/v2  
- github.com/onsi/gomega  
  
### External Data, Input Sources:  
  
- CONFIG_FILE: Environment variable containing the path to the configuration file.  
- MODELS_PATH: Environment variable containing the path to the models directory.  
  
### Test cases for config related functions:  
  
#### Test Read configuration functions:  
  
- Test readConfigFile:  
    - Reads multiple backend configurations from the specified configuration file.  
    - Checks if the error is nil and the config is not nil.  
    - Verifies that the config contains two configurations with names "list1" and "list2".  
- Test LoadConfigs:  
    - Creates a BackendConfigLoader instance with the models path.  
    - Loads backend configurations from the models path.  
    - Checks if the error is nil and the loaded configurations are not nil.  
    - Verifies that the loaded configurations include models with names "code-search-ada-code-001", "text-embedding-ada-002", "rwkv_test", and "whisper-1".  
  
  
  
# core/config/gallery.go  
## Package: config  
  
### Imports:  
  
- json  
- yaml  
  
### External Data, Input Sources:  
  
- None  
  
### Summary:  
  
The provided code defines a struct named `Gallery` within the `config` package. The `Gallery` struct has two fields:  
  
1. `URL`: A string field representing the URL of the gallery, with the tag `json:"url"` and `yaml:"url"` for JSON and YAML serialization, respectively.  
  
2. `Name`: A string field representing the name of the gallery, with the tag `json:"name"` and `yaml:"name"` for JSON and YAML serialization, respectively.  
  
# core/config/guesser.go  
  
  
