# tests/e2e-aio/e2e_suite_test.go  
## Package: e2e_test  
  
### Imports:  
  
* context  
* fmt  
* os  
* runtime  
* testing  
* github.com/onsi/ginkgo/v2  
* github.com/onsi/gomega  
* github.com/ory/dockertest/v3  
* github.com/ory/dockertest/v3/docker  
* github.com/sashabaranov/go-openai  
  
### External Data, Input Sources:  
  
* LOCALAI_IMAGE: Container image to use for the local AI instance.  
* LOCALAI_IMAGE_TAG: Tag for the container image.  
* LOCALAI_MODELS_DIR: Directory containing the models to use with the local AI instance.  
* LOCALAI_API_PORT: Port on which the local AI instance will listen for API requests.  
* LOCALAI_API_ENDPOINT: Base URL for the local AI API.  
* LOCALAI_API_KEY: API key for the local AI instance.  
  
### Summary:  
  
#### TestLocalAI function:  
  
This function registers a fail handler and runs the E2E test suite for LocalAI.  
  
#### BeforeSuite function:  
  
This function sets up the environment for the tests. It first checks if the API port is provided, and if not, it defaults to 8080. Then, it creates an openai.ClientConfig object based on the provided API key and endpoint. If the endpoint is not provided, it starts a Docker container with the specified image and tag, and sets the API endpoint to the local address and port. Finally, it waits for the API to be ready by checking if the list models endpoint returns an error.  
  
#### AfterSuite function:  
  
This function cleans up the environment after the tests are completed. It purges the Docker container and logs the output.  
  
#### AfterEach function:  
  
This function clears the database after each test.  
  
#### startDockerImage function:  
  
This function starts a Docker container with the specified image and tag, mounts the models directory, and sets the environment variables for the container. It also sets the port bindings and environment variables for the container.  
  
