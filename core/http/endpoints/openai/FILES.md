# core/http/endpoints/openai/assistant.go  
```  
package openai  
  
import (  
	"fmt"  
	"net/http"  
	"sort"  
	"strconv"  
	"strings"  
	"sync/atomic"  
	"time"  
  
	"github.com/gofiber/fiber/v2"  
	"github.com/rs/zerolog/log"  
	"github.com/mudler/LocalAI/core/config"  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/model"  
	"github.com/mudler/LocalAI/pkg/utils"  
)  
  
// ToolType defines a type for tool options  
type ToolType string  
  
const (  
	CodeInterpreter ToolType = "code_interpreter"  
	Retrieval       ToolType = "retrieval"  
	Function        ToolType = "function"  
  
	MaxCharacterInstructions  = 32768  
	MaxCharacterDescription   = 512  
	MaxCharacterName          = 256  
	MaxToolsSize              = 128  
	MaxFileIdSize             = 20  
	MaxCharacterMetadataKey   = 64  
	MaxCharacterMetadataValue = 512  
)  
  
type Tool struct {  
	Type ToolType `json:"type"`  
}  
  
// Assistant represents the structure of an assistant object from the OpenAI API.  
type Assistant struct {  
	ID           string            `json:"id"`  
	Object       string            `json:"object"`  
	Created      int64             `json:"created"`  
	Model        string            `json:"model"`  
	Name         string            `json:"name,omitempty"`  
	Description  string            `json:"description,omitempty"`  
	Instructions string            `json:"instructions,omitempty"`  
	Tools        []Tool            `json:"tools,omitempty"`  
	FileIDs      []string          `json:"file_ids,omitempty"`  
	Metadata     map[string]string `json:"metadata,omitempty"`  
}  
  
var (  
	Assistants           = []Assistant{}  
	AssistantsConfigFile = "assistants.json"  
)  
  
type AssistantRequest struct {  
	Model        string            `json:"model"`  
	Name         string            `json:"name,omitempty"`  
	Description  string            `json:"description,omitempty"`  
	Instructions string            `json:"instructions,omitempty"`  
	Tools        []Tool            `json:"tools,omitempty"`  
	FileIDs      []string          `json:"file_ids,omitempty"`  
	Metadata     map[string]string `json:"metadata,omitempty"`  
}  
  
// CreateAssistantEndpoint is the OpenAI Assistant API endpoint https://platform.openai.com/docs/api-reference/assistants/createAssistant  
// @Summary Create an assistant with a model and instructions.  
// @Param request body AssistantRequest true "query params"  
// @Success 200 {object} Assistant "Response"  
// @Router /v1/assistants [post]  
func CreateAssistantEndpoint(cl *config.BackendConfigLoader, ml *model.ModelLoader, appConfig *config.ApplicationConfig) func(c *fiber.Ctx) error {  
	return func(c *fiber.Ctx) error {  
		request := new(AssistantRequest)  
		if err := c.BodyParser(request); err != nil {  
			log.Warn().AnErr("Unable to parse AssistantRequest", err)  
			return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error": "Cannot parse JSON"})  
		}  
  
		if !modelExists(cl, ml, request.Model) {  
			log.Warn().Msgf("Model: %s was not found in list of models.", request.Model)  
			return c.Status(fiber.StatusBadRequest).SendString(bluemonday.StrictPolicy().Sanitize(fmt.Sprintf("Model %q not found", request.Model)))  
		}  
  
		if request.Tools == nil {  
			request.Tools = []Tool{}  
		}  
  
		if request.FileIDs == nil {  
			request.FileIDs = []string{}  
		}  
  
		if request.Metadata == nil {  
			request.Metadata = make(map[string]string)  
		}  
  
		id := "asst_" + strconv.FormatInt(generateRandomID(), 10)  
  
		assistant := Assistant{  
			ID:           id,  
			Object:       "assistant",  
			Created:      time.Now().Unix(),  
			Model:        request.Model,  
			Name:         request.Name,  
			Description:  request.Description,  
			Instructions: request.Instructions,  
			Tools:        request.Tools,  
			FileIDs:      request.FileIDs,  
			Metadata:     request.Metadata,  
		}  
  
		Assistants = append(Assistants, assistant)  
		utils.SaveConfig(appConfig.ConfigsDir, AssistantsConfigFile, Assistants)  
		return c.Status(fiber.StatusOK).JSON(assistant)  
	}  
}  
  
var currentId int64 = 0  
  
func generateRandomID() int64 {  
	atomic.AddInt64(&currentId, 1)  
	return currentId  
}  
  
// ListAssistantsEndpoint is the OpenAI Assistant API endpoint to list assistents https://platform.openai.com/docs/api-reference/assistants/listAssistants  
// @Summary List available assistents  
// @Param limit query int false "Limit the number of assistants returned"  
// @Param order query string false "Order of assistants returned"  
// @Param after query string false "Return assistants created after the given ID"  
// @Param before query string false "Return assistants created before the given ID"  
// @Success 200 {object} []Assistant "Response"  
// @Router /v1/assistants [get]  
func ListAssistantsEndpoint(cl *config.BackendConfigLoader, ml *model.ModelLoader, appConfig *config.ApplicationConfig) func(c *fiber.Ctx) error {  
	return func(c *fiber.Ctx) error {  
		// Because we're altering the existing assistants list we should just duplicate it for now.  
		returnAssistants := Assistants  
		// Parse query parameters  
		limitQuery := c.Query("limit", "20")  
		orderQuery := c.Query("order", "desc")  
		afterQuery := c.Query("after")  
		beforeQuery := c.Query("before")  
  
		// Convert string limit to integer  
		limit, err := strconv.Atoi(limitQuery)  
		if err != nil {  
			return c.Status(http.StatusBadRequest).SendString(bluemonday.StrictPolicy().Sanitize(fmt.Sprintf("Invalid limit query value: %s", limitQuery)))  
		}  
  
		// Sort assistants  
		sort.SliceStable(returnAssistants, func(i, j int) bool {  
			if orderQuery == "asc" {  
				return returnAssistants[i].Created < returnAssistants[j].Created  
			}  
			return returnAssistants[i].Created > returnAssistants[j].Created  
		})  
  
		// After and before cursors  
		if afterQuery != "" {  
			returnAssistants = filterAssistantsAfterID(returnAssistants, afterQuery)  
		}  
		if beforeQuery != "" {  
			returnAssistants = filterAssistantsBeforeID(returnAssistants, beforeQuery)  
		}  
  
		// Apply limit  
		if limit < len(returnAssistants) {  
			returnAssistants = returnAssistants[:limit]  
		}  
  
		return c.JSON(returnAssistants)  
	}  
}  
  
// FilterAssistantsBeforeID filters out those assistants whose ID comes before the given ID  
// We assume that the assistants are already sorted  
func filterAssistantsBeforeID(assistants []Assistant, id string) []Assistant {  
	idInt, err := strconv.Atoi(id)  
	if err != nil {  
		return assistants // Return original slice if invalid id format is provided  
	}  
  
	var filteredAssistants []Assistant  
  
	for _, assistant := range assistants {  
		aid, err := strconv.Atoi(strings.TrimPrefix(assistant.ID, "asst_"))  
		if err != nil {  
			continue // Skip if invalid id in assistant  
		}  
  
		if aid < idInt {  
			filteredAssistants = append(filteredAssistants, assistant)  
		}  
	}  
  
	return filteredAssistants  
}  
  
// FilterAssistantsAfterID filters out those assistants whose ID comes after the given ID  
// We assume that the assistants are already sorted  
func filterAssistantsAfterID(assistants []Assistant, id string) []Assistant {  
	idInt, err := strconv.Atoi(id)  
	if err != nil {  
		return assistants // Return original slice if invalid id format is provided  
	}  
  
	var filteredAssistants []Assistant  
  
	for _, assistant := range assistants {  
		aid, err := strconv.Atoi(strings.TrimPrefix(assistant.ID, "asst_"))  
		if err != nil {  
			continue // Skip if invalid id in assistant  
		}  
  
		if aid > idInt {  
			filteredAssistants = append(filteredAssistants, assistant)  
		}  
	}  
  
	return filteredAssistants  
}  
  
func modelExists(cl *config.BackendConfigLoader, ml *model.ModelLoader, modelName string) (found bool) {  
	models, err := model.ListModels(cl, ml, config.NoFilterFn, model.SKIP_IF_CONFIGURED)  
	if err != nil {  
		return  
	}  
  
	for _, model := range models {  
		if model == modelName {  
			found = true  
			return  
		}  
	}  
	return  
}  
  
// DeleteAssistantEndpoint is the OpenAI Assistant API endpoint to delete assistents https://platform.openai.com/docs/api-reference/assistants/deleteAssistant  
// @Summary Delete assistents  
// @Success 200 {object} schema.DeleteAssistantResponse "Response"  
// @Router /v1/assistants/{assistant_id} [delete]  
func DeleteAssistantEndpoint(cl *config.BackendConfigLoader, ml *model.ModelLoader, appConfig *config.ApplicationConfig) func(c *fiber.Ctx) error {  
	return func(c *fiber.Ctx) error {  
		assistantID := c.Params("assistant_id")  
		if assistantID == "" {  
			return c.Status(fiber.StatusBadRequest).SendString("parameter assistant_id is required")  
		}  
  
		for i, assistant := range Assistants {  
			if assistant.ID == assistantID {  
				Assistants = append(Assistants[:i], Assistants[i+1:]...)  
				utils.SaveConfig(appConfig.ConfigsDir, AssistantsConfigFile, Assistants)  
				return c.Status(fiber.StatusOK).JSON(schema.DeleteAssistantResponse{  
					ID:      assistantID,  
					Object:  "assistant.deleted",  
					Deleted: true,  
				})  
			}  
		}  
  
		log.Warn().Msgf("Unable to find assistant %s for deletion", assistantID)  
		return c.Status(fiber.StatusNotFound).JSON(schema.DeleteAssistantResponse{  
			ID:      assistantID,  
			Object:  "assistant.deleted",  
			Deleted: false,  
		})  
	}  
}  
  
// GetAssistantEndpoint is the OpenAI Assistant API endpoint to get assistents https://platform.openai.com/docs/api-reference/assistants/getAssistant  
// @Summary Get assistent data  
// @Success 200 {object} Assistant "Response"  
// @Router /v1/assistants/{assistant_id} [get]  
func GetAssistantEndpoint(cl *config.BackendConfigLoader, ml *model.ModelLoader, appConfig *config.ApplicationConfig) func(c *fiber.Ctx) error {  
	return func(c *fiber.Ctx) error {  
		assistantID := c.Params("assistant_id")  
		if assistantID == "" {  
			return c.Status(fiber.StatusBadRequest).SendString("parameter assistant_id is required")  
		}  
  
		for _, assistant := range Assistants {  
			if assistant.ID == assistantID {  
				return c.Status(  
  
# core/http/endpoints/openai/assistant_test.go  
```  
package openai  
  
