# core/http/routes/elevenlabs.go  
## Package: routes  
  
### Imports:  
  
- github.com/gofiber/fiber/v2  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/http/endpoints/elevenlabs  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
- Backend configuration (cl)  
- Model loader (ml)  
- Application configuration (appConfig)  
  
### RegisterElevenLabsRoutes Function:  
  
This function registers two endpoints related to ElevenLabs functionality:  
  
1. Text-to-speech endpoint:  
   - URL: /v1/text-to-speech/:voice-id  
   - Method: POST  
   - Handler: elevenlabs.TTSEndpoint(cl, ml, appConfig)  
  
2. Sound generation endpoint:  
   - URL: /v1/sound-generation  
   - Method: POST  
   - Handler: elevenlabs.SoundGenerationEndpoint(cl, ml, appConfig)  
  
Both endpoints utilize the provided configuration, model loader, and application configuration to process requests and generate responses.  
  
  
  
# core/http/routes/explorer.go  
## Package: routes  
  
### Imports:  
- github.com/gofiber/fiber/v2  
- github.com/mudler/LocalAI/core/explorer  
- github.com/mudler/LocalAI/core/http/endpoints/explorer  
  
### External Data, Input Sources:  
- `app *fiber.App`: A Fiber application instance.  
- `db *coreExplorer.Database`: A database instance for the explorer module.  
  
### RegisterExplorerRoutes Function:  
This function registers routes for the explorer module. It takes a Fiber application instance and a database instance as input.  
  
- `app.Get("/", explorer.Dashboard())`: Registers a GET route for the root path, which calls the `Dashboard()` function from the `explorer` package. This likely displays the explorer dashboard.  
- `app.Post("/network/add", explorer.AddNetwork(db))`: Registers a POST route for the "/network/add" path, which calls the `AddNetwork()` function from the `explorer` package, passing the database instance as an argument. This likely handles adding a new network to the explorer.  
- `app.Get("/networks", explorer.ShowNetworks(db))`: Registers a GET route for the "/networks" path, which calls the `ShowNetworks()` function from the `explorer` package, passing the database instance as an argument. This likely displays a list of networks in the explorer.  
  
  
  
# core/http/routes/health.go  
## Package: routes  
  
### Imports:  
- "github.com/gofiber/fiber/v2"  
  
### External Data and Input Sources:  
- None  
  
### Code Summary:  
#### Health Routes:  
This function defines two health check routes for the application. The first route, "/healthz", returns a 200 status code when accessed, indicating that the service is healthy. The second route, "/readyz", also returns a 200 status code, indicating that the service is ready to handle requests. Both routes are implemented using the `ok` function, which simply sends a 200 status code to the client.  
  
# core/http/routes/jina.go  
## Package: routes  
  
### Imports:  
  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/http/endpoints/jina  
- github.com/gofiber/fiber/v2  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
- config.BackendConfigLoader  
- model.ModelLoader  
- config.ApplicationConfig  
  
### RegisterJINARoutes Function:  
  
This function registers a POST endpoint at "/v1/rerank" to mimic the reranking functionality. It takes an instance of the Fiber application, a BackendConfigLoader, a ModelLoader, and an ApplicationConfig as input. The function then calls the JINARerankEndpoint function from the jina package, passing in the provided input parameters. This endpoint is responsible for handling the reranking logic and returning the appropriate response.  
  
# core/http/routes/localai.go  
## Package: routes  
  
### Imports:  
  
- github.com/gofiber/fiber/v2  
- github.com/gofiber/swagger  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/http/endpoints/localai  
- github.com/mudler/LocalAI/core/p2p  
- github.com/mudler/LocalAI/core/services  
- github.com/mudler/LocalAI/internal  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
- config.BackendConfigLoader  
- model.ModelLoader  
- config.ApplicationConfig  
- services.GalleryService  
  
### Summary:  
  
#### LocalAI API Endpoints:  
  
- The code defines a function `RegisterLocalAIRoutes` that registers various API endpoints related to LocalAI functionality.  
- It first registers a default swagger handler for the `/swagger/*` path.  
- If the `DisableGalleryEndpoint` flag in the `ApplicationConfig` is false, it registers endpoints for managing model galleries, including applying, deleting, listing available models, listing galleries, adding and removing galleries, and getting the status of jobs.  
- It also registers an endpoint for text-to-speech (TTS) functionality using the `TTSEndpoint` function.  
  
#### Stores:  
  
- The code registers endpoints for managing stores, including setting, deleting, getting, and finding stores.  
  