# tests/e2e-aio/e2e_test.go  
```  
package e2e_test  
  
import (  
	"bytes"  
	"context"  
	"encoding/json"  
	"fmt"  
	"io"  
	"net/http"  
	"os"  
  
	"github.com/mudler/LocalAI/core/schema"  
	. "github.com/onsi/ginkgo/v2"  
	. "github.com/onsi/gomega"  
	"github.com/sashabaranov/go-openai"  
	"github.com/sashabaranov/go-openai/jsonschema"  
)  
  
var _ = Describe("E2E test", func() {  
	Context("Generating", func() {  
		BeforeEach(func() {  
			//  
		})  
  
		AfterEach(func() {  
			//  
		})  
  
		Context("text", func() {  
			It("correctly", func() {  
				model := "gpt-4"  
				resp, err := client.CreateChatCompletion(context.TODO(),  
					openai.ChatCompletionRequest{  
						Model: model, Messages: []openai.ChatCompletionMessage{  
							{  
								Role:    "user",  
								Content: "How much is 2+2?",  
							},  
						}})  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Choices)).To(Equal(1), fmt.Sprint(resp))  
				Expect(resp.Choices[0].Message.Content).To(Or(ContainSubstring("4"), ContainSubstring("four")), fmt.Sprint(resp.Choices[0].Message.Content))  
			})  
		})  
  
		Context("function calls", func() {  
			It("correctly invoke", func() {  
				params := jsonschema.Definition{  
					Type: jsonschema.Object,  
					Properties: map[string]jsonschema.Definition{  
						"location": {  
							Type:        jsonschema.String,  
							Description: "The city and state, e.g. San Francisco, CA",  
						},  
						"unit": {  
							Type: jsonschema.String,  
							Enum: []string{"celsius", "fahrenheit"},  
						},  
					},  
					Required: []string{"location"},  
				}  
  
				f := openai.FunctionDefinition{  
					Name:        "get_current_weather",  
					Description: "Get the current weather in a given location",  
					Parameters:  params,  
				}  
				t := openai.Tool{  
					Type:     openai.ToolTypeFunction,  
					Function: &f,  
				}  
  
				dialogue := []openai.ChatCompletionMessage{  
					{Role: openai.ChatMessageRoleUser, Content: "What is the weather in Boston today?"},  
				}  
				resp, err := client.CreateChatCompletion(context.TODO(),  
					openai.ChatCompletionRequest{  
						Model:    openai.GPT4,  
						Messages: dialogue,  
						Tools:    []openai.Tool{t},  
					},  
				)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Choices)).To(Equal(1), fmt.Sprint(resp))  
  
				msg := resp.Choices[0].Message  
				Expect(len(msg.ToolCalls)).To(Equal(1), fmt.Sprint(msg.ToolCalls))  
				Expect(msg.ToolCalls[0].Function.Name).To(Equal("get_current_weather"), fmt.Sprint(msg.ToolCalls[0].Function.Name))  
				Expect(msg.ToolCalls[0].Function.Arguments).To(ContainSubstring("Boston"), fmt.Sprint(msg.ToolCalls[0].Function.Arguments))  
			})  
		})  
		Context("json", func() {  
			It("correctly", func() {  
				model := "gpt-4"  
  
				req := openai.ChatCompletionRequest{  
					ResponseFormat: &openai.ChatCompletionResponseFormat{Type: openai.ChatCompletionResponseFormatTypeJSONObject},  
					Model:          model,  
					Messages: []openai.ChatCompletionMessage{  
						{  
  
							Role:    "user",  
							Content: "An animal with 'name', 'gender' and 'legs' fields",  
						},  
					},  
				}  
  
				resp, err := client.CreateChatCompletion(context.TODO(), req)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Choices)).To(Equal(1), fmt.Sprint(resp))  
  
				var i map[string]interface{}  
				err = json.Unmarshal([]byte(resp.Choices[0].Message.Content), &i)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(i).To(HaveKey("name"))  
				Expect(i).To(HaveKey("gender"))  
				Expect(i).To(HaveKey("legs"))  
			})  
		})  
  
		Context("images", func() {  
			It("correctly", func() {  
				resp, err := client.CreateImage(context.TODO(),  
					openai.ImageRequest{  
						Prompt: "test",  
						Size:   openai.CreateImageSize512x512,  
					},  
				)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Data)).To(Equal(1), fmt.Sprint(resp))  
				Expect(resp.Data[0].URL).To(ContainSubstring("png"), fmt.Sprint(resp.Data[0].URL))  
			})  
			It("correctly changes the response format to url", func() {  
				resp, err := client.CreateImage(context.TODO(),  
					openai.ImageRequest{  
						Prompt:         "test",  
						Size:           openai.CreateImageSize512x512,  
						ResponseFormat: openai.CreateImageResponseFormatURL,  
					},  
				)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Data)).To(Equal(1), fmt.Sprint(resp))  
				Expect(resp.Data[0].URL).To(ContainSubstring("png"), fmt.Sprint(resp.Data[0].URL))  
			})  
			It("correctly changes the response format to base64", func() {  
				resp, err := client.CreateImage(context.TODO(),  
					openai.ImageRequest{  
						Prompt:         "test",  
						Size:           openai.CreateImageSize512x512,  
						ResponseFormat: openai.CreateImageResponseFormatB64JSON,  
					},  
				)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Data)).To(Equal(1), fmt.Sprint(resp))  
				Expect(resp.Data[0].B64JSON).ToNot(BeEmpty(), fmt.Sprint(resp.Data[0].B64JSON))  
			})  
		})  
		Context("embeddings", func() {  
			It("correctly", func() {  
				resp, err := client.CreateEmbeddings(context.TODO(),  
					openai.EmbeddingRequestStrings{  
						Input: []string{"doc"},  
						Model: openai.AdaEmbeddingV2,  
					},  
				)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Data)).To(Equal(1), fmt.Sprint(resp))  
				Expect(resp.Data[0].Embedding).ToNot(BeEmpty())  
			})  
		})  
		Context("vision", func() {  
			It("correctly", func() {  
				model := "gpt-4o"  
				resp, err := client.CreateChatCompletion(context.TODO(),  
					openai.ChatCompletionRequest{  
						Model: model, Messages: []openai.ChatCompletionMessage{  
							{  
  
								Role: "user",  
								MultiContent: []openai.ChatMessagePart{  
									{  
										Type: openai.ChatMessagePartTypeText,  
										Text: "What is in the image?",  
									},  
									{  
										Type: openai.ChatMessagePartTypeImageURL,  
										ImageURL: &openai.ChatMessageImageURL{  
											URL:    "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",  
											Detail: openai.ImageURLDetailLow,  
										},  
									},  
								},  
							},  
						}})  
				Expect(err).ToNot(HaveOccurred())  
				Expect(len(resp.Choices)).To(Equal(1), fmt.Sprint(resp))  
				Expect(resp.Choices[0].Message.Content).To(Or(ContainSubstring("wooden"), ContainSubstring("grass")), fmt.Sprint(resp.Choices[0].Message.Content))  
			})  
		})  
		Context("text to audio", func() {  
			It("correctly", func() {  
				res, err := client.CreateSpeech(context.Background(), openai.CreateSpeechRequest{  
					Model: openai.TTSModel1,  
					Input: "Hello!",  
					Voice: openai.VoiceAlloy,  
				})  
				Expect(err).ToNot(HaveOccurred())  
				defer res.Close()  
  
				_, err = io.ReadAll(res)  
				Expect(err).ToNot(HaveOccurred())  
  
			})  
		})  
		Context("audio to text", func() {  
			It("correctly", func() {  
  
				downloadURL := "https://cdn.openai.com/whisper/draft-20220913a/micro-machines.wav"  
				file, err := downloadHttpFile(downloadURL)  
				Expect(err).ToNot(HaveOccurred())  
  
				req := openai.AudioRequest{  
					Model:    openai.Whisper1,  
					FilePath: file,  
				}  
				resp, err := client.CreateTranscription(context.Background(), req)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(resp.Text).To(ContainSubstring("This is the"), fmt.Sprint(resp.Text))  
			})  
		})  
  
		Context("reranker", func() {  
			It("correctly", func() {  
				modelName := "jina-reranker-v1-base-en"  
  
				req := schema.JINARerankRequest{  
					Model: modelName,  
					Query: "Organic skincare products for sensitive skin",  
					Documents: []string{  
						"Eco-friendly kitchenware for modern homes",  
						"Biodegradable cleaning supplies for eco-conscious consumers",  
						"Organic cotton baby clothes for sensitive skin",  
						"Natural organic skincare range for sensitive skin",  
						"Tech gadgets for smart homes: 2024 edition",  
						"Sustainable gardening tools and compost solutions",  
						"Sensitive skin-friendly facial cleansers and toners",  
						"Organic food wraps and storage solutions",  
						"All-natural pet food for dogs with allergies",  
						"Yoga mats made from recycled materials",  
					},  
					TopN: 3,  
				}  
  
				serialized, err := json.Marshal(req)  
				Expect(err).To(BeNil())  
				Expect(serialized).ToNot(BeNil())  
  
				rerankerEndpoint := apiEndpoint + "/rerank"  
				resp, err := http.Post(rerankerEndpoint, "application/json", bytes.NewReader(serialized))  
				Expect(err).To(BeNil())  
				Expect(resp).ToNot(BeNil())  
				body, err := io.ReadAll(resp.Body)  
				Expect(err).ToNot(HaveOccurred())  
				Expect(resp.StatusCode).To(Equal(200), fmt.Sprintf("body: %s, response: %+v", body, resp))  
  
				deserializedResponse := schema.JINARerankResponse{}  
				err = json.Unmarshal(body, &deserializedResponse)  
				Expect(err).To(BeNil())  
				Expect(deserializedResponse).ToNot(BeZero())  
				Expect(deserializedResponse.Model).To(Equal(modelName))  
				Expect(len(deserializedResponse.Results)).To(BeNumerically(">", 0))  
			})  
		})  
	})  
})  
  
func downloadHttpFile(url string) (string, error) {  
	resp, err := http.Get(url)  
	if err != nil {  
		return "", err  
	}  
	defer resp.Body.Close()  
  
	tmpfile, err := os.CreateTemp("", "example")  
	if err != nil {  
		return "", err  
	}  
	defer tmpfile.Close()  
  
	_, err = io.Copy(tmpfile, resp.Body)  
	if err != nil {  
		return "", err  
	}  
  
	return tmpfile.Name(), nil  
}  
```  
  