import (  
	"encoding/json"  
	"fmt"  
	"io"  
	"net/http"  
	"net/http/httptest"  
	"os"  
	"path/filepath"  
	"strings"  
	"testing"  
	"time"  
  
	"github.com/gofiber/fiber/v2"  
	"github.com/mudler/LocalAI/core/config"  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/model"  
	"github.com/stretchr/testify/assert"  
)  
  
var configsDir string = "/tmp/localai/configs"  
  
type MockLoader struct {  
	models []string  
}  
  
func tearDown() func() {  
	return func() {  
		UploadedFiles = []schema.File{}  
		Assistants = []Assistant{}  
		AssistantFiles = []AssistantFile{}  
		_ = os.Remove(filepath.Join(configsDir, AssistantsConfigFile))  
		_ = os.Remove(filepath.Join(configsDir, AssistantsFileConfigFile))  
	}  
}  
  
func TestAssistantEndpoints(t *testing.T) {  
	// Preparing the mocked objects  
	cl := &config.BackendConfigLoader{}  
	//configsDir := "/tmp/localai/configs"  
	modelPath := "/tmp/localai/model"  
	var ml = model.NewModelLoader(modelPath)  
  
	appConfig := &config.ApplicationConfig{  
		ConfigsDir:    configsDir,  
		UploadLimitMB: 10,  
		UploadDir:     "test_dir",  
		ModelPath:     modelPath,  
	}  
  
	_ = os.RemoveAll(appConfig.ConfigsDir)  
	_ = os.MkdirAll(appConfig.ConfigsDir, 0750)  
	_ = os.MkdirAll(modelPath, 0750)  
	os.Create(filepath.Join(modelPath, "ggml-gpt4all-j"))  
  
	app := fiber.New(fiber.Config{  
		BodyLimit: 20 * 1024 * 1024, // sets the limit to 20MB.  
	})  
  
	// Create a Test Server  
	app.Get("/assistants", ListAssistantsEndpoint(cl, ml, appConfig))  
	app.Post("/assistants", CreateAssistantEndpoint(cl, ml, appConfig))  
	app.Delete("/assistants/:assistant_id", DeleteAssistantEndpoint(cl, ml, appConfig))  
	app.Get("/assistants/:assistant_id", GetAssistantEndpoint(cl, ml, appConfig))  
	app.Post("/assistants/:assistant_id", ModifyAssistantEndpoint(cl, ml, appConfig))  
  
	app.Post("/files", UploadFilesEndpoint(cl, appConfig))  
	app.Get("/assistants/:assistant_id/files", ListAssistantFilesEndpoint(cl, ml, appConfig))  
	app.Post("/assistants/:assistant_id/files", CreateAssistantFileEndpoint(cl, ml, appConfig))  
	app.Delete("/assistants/:assistant_id/files/:file_id", DeleteAssistantFileEndpoint(cl, ml, appConfig))  
	app.Get("/assistants/:assistant_id/files/:file_id", GetAssistantFileEndpoint(cl, ml, appConfig))  
  
	t.Run("CreateAssistantEndpoint", func(t *testing.T) {  
		t.Cleanup(tearDown())  
		ar := &AssistantRequest{  
			Model:        "ggml-gpt4all-j",  
			Name:         "3.5-turbo",  
			Description:  "Test Assistant",  
			Instructions: "You are computer science teacher answering student questions",  
			Tools:        []Tool{{Type: Function}},  
			FileIDs:      nil,  
			Metadata:     nil,  
		}  
  
		resultAssistant, resp, err := createAssistant(app, *ar)  
		assert.NoError(t, err)  
		assert.Equal(t, fiber.StatusOK, resp.StatusCode)  
  
		assert.Equal(t, 1, len(Assistants))  
		//t.Cleanup(cleanupAllAssistants(t, app, []string{resultAssistant.ID}))  
  
		assert.Equal(t, ar.Name, resultAssistant.Name)  
		assert.Equal(t, ar.Model, resultAssistant.Model)  
		assert.Equal(t, ar.Tools, resultAssistant.Tools)  
		assert.Equal(t, ar.Description, resultAssistant.Description)  
		assert.Equal(t, ar.Instructions, resultAssistant.Instructions)  
		assert.Equal(t, ar.FileIDs, resultAssistant.FileIDs)  
		assert.Equal(t, ar.Metadata, resultAssistant.Metadata)  
	})  
  
	t.Run("ListAssistantsEndpoint", func(t *testing.T) {  
		var ids []string  
		var resultAssistant []Assistant  
		for i := 0; i < 4; i++ {  
			ar := &AssistantRequest{  
				Model:        "ggml-gpt4all-j",  
				Name:         fmt.Sprintf("3.5-turbo-%d", i),  
				Description:  fmt.Sprintf("Test Assistant - %d", i),  
				Instructions: fmt.Sprintf("You are computer science teacher answering student questions - %d", i),  
				Tools:        []Tool{{Type: Function}},  
				FileIDs:      []string{"fid-1234"},  
				Metadata:     map[string]string{"meta": "data"},  
			}  
  
			//var err error  
			ra, _, err := createAssistant(app, *ar)  
			// Because we create the assistants so fast all end up with the same created time.  
			time.Sleep(time.Second)  
			resultAssistant = append(resultAssistant, ra)  
			assert.NoError(t, err)  
			ids = append(ids, resultAssistant[i].ID)  
		}  
  
		t.Cleanup(cleanupAllAssistants(t, app, ids))  
  
		tests := []struct {  
			name                 string  
			reqURL               string  
			expectedStatus       int  
			expectedResult       []Assistant  
			expectedStringResult string  
		}{  
			{  
				name:           "Valid Usage - limit only",  
				reqURL:         "/assistants?limit=2",  
				expectedStatus: http.StatusOK,  
				expectedResult: Assistants[:2], // Expecting the first two assistants  
			},  
			{  
				name:           "Valid Usage - order asc",  
				reqURL:         "/assistants?order=asc",  
				expectedStatus: http.StatusOK,  
				expectedResult: Assistants, // Expecting all assistants in ascending order  
			},  
			{  
				name:           "Valid Usage - order desc",  
				reqURL:         "/assistants?order=desc",  
				expectedStatus: http.StatusOK,  
				expectedResult: []Assistant{Assistants[3], Assistants[2], Assistants[1], Assistants[0]}, // Expecting all assistants in descending order  
			},  
			{  
				name:           "Valid Usage - after specific ID",  
				reqURL:         "/assistants?after=2",  
				expectedStatus: http.StatusOK,  
				// Note this is correct because it's put in descending order already  
				expectedResult: Assistants[:3], // Expecting assistants after (excluding) ID 2  
			},  
			{  
				name:           "Valid Usage - before specific ID",  
				reqURL:         "/assistants?before=4",  
				expectedStatus: http.StatusOK,  
				expectedResult: Assistants[2:], // Expecting assistants before (excluding) ID 3.  
			},  
			{  
				name:                 "Invalid Usage - non-integer limit",  
				reqURL:               "/assistants?limit=two",  
				expectedStatus:       http.StatusBadRequest,  
				expectedStringResult: "Invalid limit query value: two",  
			},  
			{  
				name:           "Invalid Usage - non-existing id in after",  
				reqURL:         "/assistants?after=100",  
				expectedStatus: http.StatusOK,  
				expectedResult: []Assistant(nil), // Expecting empty list as there are no IDs above 100  
			},  
		}  
  
		for _, tt := range tests {  
			t.Run(tt.name, func(t *testing.T) {  
				request := httptest.NewRequest(http.MethodGet, tt.reqURL, nil)  
				response, err := app.Test(request)  
				assert.NoError(t, err)  
				assert.Equal(t, tt.expectedStatus, response.StatusCode)  
				if tt.expectedStatus != fiber.StatusOK {  
					all, _ := io.ReadAll(response.Body)  
					assert.Equal(t, tt.expectedStringResult, string(all))  
				} else {  
					var result []Assistant  
					err = json.NewDecoder(response.Body).Decode(&result)  
					assert.NoError(t, err)  
  
					assert.Equal(t, tt.expectedResult, result)  
				}  
			})  
		}  
	})  
  
	t.Run("DeleteAssistantEndpoint", func(t *testing.T) {  
		ar := &AssistantRequest{  
			Model:        "ggml-gpt4all-j",  
			Name:         "3.5-turbo",  
			Description:  "Test Assistant",  
			Instructions: "You are computer science teacher answering student questions",  
			Tools:        []Tool{{Type: Function}},  
			FileIDs:      nil,  
			Metadata:     nil,  
		}  
  
		resultAssistant, _, err := createAssistant(app, *ar)  
		assert.NoError(t, err)  
  
		target := fmt.Sprintf("/assistants/%s", resultAssistant.ID)  
		deleteReq := httptest.NewRequest(http.MethodDelete, target, nil)  
		_, err = app.Test(deleteReq)  
		assert.NoError(t, err)  
		assert.Equal(t, 0, len(Assistants))  
	})  
  
	t.Run("GetAssistantEndpoint", func(t *testing.T) {  
		ar := &AssistantRequest{  
			Model:        "ggml-gpt4all-j",  
			Name:         "3.5-turbo",  
			Description:  "Test Assistant",  
			Instructions: "You are computer science teacher answering student questions",  
			Tools:        []Tool{{Type: Function}},  
			FileIDs:      nil,  
			Metadata:     nil,  
		}  
  
		resultAssistant, _, err := createAssistant(app, *ar)  
		assert.NoError(t, err)  
		t.Cleanup(cleanupAllAssistants(t, app, []string{resultAssistant.ID}))  
  
		target := fmt.Sprintf("/assistants/%s", resultAssistant.ID)  
		request := httptest.NewRequest(http.MethodGet, target, nil)  
		response, err := app.Test(request)  
		assert.NoError(t, err)  
  
		var getAssistant Assistant  
		err = json.NewDecoder(response.Body).Decode(&getAssistant)  
		assert.NoError(t, err)  
  
		assert.Equal(t, resultAssistant.ID, getAssistant.ID)  
	})  
  
	t.Run("ModifyAssistantEndpoint", func(t *testing.T) {  
		ar := &AssistantRequest{  
			Model:        "ggml-gpt4all-j",  
			Name:         "3.5-turbo",  
			Description:  "Test Assistant",  
			Instructions: "You are computer science teacher answering student questions",  
			Tools:        []Tool{{Type: Function}},  
			FileIDs:      nil,  
			Metadata:     nil,  
		}  
  
		resultAssistant, _, err := createAssistant(app, *ar)  
		assert.NoError(t, err)  
  
		modifiedAr := &AssistantRequest{  
			Model:        "ggml-gpt4all-j",  
			Name:         "4.0-turbo",  
			Description:  "Modified Test Assistant",  
			Instructions: "You are math teacher answering student questions",  
			Tools:        []Tool{{Type: CodeInterpreter}},  
			FileIDs:      nil,  
			Metadata:     nil,  
		}  
  
		modifiedArJson, err := json.Marshal(modifiedAr)  
		assert.NoError(t, err)  
  
		target := fmt.Sprintf("/assistants/%s", resultAssistant.ID)  
		request := httptest.NewRequest(http.MethodPost, target, strings.NewReader(string(modifiedArJson)))  
		request.Header.Set(fiber.HeaderContentType, "application/json")  
  
		modifyResponse, err := app.Test(request)  
		assert.NoError(t, err)  
		var getAssistant Assistant  
		err = json.NewDecoder(modifyResponse.Body).Decode(&getAssistant)  
		assert.NoError(t, err)  
  
		t.Cleanup(cleanupAllAssistants(t, app, []string{getAssistant.ID}))  
  
		assert.Equal(t, resultAssistant.ID, getAssistant.ID)  
		assert.Equal(t, modifiedAr.Tools, getAssistant.Tools)  
		assert.Equal(t, modifiedAr.Name, getAssistant.Name)  
		assert.Equal(t, modifiedAr.Instructions, getAssistant.Instructions)  
		assert.Equal(t, modifiedAr.Description, getAssistant.Description)  
	})  
  
	t.Run("CreateAssistantFileEndpoint", func(t *testing.T) {  
		t.Cleanup(tearDown())  
		file, assistant, err := createFileAndAssistant(t, app, appConfig)  
		assert.NoError(t, err)  
  
		afr := schema.AssistantFileRequest{FileID: file.ID}  
		af, _, err := createAssistantFile(app, afr, assistant.ID)  
  
		assert.NoError(t, err)  
		assert.Equal(t, assistant.ID, af.AssistantID)  
	})  
	t.Run("ListAssistantFilesEndpoint", func(t *testing.T) {  
		t.Cleanup(tearDown())  
		file, assistant, err := createFileAndAssistant(t, app, appConfig)  
		assert.NoError(t, err)  
  
		af  
  
# core/http/endpoints/openai/chat.go  
```  
package openai  
  
import (  
	"bufio"  
	"bytes"  
	"encoding/json"  
	"fmt"  
	"strings"  
	"time"  
  
	"github.com/gofiber/fiber/v2"  
	"github.com/google/uuid"  
	"github.com/mudler/LocalAI/core/backend"  
	"github.com/mudler/LocalAI/core/config"  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/functions"  
	model "github.com/mudler/LocalAI/pkg/model"  
	"github.com/rs/zerolog/log"  
	"github.com/valyala/fasthttp"  
)  
  
// ChatEndpoint is the OpenAI Completion API endpoint https://platform.openai.com/docs/api-reference/chat/create  
// @Summary Generate a chat completions for a given prompt and model.  
// @Param request body schema.OpenAIRequest true "query params"  
// @Success 200 {object} schema.OpenAIResponse "Response"  
// @Router /v1/chat/completions [post]  
func ChatEndpoint(cl *config.BackendConfigLoader, ml *model.ModelLoader, startupOptions *config.ApplicationConfig) func(c *fiber.Ctx) error {  
	var id, textContentToReturn string  
	var created int  
  
	process := func(s string, req *schema.OpenAIRequest, config *config.BackendConfig, loader *model.ModelLoader, responses chan schema.OpenAIResponse) {  
		initialMessage := schema.OpenAIResponse{  
			ID:      id,  
			Created: created,  
			Model:   req.Model, // we have to return what the user sent here, due to OpenAI spec.  
			Choices: []schema.Choice{{Delta: &schema.Message{Role: "assistant", Content: &textContentToReturn}}},  
			Object:  "chat.completion.chunk",  
		}  
		responses <- initialMessage  
  
		ComputeChoices(req, s, config, startupOptions, loader, func(s string, c *[]schema.Choice) {}, func(s string, usage backend.TokenUsage) bool {  
			resp := schema.OpenAIResponse{  
				ID:      id,  
				Created: created,  
				Model:   req.Model, // we have to return what the user sent here, due to OpenAI spec.  
				Choices: []schema.Choice{{Delta: &schema.Message{Content: &s}, Index: 0}},  
				Object:  "chat.completion.chunk",  
				Usage: schema.OpenAIUsage{  
					PromptTokens:     usage.Prompt,  
					CompletionTokens: usage.Completion,  
					TotalTokens:      usage.Prompt + usage.Completion,  
				},  
			}  
  
			responses <- resp  
			return true  
		})  
		close(responses)  
	}  
  
	return func(c *fiber.Ctx) error {  
		textContentToReturn = ""  
		id = uuid.New().String()  
		created = int(time.Now().Unix())  
		// Set CorrelationID  
		correlationID := c.Get("X-Correlation-ID")  
		if len(strings.TrimSpace(correlationID)) == 0 {  
			correlationID = id  
		}  
		c.Set("X-Correlation-ID", correlationID)  
  
		modelFile, input, err := readRequest(c, cl, ml, startupOptions, true)  
		if err != nil {  
			return fmt.Errorf("failed reading parameters from request:%w", err)  
		}  
  
		config, input, err := mergeRequestWithConfig(modelFile, input, cl, ml, startupOptions.Debug, startupOptions.Threads, startupOptions.ContextSize, startupOptions.F16)  
		if err != nil {  
			return fmt.Errorf("failed reading parameters from request:%w", err)  
		}  
		log.Debug().Msgf("Configuration read: %+v", config)  
  
		funcs := input.Functions  
		shouldUseFn := len(input.Functions) > 0 && config.FunctionsConfig.ShouldUseFunctions()  
		strictMode := false  
  
		for _, f := range input.Functions {  
			if f.Strict {  
				strictMode = true  
				break  
			}  
		}  
  
		// Allow the user to set custom actions via config file  
		// to be "embedded" in each model  
		noActionName := "answer"  
		noActionDescription := "use this action to answer without performing any action"  
  
		if config.FunctionsConfig.NoActionFunctionName != "" {  
			noActionName = config.FunctionsConfig.NoActionFunctionName  
		}  
		if config.FunctionsConfig.NoActionDescriptionName != "" {  
			noActionDescription = config.FunctionsConfig.NoActionDescriptionName  
		}  
  
		if config.ResponseFormatMap != nil {  
			d := schema.ChatCompletionResponseFormat{}  
			dat, err := json.Marshal(config.ResponseFormatMap)  
			if err != nil {  
				return err  
			}  
			err = json.Unmarshal(dat, &d)  
			if err != nil {  
				return err  
			}  
			if d.Type == "json_object" {  
				input.Grammar = functions.JSONBNF  
			} else if d.Type == "json_schema" {  
				d := schema.JsonSchemaRequest{}  
				dat, err := json.Marshal(config.ResponseFormatMap)  
				if err != nil {  
					return err  
				}  
				err = json.Unmarshal(dat, &d)  
				if err != nil {  
					return err  
				}  
				fs := &functions.JSONFunctionStructure{  
					AnyOf: []functions.Item{d.JsonSchema.Schema},  
				}  
				g, err := fs.Grammar(config.FunctionsConfig.GrammarOptions()...)  
				if err == nil {  
					config.Grammar = g  
				}  
			}  
		}  
  
		config.Grammar = input.Grammar  
  
		if shouldUseFn {  
			log.Debug().Msgf("Response needs to process functions")  
		}  
  
		switch {  
		case (!config.FunctionsConfig.GrammarConfig.NoGrammar || strictMode) && shouldUseFn:  
			noActionGrammar := functions.Function{  
				Name:        noActionName,  
				Description: noActionDescription,  
				Parameters: map[string]interface{}{  
					"properties": map[string]interface{}{  
						"message": map[string]interface{}{  
							"type":        "string",  
							"description": "The message to reply the user with",  
						}},  
				},  
			}  
  
			// Append the no action function  
			if !config.FunctionsConfig.DisableNoAction {  
				funcs = append(funcs, noActionGrammar)  
			}  
  
			// Force picking one of the functions by the request  
			if config.FunctionToCall() != "" {  
				funcs = funcs.Select(config.FunctionToCall())  
			}  
  
			// Update input grammar  
			jsStruct := funcs.ToJSONStructure(config.FunctionsConfig.FunctionNameKey, config.FunctionsConfig.FunctionNameKey)  
			g, err := jsStruct.Grammar(config.FunctionsConfig.GrammarOptions()...)  
			if err == nil {  
				config.Grammar = g  
			}  
		case input.JSON  
  
# core/http/endpoints/openai/completion.go  
## openai package  
  
This package provides an endpoint for generating completions for a given prompt and model, similar to the OpenAI Completion API.  
  
### Imports  
  
```  
import (  
	"bufio"  
	"bytes"  
	"encoding/json"  
	"errors"  
	"fmt"  
	"time"  
  
	"github.com/mudler/LocalAI/core/backend"  
	"github.com/mudler/LocalAI/core/config"  
  
	"github.com/gofiber/fiber/v2"  
	"github.com/google/uuid"  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/functions"  
	model "github.com/mudler/LocalAI/pkg/model"  
	"github.com/rs/zerolog/log"  
	"github.com/valyala/fasthttp"  
)  
```  
  
### External Data, Input Sources  
  
The package uses the following external data and input sources:  
  
1. `config.BackendConfigLoader`: This is used to load backend configuration.  
2. `model.ModelLoader`: This is used to load model data.  
3. `config.ApplicationConfig`: This is used to load application configuration.  
4. `schema.OpenAIRequest`: This is the request structure for the OpenAI Completion API.  
5. `schema.OpenAIResponse`: This is the response structure for the OpenAI Completion API.  
  
### Code Summary  
  
The package contains a single function, `CompletionEndpoint`, which handles the OpenAI Completion API endpoint. This function takes three arguments:  
  
1. `cl`: A pointer to a `config.BackendConfigLoader` instance.  
2. `ml`: A pointer to a `model.ModelLoader` instance.  
3. `appConfig`: A pointer to a `config.ApplicationConfig` instance.  
  
The function first generates a unique ID and a timestamp for the request. Then, it defines a nested function called `process` that takes the input string, request, configuration, model loader, and a channel for responses as arguments. This function is responsible for generating the completion and sending it back to the caller.  
  
The main function then reads the request parameters, merges them with the configuration, and checks if a template file is associated with the model. If a template file is found, it is used to modify the input string.  
  
If the request is a streaming request, the function sets the appropriate headers and starts a goroutine to process the input and send the response in chunks. Otherwise, it calls the `ComputeChoices` function to generate the completion and returns the result as a JSON response.  
  
The `ComputeChoices` function is responsible for generating the completion based on the input, configuration, and model loader. It takes the input string, configuration, model loader, and two callback functions as arguments. The first callback function is used to append the generated completion to a slice of choices, while the second callback function is used to track the token usage.  
  
Finally, the function returns the completion as a JSON response, including the ID, timestamp, model, choices, and usage information.  
  
# core/http/endpoints/openai/edit.go  
## Package: openai  
  
### Imports:  
  
```  
encoding/json  
fmt  
time  
  
github.com/mudler/LocalAI/core/backend  
github.com/mudler/LocalAI/core/config  
  
github.com/gofiber/fiber/v2  
github.com/google/uuid  
github.com/mudler/LocalAI/core/schema  
model "github.com/mudler/LocalAI/pkg/model"  
  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
1. Request body: The code expects a request body containing the OpenAI edit request parameters.  
  
2. Configuration files: The code uses configuration files to load backend settings and application configuration.  
  
3. Model files: The code uses model files to load pre-trained models and templates.  
  
### Summary:  
  
#### EditEndpoint:  
  
The `EditEndpoint` function is responsible for handling the OpenAI edit API endpoint. It takes a configuration loader, model loader, and application configuration as input. The function first reads the request parameters from the request body and merges them with the configuration settings. It then uses the loaded model and template files to generate a response.  
  
The function iterates through the input strings and uses the loaded model to generate choices for each input. It calculates the total token usage for each input and appends the choices to a result list. Finally, it creates an OpenAI response object containing the choices, usage information, and other relevant data. The response is then returned as JSON.  
  
# core/http/endpoints/openai/embeddings.go  
## Package: openai  
  
### Imports:  
  
```  
encoding/json  
fmt  
time  
  
github.com/mudler/LocalAI/core/backend  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/pkg/model  
  
github.com/google/uuid  
github.com/mudler/LocalAI/core/schema  
  
github.com/gofiber/fiber/v2  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
- The code uses a `config.BackendConfigLoader` to load backend configuration.  
- It also uses a `model.ModelLoader` to load models.  
- The code expects a `config.ApplicationConfig` to be provided, which contains settings like debug mode, number of threads, context size, and F16 support.  
- The code expects a request body containing an `OpenAIRequest` object, which includes the model and input parameters.  
  
### EmbeddingsEndpoint:  
  
This function handles the OpenAI Embeddings API endpoint. It takes a `config.BackendConfigLoader`, a `model.ModelLoader`, and an `config.ApplicationConfig` as input.  
  
1. The function first reads the request parameters, including the model and input, from the request body and the provided configuration objects.  
2. It then merges the request parameters with the configuration settings, such as debug mode, number of threads, context size, and F16 support.  
3. The code then iterates through the input tokens and strings, calling the appropriate model embedding function for each item.  
4. For each item, it retrieves the embedding vector and appends it to a list of `schema.Item` objects.  
5. Finally, it creates an `schema.OpenAIResponse` object containing the embedding vectors, model information, and other relevant data. The response is then marshaled into JSON format and returned as the response body.  
  
The EmbeddingsEndpoint function provides a way to generate vector representations of input data using the OpenAI Embeddings API. It takes care of reading the request parameters, merging them with configuration settings, and calling the appropriate model embedding functions to generate the embedding vectors. The resulting embedding vectors are then returned as part of an `schema.OpenAIResponse` object.  
  
# core/http/endpoints/openai/files.go  
## openai package  
  
This package provides endpoints for uploading, listing, retrieving, deleting, and retrieving the contents of files.  
  
### Imports  
  
```  
import (  
	"errors"  
	"fmt"  
	"os"  
	"path/filepath"  
	"sync/atomic"  
	"time"  
  
	"github.com/microcosm-cc/bluemonday"  
	"github.com/mudler/LocalAI/core/config"  
	"github.com/mudler/LocalAI/core/schema"  
  
	"github.com/gofiber/fiber/v2"  
	"github.com/mudler/LocalAI/pkg/utils"  
)  
```  
  
### External data, input sources  
  
- UploadedFiles: A slice of schema.File structs that stores information about uploaded files.  
- UploadedFilesFile: The name of the file that stores the UploadedFiles data.  
- UploadedFilesFile: The name of the file that stores the UploadedFiles data.  
  
### Code summary  
  
#### UploadFilesEndpoint  
  
This endpoint handles the uploading of files to the system. It takes a file from the request, checks its size against the upload limit, and then saves the file to the specified directory. It also creates a schema.File struct to store information about the uploaded file and appends it to the UploadedFiles slice. Finally, it saves the updated UploadedFiles data to the UploadedFilesFile.  
  
#### ListFilesEndpoint  
  
This endpoint lists all uploaded files. It takes a purpose parameter from the query string and filters the UploadedFiles slice based on the purpose. If no purpose is provided, it returns all files.  
  
#### GetFilesEndpoint  
  
This endpoint retrieves information about a specific file based on its ID. It first retrieves the file from the UploadedFiles slice and then returns it as a JSON response.  
  
#### DeleteFilesEndpoint  
  
This endpoint deletes a file based on its ID. It first retrieves the file from the UploadedFiles slice and then removes it from the slice. It also removes the file from the file system and saves the updated UploadedFiles data to the UploadedFilesFile.  
  
#### GetFilesContentsEndpoint  
  
This endpoint retrieves the contents of a specific file based on its ID. It first retrieves the file from the UploadedFiles slice and then reads the file from the file system. Finally, it sends the file contents as a binary response.  
  
# core/http/endpoints/openai/files_test.go  
## Package: openai  
  
This package contains code for handling file uploads, listing files, and retrieving file content. It uses the Fiber framework for handling HTTP requests and the Zerolog library for logging.  
  
### Imports:  
  
- encoding/json  
- fmt  
- io  
- mime/multipart  
- net/http  
- net/http/httptest  
- os  
- path/filepath  
- strings  
  
- github.com/rs/zerolog/log  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/schema  
- github.com/gofiber/fiber/v2  
- github.com/mudler/LocalAI/pkg/utils  
- github.com/stretchr/testify/assert  
  
### External Data and Input Sources:  
  
- config.ApplicationConfig: This struct contains configuration settings for the application, such as the upload limit and upload directory.  
- config.BackendConfigLoader: This struct is responsible for loading backend configuration.  
  
### Code Summary:  
  
#### 1. startUpApp() function:  
  
This function sets up a test server using the Fiber framework. It creates a new Fiber app, sets up the upload limit, and defines routes for handling file uploads, listing files, retrieving file content, and deleting files.  
  
#### 2. TestUploadFileExceedSizeLimit() function:  
  
This function tests the UploadFilesEndpoint for various scenarios, including exceeding the upload limit, missing purpose, and file already existing. It uses the httptest package to create test requests and assert the expected responses.  
  
#### 3. CallListFilesEndpoint() function:  
  
This helper function creates a test request for listing files with or without a purpose parameter and returns the response.  
  
#### 4. CallFilesContentEndpoint() function:  
  
This helper function creates a test request for retrieving file content and returns the response.  
  
#### 5. CallFilesUploadEndpoint() and CallFilesUploadEndpointWithCleanup() functions:  
  
These functions create test requests for uploading files with different parameters and return the response. The CallFilesUploadEndpointWithCleanup() function also cleans up the uploaded file after the test.  
  
#### 6. CallFilesDeleteEndpoint() function:  
  
This helper function creates a test request for deleting a file and returns the response.  
  
#### 7. newMultipartFile() function:  
  
This helper function creates a multipart file for uploading, including the file content, tag, and purpose.  
  
#### 8. createTestFile() function:  
  
This helper function creates a test file with a specified size and name.  
  
#### 9. bodyToString() and bodyToByteArray() functions:  
  
These helper functions extract the response body as a string or byte array.  
  
#### 10. responseToFile() and responseToListFile() functions:  
  
These helper functions decode the response body into a schema.File or schema.ListFiles object.  
  
  
  
# core/http/endpoints/openai/image.go  
## openai package  
  
This package provides an endpoint for generating images using OpenAI's image generation API. It supports Stable Diffusion and TinyDream backends.  
  
### Imports  
  
```  
import (  
	"bufio"  
	"encoding/base64"  
	"encoding/json"  
	"fmt"  
	"io"  
	"net/http"  
	"os"  
	"path/filepath"  
	"strconv"  
	"strings"  
	"time"  
  
	"github.com/google/uuid"  
	"github.com/mudler/LocalAI/core/config"  
	"github.com/mudler/LocalAI/core/schema"  
  
	"github.com/mudler/LocalAI/core/backend"  
  
	"github.com/gofiber/fiber/v2"  
	model "github.com/mudler/LocalAI/pkg/model"  
	"github.com/rs/zerolog/log"  
)  
```  
  
### External Data, Input Sources  
  
The package uses the following external data and input sources:  
  
1. `config.BackendConfigLoader`: This is used to load the backend configuration.  
2. `model.ModelLoader`: This is used to load the model configuration.  
3. `config.ApplicationConfig`: This is used to load the application configuration.  
4. `schema.OpenAIRequest`: This is used to define the request parameters.  
5. `schema.OpenAIResponse`: This is used to define the response structure.  
  
### Code Summary  
  
1. `downloadFile(url string) (string, error)`: This function downloads a file from a given URL and saves it to a temporary file.  
  
2. `ImageEndpoint(cl *config.BackendConfigLoader, ml *model.ModelLoader, appConfig *config.ApplicationConfig) func(c *fiber.Ctx) error`: This function defines the API endpoint for image generation. It takes the backend configuration, model configuration, and application configuration as input.  
  
3. Inside the `ImageEndpoint` function, the following steps are performed:  
    - Read the request parameters from the incoming request.  
    - Load the model configuration based on the selected backend.  
    - Merge the request parameters with the model configuration.  
    - Download the input file if it's an URL, otherwise decode the base64 encoded file.  
    - Generate the images using the selected backend and model.  
    - Create a response object containing the generated images.  
    - Return the response as JSON.  
  
4. The code also includes error handling and logging for each step.  
  
5. The package uses the `fiber` framework for handling the API requests.  
  
6. The code is well-structured and follows best practices for writing Go code.  
  
# core/http/endpoints/openai/inference.go  
## Package: openai  
  
### Imports:  
  
- github.com/mudler/LocalAI/core/backend  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/schema  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
- req: schema.OpenAIRequest  
- predInput: string  
- config: config.BackendConfig  
- o: config.ApplicationConfig  
- loader: model.ModelLoader  
- cb: func(string, *[]schema.Choice)  
- tokenCallback: func(string, backend.TokenUsage) bool  
  
### ComputeChoices Function:  
  
The ComputeChoices function takes an OpenAI request, a prediction input, backend and application configurations, a model loader, a callback function, and a token callback function as input. It returns a list of choices, token usage, and an error.  
  
1. It initializes the number of completions to return (n) to 1 if it's 0.  
2. It extracts images, videos, and audios from the messages in the request.  
3. It calls the backend.ModelInference function to get the model function to call for the result.  
4. It iterates n times, calling the model function to get a prediction and appending the prediction to the result list.  
5. It updates the token usage for the prompt and completion based on the prediction.  
6. It calls the callback function with the finetuned response and the result list.  
7. Finally, it returns the result list, token usage, and any error encountered.  
  
# core/http/endpoints/openai/list.go  
## Package: openai  
  
### Imports:  
  
- github.com/gofiber/fiber/v2  
- github.com/mudler/LocalAI/core/config  
- github.com/mudler/LocalAI/core/schema  
- github.com/mudler/LocalAI/core/services  
- github.com/mudler/LocalAI/pkg/model  
  
### External Data, Input Sources:  
  
- Backend configuration loader (bcl)  
- Model loader (ml)  
- Query parameters: filter, excludeConfigured  
  
### ListModelsEndpoint:  
  
This function defines the OpenAI Models API endpoint, which lists and describes the available models. It takes a backend configuration loader (bcl) and a model loader (ml) as input.  
  
1. It first retrieves the filter parameter from the query parameters.  
2. It then determines the loose file policy based on the excludeConfigured query parameter.  
3. It builds a name filter function using the filter parameter and the loose file policy.  
4. It calls the ListModels function to retrieve a list of model names.  
5. It maps the list of model names to a slice of OpenAIModel response objects.  
6. Finally, it returns a JSON response containing the list of models.  
  
# core/http/endpoints/openai/request.go  
```  
package openai  
  
import (  
	"context"  
	"encoding/json"  
	"fmt"  
  
	"github.com/gofiber/fiber/v2"  
	"github.com/google/uuid"  
	"github.com/mudler/LocalAI/core/config"  
	fiberContext "github.com/mudler/LocalAI/core/http/ctx"  
	"github.com/mudler/LocalAI/core/schema"  
	"github.com/mudler/LocalAI/pkg/functions"  
	"github.com/mudler/LocalAI/pkg/model"  
	"github.com/mudler/LocalAI/pkg/templates"  
	"github.com/mudler/LocalAI/pkg/utils"  
	"github.com/rs/zerolog/log"  
)  
  
type correlationIDKeyType string  
  
// CorrelationIDKey to track request across process boundary  
const CorrelationIDKey correlationIDKeyType = "correlationID"  
  
func readRequest(c *fiber.Ctx, cl *config.BackendConfigLoader, ml *model.ModelLoader, o *config.ApplicationConfig, firstModel bool) (string, *schema.OpenAIRequest, error) {  
	input := new(schema.OpenAIRequest)  
  
	// Get input data from the request body  
	if err := c.BodyParser(input); err != nil {  
		return "", nil, fmt.Errorf("failed parsing request body: %w", err)  
	}  
  
	received, _ := json.Marshal(input)  
	// Extract or generate the correlation ID  
	correlationID := c.Get("X-Correlation-ID", uuid.New().String())  
  
	ctx, cancel := context.WithCancel(o.Context)  
	// Add the correlation ID to the new context  
	ctxWithCorrelationID := context.WithValue(ctx, CorrelationIDKey, correlationID)  
  
	input.Context = ctxWithCorrelationID  
	input.Cancel = cancel  
  
	log.Debug().Msgf("Request received: %s", string(received))  
  
	modelFile, err := fiberContext.ModelFromContext(c, cl, ml, input.Model, firstModel)  
  
	return modelFile, input, err  
}  
  
func updateRequestConfig(config *config.BackendConfig, input *schema.OpenAIRequest) {  
	if input.Echo {  
		config.Echo = input.Echo  
	}  
	if input.TopK != nil {  
		config.TopK = input.TopK  
	}  
	if input.TopP != nil {  
		config.TopP = input.TopP  
	}  
  
	if input.Backend != "" {  
		config.Backend = input.Backend  
	}  
  
	if input.ClipSkip != 0 {  
		config.Diffusers.ClipSkip = input.ClipSkip  
	}  
  
	if input.ModelBaseName != "" {  
		config.AutoGPTQ.ModelBaseName = input.ModelBaseName  
	}  
  
	if input.NegativePromptScale != 0 {  
		config.NegativePromptScale = input.NegativePromptScale  
	}  
  
	if input.UseFastTokenizer {  
		config.UseFastTokenizer = input.UseFastTokenizer  
	}  
  
	if input.NegativePrompt != "" {  
		config.NegativePrompt = input.NegativePrompt  
	}  
  
	if input.RopeFreqBase != 0 {  
		config.RopeFreqBase = input.RopeFreqBase  
	}  
  
	if input.RopeFreqScale != 0 {  
		config.RopeFreqScale = input.RopeFreqScale  
	}  
  
	if input.Grammar != "" {  
		config.Grammar = input.Grammar  
	}  
  
	if input.Temperature != nil {  
		config.Temperature = input.Temperature  
	}  
  
	if input.Maxtokens != nil {  
		config.Maxtokens = input.Maxtokens  
	}  
  
	if input.ResponseFormat != nil {  
		switch responseFormat := input.ResponseFormat.(type) {  
		case string:  
			config.ResponseFormat = responseFormat  
		case map[string]interface{}:  
			config.ResponseFormatMap = responseFormat  
		}  
	}  
  
	switch stop := input.Stop.(type) {  
	case string:  
		if stop != "" {  
			config.StopWords = append(config.StopWords, stop)  
		}  
	case []interface{}:  
		for _, pp := range stop {  
			if s, ok := pp.(string); ok {  
				config.StopWords = append(config.StopWords, s)  
			}  
		}  
	}  
  
	if len(input.Tools) > 0 {  
		for _, tool := range input.Tools {  
			input.Functions = append(input.Functions, tool.Function)  
		}  
	}  
  
	if input.ToolsChoice != nil {  
		var toolChoice functions.Tool  
  
		switch content := input.ToolsChoice.(type) {  
		case string:  
			_ = json.Unmarshal([]byte(content), &toolChoice)  
		case map[string]interface{}:  
			dat, _ := json.Marshal(content)  
			_ = json.Unmarshal(dat, &toolChoice)  
		}  
		input.FunctionCall = map[string]interface{}{  
			"name": toolChoice.Function.Name,  
		}  
	}  
  
	// Decode each request's message content  
	imgIndex, vidIndex, audioIndex := 0, 0, 0  
	for i, m := range input.Messages {  
		nrOfImgsInMessage := 0  
		nrOfVideosInMessage := 0  
		nrOfAudiosInMessage := 0  
  
		switch content := m.Content.(type) {  
		case string:  
			input.Messages[i].StringContent = content  
		case []interface{}:  
			dat, _ := json.Marshal(content)  
			c := []schema.Content{}  
			json.Unmarshal(dat, &c)  
  
			textContent := ""  
			// we will template this at the end  
  
		CONTENT:  
			for _, pp := range c {  
				switch pp.Type {  
				case "text":  
					textContent += pp.Text  
					//input.Messages[i].StringContent = pp.Text  
				case "video", "video_url":  
					// Decode content as base64 either if it's an URL or base64 text  
					base64, err := utils.GetContentURIAsBase64(pp.VideoURL.URL)  
					if err != nil {  
						log.Error().Msgf("Failed encoding video: %s", err)  
						continue CONTENT  
					}  
					input.Messages[i].StringVideos = append(input.Messages[i].StringVideos, base64) // TODO: make sure that we only return base64 stuff  
					vidIndex++  
					nrOfVideosInMessage++  
				case "audio_url", "audio":  
					// Decode content as base64 either if it's an URL or base64 text  
					base64, err := utils.GetContentURIAsBase64(pp.AudioURL.URL)  
					if err != nil {  
						log.Error().Msgf("Failed encoding image: %s", err)  
						continue CONTENT  
					}  
					input.Messages[i].StringAudios = append(input.Messages[i].StringAudios, base64) // TODO: make sure that we only return base64 stuff  
					audioIndex++  
					nrOfAudiosInMessage++  
				case "image_url", "image":  
					// Decode content as base64 either if it's an URL or base64 text  
					base64, err := utils.GetContentURIAsBase64(pp.ImageURL.URL)  
					if err != nil {  
						log.Error().Msgf("Failed encoding image: %s", err)  
						continue CONTENT  
					}  
  
					input.Messages[i].StringImages = append(input.Messages[i].StringImages, base64) // TODO: make sure that we only return base64 stuff  
  
					imgIndex++  
					nrOfImgsInMessage++  
				}  
			}  
  
			input.Messages[i].StringContent, _ = templates.TemplateMultiModal(config.TemplateConfig.Multimodal, templates.MultiModalOptions{  
				TotalImages:     imgIndex,  
				TotalVideos:     vidIndex,  
				TotalAudios:     audioIndex,  
				ImagesInMessage: nrOfImgsInMessage,  
				VideosInMessage: nrOfVideosInMessage,  
				AudiosInMessage: nrOfAudiosInMessage,  
			}, textContent)  
		}  
	}  
  
	if input.RepeatPenalty != 0 {  
		config.RepeatPenalty = input.RepeatPenalty  
	}  
  
	if input.FrequencyPenalty != 0 {  
		config.FrequencyPenalty = input.FrequencyPenalty  
	}  
  
	if input.PresencePenalty != 0 {  
		config.PresencePenalty = input.PresencePenalty  
	}  
  
	if input.Keep != 0 {  
		config.Keep = input.Keep  
	}  
  
	if input.Batch != 0 {  
		config.Batch = input.Batch  
	}  
  
	if input.IgnoreEOS {  
		config.IgnoreEOS = input.IgnoreEOS  
	}  
  
	if input.Seed != nil {  
		config.Seed = input.Seed  
	}  
  
	if input.TypicalP != nil {  
		config.TypicalP = input.TypicalP  
	}  
  
	switch inputs := input.Input.(type) {  
	case string:  
		if inputs != "" {  
			config.InputStrings = append(config.InputStrings, inputs)  
		}  
	case []interface{}:  
		for _, pp := range inputs {  
			switch i := pp.(type) {  
			case string:  
				config.InputStrings = append(config.InputStrings, i)  
			case []interface{}:  
				tokens := []int{}  
				for _, ii := range i {  
					tokens = append(tokens, int(ii.(float64)))  
				}  
				config.InputToken = append(config.InputToken, tokens)  
			}  
		}  
	}  
  
	// Can be either a string or an object  
	switch fnc := input.FunctionCall.(type) {  
	case string:  
		if fnc != "" {  
			config.SetFunctionCallString(fnc)  
		}  
	case map[string]interface{}:  
		var name string  
		n, exists := fnc["name"]  
		if exists {  
			nn, e := n.(string)  
			if e {  
				name = nn  
			}  
		}  
		config.SetFunctionCallNameString(name)  
	}  
  
	switch p := input.Prompt.(type) {  
	case string:  
		config.PromptStrings = append(config.PromptStrings, p)  
	case []interface{}:  
		for _, pp := range p {  
			if s, ok := pp.(string); ok {  
				config.PromptStrings = append(config.PromptStrings, s)  
			}  
		}  
	}  
}  
  
func mergeRequestWithConfig(modelFile string, input *schema.OpenAIRequest, cm *config.BackendConfigLoader, loader *model.ModelLoader, debug bool, threads, ctx int, f16 bool) (*config.BackendConfig, *schema.OpenAIRequest, error) {  
	cfg, err := cm.LoadBackendConfigFileByName(modelFile, loader.ModelPath,  
		config.LoadOptionDebug(debug),  
		config.LoadOptionThreads(threads),  
		config.LoadOptionContextSize(ctx),  
		config.LoadOptionF16(f16),  
		config.ModelPath(loader.ModelPath),  
	)  
  
	// Set the parameters for the language model prediction  
	updateRequestConfig(cfg, input)  
  
	if !cfg.Validate() {  
		return nil, nil, fmt.Errorf("failed to validate config")  
	}  
  
	return cfg, input, err  
}  
```  
  
# core/http/endpoints/openai/transcription.go  
## Package: openai  
  
### Imports:  
  
```  
fmt  
io  
net/http  
os  
path  
path/filepath  
github.com/mudler/LocalAI/core/backend  
github.com/mudler/LocalAI/core/config  
github.com/mudler/LocalAI/pkg/model  
github.com/gofiber/fiber/v2  
github.com/rs/zerolog/log  
```  
  
### External Data, Input Sources:  
  
1. Request parameters: model, file  
2. Configuration files: backend configuration, application configuration  
3. Model loader: model loader for loading pre-trained models  
  
### Summary:  
  
The code defines a function called `TranscriptEndpoint` that handles audio transcription requests. It takes a configuration loader, model loader, and application configuration as input. The function first reads the request parameters and merges them with the configuration. Then, it retrieves the audio file from the request and saves it to a temporary directory.  
  
The function then calls the `ModelTranscription` function from the `backend` package to perform the transcription. The `ModelTranscription` function takes the audio file, input language, translation flag, model loader, configuration, and application configuration as input. It returns the transcribed text.  
  
Finally, the function returns the transcribed text as a JSON response with a status code of 200.  
  