#### Metrics:  
  
- If the `DisableMetrics` flag in the `ApplicationConfig` is false, it registers an endpoint for displaying metrics using the `LocalAIMetricsEndpoint` function.  
  
#### Experimental Backend Statistics Module:  
  
- It registers endpoints for monitoring and shutting down the backend using the `BackendMonitorEndpoint` and `BackendShutdownEndpoint` functions.  
  
#### p2p:  
  
- If p2p is enabled, it registers endpoints for showing p2p nodes and tokens using the `ShowP2PNodes` and `ShowP2PToken` functions.  
  
#### Version and System Information:  
  
- It registers endpoints for displaying the version and system information using the `PrintableVersion` and `SystemInformations` functions.  
  
#### Tokenization:  
  
- It registers an endpoint for tokenization using the `TokenizeEndpoint` function.  
  
  
  
# core/http/routes/openai.go  
## Package: routes  
  
### Imports:  
  
- `github.com/gofiber/fiber/v2`  
- `github.com/mudler/LocalAI/core/config`  
- `github.com/mudler/LocalAI/core/http/endpoints/localai`  
- `github.com/mudler/LocalAI/core/http/endpoints/openai`  
- `github.com/mudler/LocalAI/pkg/model`  
  
### External Data, Input Sources:  
  
- `cl`: BackendConfigLoader  
- `ml`: ModelLoader  
- `appConfig`: ApplicationConfig  
  
### RegisterOpenAIRoutes Function:  
  
This function registers OpenAI compatible API endpoints for the given Fiber app. It takes the Fiber app, BackendConfigLoader, ModelLoader, and ApplicationConfig as input.  
  
1. Chat:  
    - `app.Post("/v1/chat/completions", openai.ChatEndpoint(cl, ml, appConfig))`  
    - `app.Post("/chat/completions", openai.ChatEndpoint(cl, ml, appConfig))`  
  
2. Edit:  
    - `app.Post("/v1/edits", openai.EditEndpoint(cl, ml, appConfig))`  
    - `app.Post("/edits", openai.EditEndpoint(cl, ml, appConfig))`  
  
3. Assistant:  
    - List assistants:  
        - `app.Get("/v1/assistants", openai.ListAssistantsEndpoint(cl, ml, appConfig))`  
        - `app.Get("/assistants", openai.ListAssistantsEndpoint(cl, ml, appConfig))`  
    - Create assistant:  
        - `app.Post("/v1/assistants", openai.CreateAssistantEndpoint(cl, ml, appConfig))`  
        - `app.Post("/assistants", openai.CreateAssistantEndpoint(cl, ml, appConfig))`  
    - Delete assistant:  
        - `app.Delete("/v1/assistants/:assistant_id", openai.DeleteAssistantEndpoint(cl, ml, appConfig))`  
        - `app.Delete("/assistants/:assistant_id", openai.DeleteAssistantEndpoint(cl, ml, appConfig))`  
    - Get assistant:  
        - `app.Get("/v1/assistants/:assistant_id", openai.GetAssistantEndpoint(cl, ml, appConfig))`  
        - `app.Get("/assistants/:assistant_id", openai.GetAssistantEndpoint(cl, ml, appConfig))`  
    - Modify assistant:  
        - `app.Post("/v1/assistants/:assistant_id", openai.ModifyAssistantEndpoint(cl, ml, appConfig))`  
        - `app.Post("/assistants/:assistant_id", openai.ModifyAssistantEndpoint(cl, ml, appConfig))`  
    - List assistant files:  
        - `app.Get("/v1/assistants/:assistant_id/files", openai.ListAssistantFilesEndpoint(cl, ml, appConfig))`  
        - `app.Get("/assistants/:assistant_id/files", openai.ListAssistantFilesEndpoint(cl, ml, appConfig))`  
    - Create assistant file:  
        - `app.Post("/v1/assistants/:assistant_id/files", openai.CreateAssistantFileEndpoint(cl, ml, appConfig))`  
        - `app.Post("/assistants/:assistant_id/files", openai.CreateAssistantFileEndpoint(cl, ml, appConfig))`  
    - Delete assistant file:  
        - `app.Delete("/v1/assistants/:assistant_id/files/:file_id", openai.DeleteAssistantFileEndpoint(cl, ml, appConfig))`  
        - `app.Delete("/assistants/:assistant_id/files/:file_id", openai.DeleteAssistantFileEndpoint(cl, ml, appConfig))`  
    - Get assistant file:  
        - `app.Get("/v1/assistants/:assistant_id/files/:file_id", openai.GetAssistantFileEndpoint(cl, ml, appConfig))`  
        - `app.Get("/assistants/:assistant_id/files/:file_id", openai.GetAssistantFileEndpoint(cl, ml, appConfig))`  
  