# tests/e2e/e2e_suite_test.go  
## Package: e2e_test  
  
### Imports:  
- os  
- testing  
- github.com/onsi/ginkgo/v2  
- github.com/onsi/gomega  
  
### External Data, Input Sources:  
- LOCALAI_API environment variable  
  
### Summary:  
#### TestLocalAI function:  
This function is the entry point for the LocalAI E2E test suite. It registers the Fail handler and runs the test suite using the RunSpecs function. The test suite is named "LocalAI E2E test suite".  
  
# tests/e2e/e2e_test.go  
## Package: e2e_test  
  
### Imports:  
  
* "context"  
* "fmt"  
* "os"  
* "os/exec"  
* ". "github.com/onsi/ginkgo/v2"  
* ". "github.com/onsi/gomega"  
* "github.com/otiai10/openaigo"  
* "github.com/sashabaranov/go-openai"  
  
### External Data, Input Sources:  
  
* localAIURL (environment variable)  
* BUILD_TYPE (environment variable)  
  
### Summary:  
  
#### Describe("E2E test", func() {  
  
This section describes an end-to-end test for the package. It initializes two clients, one for OpenAI and one for OpenAIGO, and sets up a context for the test.  
  
#### Context("API with ephemeral models", func() {  
  
This context focuses on testing the API with ephemeral models.  
  
#### BeforeEach(func() {  
  
This function is executed before each test in the context. It sets up the OpenAIGO client and waits for the API to be ready by checking if the models can be listed.  
  
#### AfterEach(func() {  
  
This function is executed after each test in the context. It checks if the GPU was used by examining the logs of the Docker container running the local AI tests.  
  
#### Context("Generates text", func() {  
  
This context focuses on testing the text generation capabilities of the API.  
  
#### It("streams chat tokens", func() {  
  
This test checks if the API can stream chat tokens correctly. It sends a chat completion request to the API with a model and a message, and then verifies that the response contains the expected content.  
  
  
  