4. Files:  
    - Upload files:  
        - `app.Post("/v1/files", openai.UploadFilesEndpoint(cl, appConfig))`  
        - `app.Post("/files", openai.UploadFilesEndpoint(cl, appConfig))`  
    - List files:  
        - `app.Get("/v1/files", openai.ListFilesEndpoint(cl, appConfig))`  
        - `app.Get("/files", openai.ListFilesEndpoint(cl, appConfig))`  
    - Get file:  
        - `app.Get("/v1/files/:file_id", openai.GetFilesEndpoint(cl, appConfig))`  
        - `app.Get("/files/:file_id", openai.GetFilesEndpoint(cl, appConfig))`  
    - Delete file:  
        - `app.Delete("/v1/files/:file_id", openai.DeleteFilesEndpoint(cl, appConfig))`  
        - `app.Delete("/files/:file_id", openai.DeleteFilesEndpoint(cl, appConfig))`  
    - Get file content:  
        - `app.Get("/v1/files/:file_id/content", openai.GetFilesContentsEndpoint(cl, appConfig))`  
        - `app.Get("/files/:file_id/content", openai.GetFilesContentsEndpoint(cl, appConfig))`  
  
5. Completion:  
    - `app.Post("/v1/completions", openai.CompletionEndpoint(cl, ml, appConfig))`  
    - `app.Post("/completions", openai.CompletionEndpoint(cl, ml, appConfig))`  
    - `app.Post("/v1/engines/:model/completions", openai.CompletionEndpoint(cl, ml, appConfig))`  
  
6. Embeddings:  
    - `app.Post("/v1/embeddings", openai.EmbeddingsEndpoint(cl, ml, appConfig))`  
    - `app.Post("/embeddings", openai.EmbeddingsEndpoint(cl, ml, appConfig))`  
    - `app.Post("/v1/engines/:model/embeddings", openai.EmbeddingsEndpoint(cl, ml, appConfig))`  
  
7. Audio:  
    - Transcript:  
        - `app.Post("/v1/audio/transcriptions", openai.TranscriptEndpoint(cl, ml, appConfig))`  
    - Speech:  
        - `app.Post("/v1/audio/speech", localai.TTSEndpoint(cl, ml, appConfig))`  
  
8. Images:  
    - Image generation:  
        - `app.Post("/v1/images/generations", openai.ImageEndpoint(cl, ml, appConfig))`  
  
9. Static files:  
    - If `appConfig.ImageDir` is not empty, serve generated images from the specified directory:  
        - `app.Static("/generated-images", appConfig.ImageDir)`  
    - If `appConfig.AudioDir` is not empty, serve generated audio files from the specified directory:  
        - `app.Static("/generated-audio", appConfig.AudioDir)`  
  
10. List models:  
    - `app.Get("/v1/models", openai.ListModelsEndpoint(cl, ml))`  
    - `app.Get("/models", openai.ListModelsEndpoint(cl, ml))`  
  
# core/http/routes/ui.go  
```  
package routes  
  
import (  
	"fmt"  
	"html/template"  
	"sort"  
	"strings"  
  
	"github.com/microcosm-cc/bluemonday"  
	"github.com/mudler/LocalAI/core/config"  
	"github.com/mudler/LocalAI/core/gallery"  
	"github.com/mudler/LocalAI/core/http/elements"  
	"github.com/mudler/LocalAI/core/http/endpoints/localai"  
	"github.com/mudler/LocalAI/core/p2p"  
	"github.com/mudler/LocalAI/core/services"  
	"github.com/mudler/LocalAI/internal"  
	"github.com/mudler/LocalAI/pkg/model"  
	"github.com/mudler/LocalAI/pkg/xsync"  
	"github.com/rs/zerolog/log"  
  
	"github.com/gofiber/fiber/v2"  
	"github.com/google/uuid"  
)  
  
type modelOpCache struct {  
	status *xsync.SyncedMap[string, string]  
}  
  
func NewModelOpCache() *modelOpCache {  
	return &modelOpCache{  
		status: xsync.NewSyncedMap[string, string](),  
	}  
}  
  
func (m *modelOpCache) Set(key string, value string) {  
	m.status.Set(key, value)  
}  
  
func (m *modelOpCache) Get(key string) string {  
	return m.status.Get(key)  
}  
  
func (m *modelOpCache) DeleteUUID(uuid string) {  
	for _, k := range m.status.Keys() {  
		if m.status.Get(k) == uuid {  
			m.status.Delete(k)  
		}  
	}  
}  
  
func (m *modelOpCache) Map() map[string]string {  
	return m.status.Map()  
}  
  
func (m *modelOpCache) Exists(key string) bool {  
	return m.status.Exists(key)  
}  
  
func RegisterUIRoutes(app *fiber.App,  
	cl *config.BackendConfigLoader,  
	ml *model.ModelLoader,  
	appConfig *config.ApplicationConfig,  
	galleryService *services.GalleryService) {  
  
	// keeps the state of models that are being installed from the UI  
	var processingModels = NewModelOpCache()  
  
	// modelStatus returns the current status of the models being processed (installation or deletion)  
	// it is called asynchonously from the UI  
	modelStatus := func() (map[string]string, map[string]string) {  
		processingModelsData := processingModels.Map()  
  
		taskTypes := map[string]string{}  
  
		for k, v := range processingModelsData {  
			status := galleryService.GetStatus(v)  
			taskTypes[k] = "Installation"  
			if status != nil && status.Deletion {  
				taskTypes[k] = "Deletion"  
			} else if status == nil {  
				taskTypes[k] = "Waiting"  
			}  
		}  
  
		return processingModelsData, taskTypes  
	}  
  
	app.Get("/", localai.WelcomeEndpoint(appConfig, cl, ml, modelStatus))  
  
	if p2p.IsP2PEnabled() {  
		app.Get("/p2p", func(c *fiber.Ctx) error {  
			summary := fiber.Map{  
				"Title":   "LocalAI - P2P dashboard",  
				"Version": internal.PrintableVersion(),  
				//"Nodes":          p2p.GetAvailableNodes(""),  
				//"FederatedNodes": p2p.GetAvailableNodes(p2p.FederatedID),  
				"IsP2PEnabled": p2p.IsP2PEnabled(),  
				"P2PToken":     appConfig.P2PToken,  
				"NetworkID":    appConfig.P2PNetworkID,  
			}  
  
			// Render index  
			return c.Render("views/p2p", summary)  
		})  
  
		/* show nodes live! */  
		app.Get("/p2p/ui/workers", func(c *fiber.Ctx) error {  
			return c.SendString(elements.P2PNodeBoxes(p2p.GetAvailableNodes(p2p.NetworkID(appConfig.P2PNetworkID, p2p.WorkerID))))  
		})  
		app.Get("/p2p/ui/workers-federation", func(c *fiber.Ctx) error {  
			return c.SendString(elements.P2PNodeBoxes(p2p.GetAvailableNodes(p2p.NetworkID(appConfig.P2PNetworkID, p2p.FederatedID))))  
		})  
  
		app.Get("/p2p/ui/workers-stats", func(c *fiber.Ctx) error {  
			return c.SendString(elements.P2PNodeStats(p2p.GetAvailableNodes(p2p.NetworkID(appConfig.P2PNetworkID, p2p.WorkerID))))  
		})  
		app.Get("/p2p/ui/workers-federation-stats", func(c *fiber.Ctx) error {  
			return c.SendString(elements.P2PNodeStats(p2p.GetAvailableNodes(p2p.NetworkID(appConfig.P2PNetworkID, p2p.FederatedID))))  
		})  
	}  
  
	if !appConfig.DisableGalleryEndpoint {  
  
		// Show the Models page (all models)  
		app.Get("/browse", func(c *fiber.Ctx) error {  
			term := c.Query("term")  
  
			models, _ := gallery.AvailableGalleryModels(appConfig.Galleries, appConfig.ModelPath)  
  
			// Get all available tags  
			allTags := map[string]struct{}{}  
			tags := []string{}  
			for _, m := range models {  
				for _, t := range m.Tags {  
					allTags[t] = struct{}{}  
				}  
			}  
			for t := range allTags {  
				tags = append(tags, t)  
			}  
			sort.Strings(tags)  
  
			if term != "" {  
				models = gallery.GalleryModels(models).Search(term)  
			}  
  
			// Get model statuses  
			processingModelsData, taskTypes := modelStatus()  
  
			summary := fiber.Map{  
				"Title":            "LocalAI - Models",  
				"Version":          internal.PrintableVersion(),  
				"Models":           template.HTML(elements.ListModels(models, processingModels, galleryService)),  
				"Repositories":     appConfig.Galleries,  
				"AllTags":          tags,  
				"ProcessingModels": processingModelsData,  
				"AvailableModels":  len(models),  
				"IsP2PEnabled":     p2p.IsP2PEnabled(),  
  
				"TaskTypes": taskTypes,  
				//	"ApplicationConfig": appConfig,  
			}  
  
			// Render index  
			return c.Render("views/models", summary)  
		})  
  
		// Show the models, filtered from the user input  
		// https://htmx.org/examples/active-search/  
		app.Post("/browse/search/models", func(c *fiber.Ctx) error {  
			form := struct {  
				Search string `form:"search"`  
			}{}  
			if err := c.BodyParser(&form); err != nil {  
				return c.Status(fiber.StatusBadRequest).SendString(bluemonday.StrictPolicy().Sanitize(err.Error()))  
			}  
  
			models, _ := gallery.AvailableGalleryModels(appConfig.Galleries, appConfig.ModelPath)  
  
			return c.SendString(elements.ListModels(gallery.GalleryModels(models).Search(form.Search), processingModels, galleryService))  
		})  
  
		/*  
  
			Install routes  
  
		*/  
  
		// This route is used when the "Install" button is pressed, we submit here a new job to the gallery service  
		// https://htmx.org/examples/progress-bar/  
		app.Post("/browse/install/model/:id", func(c *fiber.Ctx) error {  
			galleryID := strings.Clone(c.Params("id")) // note: strings.Clone is required for multiple requests!  
			log.Debug().Msgf("UI job submitted to install  : %+v\n", galleryID)  
  
			id, err := uuid.NewUUID()  
			if err != nil {  
				return err  
			}  
  
			uid := id.String()  
  
			processingModels.Set(galleryID, uid)  
  
			op := gallery.GalleryOp{  
				Id:               uid,  
				GalleryModelName: galleryID,  
				Galleries:        appConfig.Galleries,  
			}  
			go func() {  
				galleryService.C <- op  
			}()  
  
			return c.SendString(elements.StartProgressBar(uid, "0", "Installation"))  
		})  
  
		// This route is used when the "Install" button is pressed, we submit here a new job to the gallery service  
		// https://htmx.org/examples/progress-bar/  
		app.Post("/browse/delete/model/:id", func(c *fiber.Ctx) error {  
			galleryID := strings.Clone(c.Params("id")) // note: strings.Clone is required for multiple requests!  
			log.Debug().Msgf("UI job submitted to delete  : %+v\n", galleryID)  
			var galleryName = galleryID  
			if strings.Contains(galleryID, "@") {  
				// if the galleryID contains a @ it means that it's a model from a gallery  
				// but we want to delete it from the local models which does not need  
				// a repository ID  
				galleryName = strings.Split(galleryID, "@")[1]  
			}  
  
			id, err := uuid.NewUUID()  
			if err != nil {  
				return err  
			}  
  
			uid := id.String()  
  
			// Track the deletion job by galleryID and galleryName  
			// The GalleryID contains information about the repository,  
			// while the GalleryName is ONLY the name of the model  
			processingModels.Set(galleryName, uid)  
			processingModels.Set(galleryID, uid)  
  
			op := gallery.GalleryOp{  
				Id:               uid,  
				Delete:           true,  
				GalleryModelName: galleryName,  
			}  
			go func() {  
				galleryService.C <- op  
				cl.RemoveBackendConfig(galleryName)  
			}()  
  
			return c.SendString(elements.StartProgressBar(uid, "0", "Deletion"))  
		})  
  
		// Display the job current progress status  
		// If the job is done, we trigger the /browse/job/:uid route  
		// https://htmx.org/examples/progress-bar/  
		app.Get("/browse/job/progress/:uid", func(c *fiber.Ctx) error {  
			jobUID := strings.Clone(c.Params("uid")) // note: strings.Clone is required for multiple requests!  
  
			status := galleryService.GetStatus(jobUID)  
			if status == nil {  
				//fmt.Errorf("could not find any status for ID")  
				return c.SendString(elements.ProgressBar("0"))  
			}  
  
			if status.Progress == 100 {  
				c.Set("HX-Trigger", "done") // this triggers /browse/job/:uid (which is when the job is done)  
				return c.SendString(elements.ProgressBar("100"))  
			}  
			if status.Error != nil {  
				// TODO: instead of deleting the job, we should keep it in the cache and make it dismissable by the user  
				processingModels.DeleteUUID(jobUID)  
				return c.SendString(elements.ErrorProgress(status.Error.Error(), status.GalleryModelName))  
			}  
  
			return c.SendString(elements.ProgressBar(fmt.Sprint(status.Progress)))  
		})  
  
		// this route is hit when the job is done, and we display the  
		// final state (for now just displays "Installation completed")  
		app.Get("/browse/job/:uid", func(c *fiber.Ctx) error {  
			jobUID := strings.Clone(c.Params("uid")) // note: strings.Clone is required for multiple requests!  
  
			status := galleryService.GetStatus(jobUID)  
  
			galleryID := ""  
			processingModels.DeleteUUID(jobUID)  
			if galleryID == "" {  
				log.Debug().Msgf("no processing model found for job : %+v\n", jobUID)  
			}  
  
			log.Debug().Msgf("JOB finished  : %+v\n", status)  
			showDelete := true  
			displayText := "Installation completed"  
			if status.Deletion {  
				showDelete = false  
				displayText = "Deletion completed"  
			}  
  
			return c.SendString(elements.DoneProgress(galleryID, displayText, showDelete))  
		})  
	}  
  
	// Show the Chat page  
	app.Get("/chat/:model", func(c *fiber.Ctx) error {  
		backendConfigs, _ := services.ListModels(cl, ml, config.NoFilterFn, services.SKIP_IF_CONFIGURED)  
  
		summary := fiber.Map{  
			"Title":        "LocalAI - Chat with " + c.Params("model"),  
			"ModelsConfig": backendConfigs,  
			"Model":        c.Params("model"),  
			"Version":      internal.PrintableVersion(),  
			"IsP2PEnabled": p2p.IsP2PEnabled(),  
		}  
  
		// Render index  
		return c.Render("views/chat", summary)  
	})  
  
	app.Get("/talk/", func(c *fiber.Ctx) error {  
		backendConfigs, _ := services.ListModels(cl, ml, config.NoFilterFn, services.SKIP_IF_CONFIGURED)  
  
		if len(backendConfigs) == 0 {  
			// If no model is available redirect to the index which suggests how to install models  
			return c.Redirect("/")  
		}  
  
		summary := fiber.Map{  
			"Title":        "LocalAI - Talk",  
			"ModelsConfig": backendConfigs,  
			"Model":        backendConfigs[0],  
			"IsP2PEnabled": p2p.IsP2PEnabled(),  
			"Version":      internal.PrintableVersion(),  
		}  
  
		// Render index  
		return c.Render("views/talk", summary)  
	})  
  
	app.Get("/chat/", func(c *fiber.Ctx) error {  
  
		backendConfigs, _ := services.ListModels(cl, ml, config.NoFilterFn, services.SKIP_IF_CONFIGURED)  
  
		if len(backendConfigs) == 0 {  
			// If no model is available redirect to the index which suggests how to install models  
			return c.Redirect("/")  
		}  
  
		summary := fiber.Map{  
			"Title":        "LocalAI - Chat with " + backendConfigs[0],  
			"ModelsConfig": backendConfigs,  
			"Model":        backendConfigs[0],  
			"IsP2PEnabled": p2p.IsP2PEnabled(),  
			"Version":      internal.PrintableVersion(),  
		}  
  
		// Render index  
		return c.Render("views/chat", summary)  
	})  
  
	app.Get("/text2image/:model", func(c *fiber.Ctx) error {  
		backendConfigs := cl.GetAllBackendConfigs()  
  
		summary := fiber.Map{  
			"Title":        "LocalAI - Generate images with " + c.Params("model"),  
			"ModelsConfig": backendConfigs,  
			"Model":        c.Params("model"),  
			"Version":      internal.PrintableVersion(),  
			"IsP2PEnabled": p2p.IsP2PEnabled(),  
		}  
  
		// Render index  
		return c.Render("views/text2image", summary)  
	})  
  
	app.Get("/text2image/", func(c *fiber.Ctx) error {  
  
		backendConfigs := cl.GetAllBackendConfigs()  
  
		if len(backendConfigs) == 0 {  
			// If no model is available redirect to the index which suggests how to install models  
			return c.Redirect("/")  
		}  
  
		summary := fiber.Map{  
			"Title":        "LocalAI - Generate images with " + backendConfigs[0].Name,  
			"ModelsConfig": backendConfigs,  
			"Model":        backendConfigs[0].Name,  
			"IsP2PEnabled": p2p.IsP  
  
