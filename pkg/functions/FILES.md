# pkg/functions/function_structure.go  
## Package: functions  
  
### Imports:  
- encoding/json  
- github.com/mudler/LocalAI/pkg/functions/grammars  
  
### External Data, Input Sources:  
- JSONFunctionStructure: A struct that represents a JSON function structure. It has three fields: OneOf, AnyOf, and Defs.  
- grammars.GrammarOption: An option struct that can be used to configure the grammar generation process.  
  
### Code Summary:  
#### JSONFunctionStructure:  
- This struct represents a JSON function structure. It has three fields: OneOf, AnyOf, and Defs. These fields can be used to define the structure of the function.  
  
#### Grammar() function:  
- This function takes a JSONFunctionStructure and a list of options as input. It then converts the JSONFunctionStructure to a string representation of the grammar.  
- The function first applies the given options to a grammars.GrammarOption struct.  
- Then, it marshals the JSONFunctionStructure to a byte array using the json.Marshal() function.  
- Next, it creates a SchemaConverter instance based on the given options.  
- Finally, it calls the GrammarFromBytes() method of the SchemaConverter to generate the grammar string.  
  
#### SchemaConverter interface:  
- This interface defines a method called GrammarFromBytes() that takes a byte array and a list of options as input and returns a string representation of the grammar.  
  
#### NewSchemaConverter() function:  
- This function creates a new instance of the SchemaConverter based on the given options.  
- It checks the value of the SchemaType field in the options and returns a corresponding converter.  
- If the SchemaType is grammars.LLama31Schema, it returns a grammars.NewLLama31SchemaConverter instance.  
- Otherwise, it returns a grammars.NewJSONSchemaConverter instance.  
  
  
  
# pkg/functions/functions.go  
## Package: functions  
  
### Imports:  
  
- encoding/json  
- github.com/rs/zerolog/log  
  
### External Data, Input Sources:  
  
- None  
  
### Function: ToJSONStructure  
  
This function takes a list of functions and converts it into a JSON structure that can be parsed by a large language model (LLM). The structure is designed to allow the LLM to return a response in the format: { "name": "function_name", "arguments": { "arg1": "value1", "arg2": "value2" } }.  
  
The function first checks if the provided name and arguments are not empty. If they are not empty, it uses the provided values as the keys for the name and arguments fields in the JSON structure. Otherwise, it uses the default keys "name" and "arguments".  
  
The function then iterates through the list of functions and creates a map of properties and definitions for each function. It uses the json.Marshal and json.Unmarshal functions to convert the properties and definitions to and from JSON format.  
  
Finally, the function creates a JSONFunctionStructure object and populates it with the properties and definitions for each function. The function returns the JSONFunctionStructure object.  
  
### Function: Select  
  
This function takes a list of functions and a function name as input. It returns a new list of functions containing only the function with the given name.  
  
The function iterates through the list of functions and checks if the name of the current function matches the provided function name. If it does, the function is added to a new list, and the loop breaks.  
  
Finally, the function returns the new list of functions containing only the function with the given name.  
  
# pkg/functions/json_mode.go  
package functions  
  
imports:  
- JSONBNF  
  
external data:  
- JSONBNF  
  
summary:  
This package defines a function to parse JSON data. The function takes a string as input and returns a parsed JSON object. The JSONBNF constant contains a regular expression that defines the syntax of the JSON data. The function uses this regular expression to parse the input string and return the parsed JSON object.  
  
# pkg/functions/parse.go  
```  
package functions  
  
import (  
	"encoding/json"  
	"errors"  
	"io"  
	"regexp"  
	"strings"  
  
	"github.com/mudler/LocalAI/pkg/functions/grammars"  
	"github.com/mudler/LocalAI/pkg/utils"  
	"github.com/rs/zerolog/log"  
)  
  
type GrammarConfig struct {  
	// ParallelCalls enables the LLM to return multiple function calls in the same response  
	ParallelCalls bool `yaml:"parallel_calls"`  
  
	DisableParallelNewLines bool `yaml:"disable_parallel_new_lines"`  
  
	// MixedMode enables the LLM to return strings and not only JSON objects  
	// This is useful for models to not constraing returning only JSON and also messages back to the user  
	MixedMode bool `yaml:"mixed_mode"`  
  
	// NoMixedFreeString disables the mixed mode for free strings  
	// In this way if the LLM selects a free string, it won't be mixed necessarly with JSON objects.  
	// For example, if enabled the LLM or returns a JSON object or a free string, but not a mix of both  
	// If disabled(default): the LLM can return a JSON object surrounded by free strings (e.g. `this is the JSON result: { "bar": "baz" } for your question`). This forces the LLM to return at least a JSON object, but its not going to be strict  
	NoMixedFreeString bool `yaml:"no_mixed_free_string"`  
  
	// NoGrammar disables the grammar parsing and parses the responses directly from the LLM  
	NoGrammar bool `yaml:"disable"`  
  
	// Prefix is the suffix to append to the grammar when being generated  
	// This is useful when models prepend a tag before returning JSON  
	Prefix string `yaml:"prefix"`  
  
	// ExpectStringsAfterJSON enables mixed string suffix  
	ExpectStringsAfterJSON bool `yaml:"expect_strings_after_json"`  
  
	// PropOrder selects what order to print properties  
	// for instance name,arguments will make print { "name": "foo", "arguments": { "bar": "baz" } }  
	// instead of { "arguments": { "bar": "baz" }, "name": "foo" }  
	PropOrder string `yaml:"properties_order"`  
  
	// SchemaType can be configured to use a specific schema type to force the grammar  
	// available : json, llama3.1  
	SchemaType string `yaml:"schema_type"`  
}  
  
// FunctionsConfig is the configuration for the tool/function call.  
// It includes setting to map the function name and arguments from the response  
// and, for instance, also if processing the requests with BNF grammars.  
type FunctionsConfig struct {  
	// DisableNoAction disables the "no action" tool  
	// By default we inject a tool that does nothing and is used to return an answer from the LLM  
	DisableNoAction bool `yaml:"disable_no_action"`  
  
	// Grammar is the configuration for the grammar  
	GrammarConfig GrammarConfig `yaml:"grammar"`  
  
	// NoActionFunctionName is the name of the function that does nothing. It defaults to "answer"  
	NoActionFunctionName string `yaml:"no_action_function_name"`  
  
	// NoActionDescriptionName is the name of the function that returns the description of the no action function  
	NoActionDescriptionName string `yaml:"no_action_description_name"`  
  
	// ResponseRegex is a named regex to extract the function name and arguments from the response  
	ResponseRegex []string `yaml:"response_regex"`  
  
	// JSONRegexMatch is a regex to extract the JSON object from the response  
	JSONRegexMatch []string `yaml:"json_regex_match"`  
  
	// ReplaceFunctionResults allow to replace strings in the results before parsing them  
	ReplaceFunctionResults []ReplaceResult `yaml:"replace_function_results"`  
  
	// ReplaceLLMResult allow to replace strings in the results before parsing them  
	ReplaceLLMResult []ReplaceResult `yaml:"replace_llm_results"`  
  
	// CaptureLLMResult is a regex to extract a string from the LLM response  
	// that is used as return string when using tools.  
	// This is useful for e.g. if the LLM outputs a reasoning and we want to get the reasoning as a string back  
	CaptureLLMResult []string `yaml:"capture_llm_results"`  
  
	// FunctionNameKey enable the LLM to return { "name": "function_name", "arguments": { "arg1": "value1", "arg2": "value2" } }  
	// instead of { "function": "function_name", "arguments": { "arg1": "value1", "arg2": "value2" } }.  
	// This might be useful for certain models trained with the function name as the first token.  
	FunctionNameKey      string `yaml:"function_name_key"`  
	FunctionArgumentsKey string `yaml:"function_arguments_key"`  
}  
  
type ReplaceResult struct {  
	Key   string `yaml:"key"`  
	Value string `yaml:"value"`  
}  
  
type FuncCallResults struct {  
	Name      string  
	Arguments string  
}  
  
func (g FunctionsConfig) GrammarOptions() []func(o *grammars.GrammarOption) {  
	opts := []func(o *grammars.GrammarOption){}  
	if g.GrammarConfig.MixedMode {  
		opts = append(opts, grammars.EnableMaybeString)  
	}  
	if g.GrammarConfig.ParallelCalls {  
		opts = append(opts, grammars.EnableMaybeArray)  
	}  
	if g.GrammarConfig.DisableParallelNewLines {  
		opts = append(opts, grammars.DisableParallelNewLines)  
	}  
	if g.GrammarConfig.Prefix != "" {  
		opts = append(opts, grammars.SetPrefix(g.GrammarConfig.Prefix))  
	}  
	if g.GrammarConfig.NoMixedFreeString {  
		opts = append(opts, grammars.NoMixedFreeString)  
	}  
	if g.GrammarConfig.ExpectStringsAfterJSON {  
		opts = append(opts, grammars.ExpectStringsAfterJSON)  
	}  
  
	if g.GrammarConfig.SchemaType != "" {  
		opts = append(opts, grammars.WithSchemaType(grammars.NewType(g.GrammarConfig.SchemaType)))  
	}  
  
	if g.FunctionNameKey != "" {  
		opts = append(opts, grammars.WithFunctionName(g.FunctionNameKey))  
	}  
  
	opts = append(opts, grammars.SetPropOrder(g.GrammarConfig.PropOrder))  
	return opts  
}  
  
func CleanupLLMResult(llmresult string, functionConfig FunctionsConfig) string {  
	log.Debug().Msgf("LLM result: %s", llmresult)  
  
	for _, item := range functionConfig.ReplaceLLMResult {  
		k, v := item.Key, item.Value  
		log.Debug().Msgf("Replacing %s with %s", k, v)  
		re := regexp.MustCompile(k)  
		llmresult = re.ReplaceAllString(llmresult, v)  
	}  
	log.Debug().Msgf("LLM result(processed): %s", llmresult)  
  
	return llmresult  
}  
  
func ParseTextContent(llmresult string, functionConfig FunctionsConfig) string {  
	log.Debug().Msgf("ParseTextContent: %s", llmresult)  
	log.Debug().Msgf("CaptureLLMResult: %s", functionConfig.CaptureLLMResult)  
  
	for _, r := range functionConfig.CaptureLLMResult {  
		// We use a regex to extract the JSON object from the response  
		var respRegex = regexp.MustCompile(r)  
		match := respRegex.FindStringSubmatch(llmresult)  
		if len(match) >= 1 {  
			m := strings.TrimSpace(match[1])  
			return m  
		}  
	}  
  
	return ""  
}  
  
// ParseJSON is a function that parses a JSON string that might contain multiple JSON objects  
// and syntax errors in between by shifting the offset  
// This for e.g. allow to parse  
// { "foo": "bar" } invalid { "baz": "qux" }  
// into  
// [ { "foo": "bar" }, { "baz": "qux" } ]  
// Credits to Michael Yang (https://github.com/mxyng) for the original implementation  
// This is a slighly reworked version, improved for readability and error handling  
func ParseJSON(s string) ([]map[string]any, error) {  
	var objs []map[string]any  
	offset := 0  
  
	for offset < len(s) {  
		var obj map[string]any  
		decoder := json.NewDecoder(strings.NewReader(s[offset:]))  
  
		err := decoder.Decode(&obj)  
		switch {  
		case errors.Is(err, io.EOF):  
			return objs, nil  
		case err == nil:  
			offset += int(decoder.InputOffset())  
			objs = append(objs, obj)  
		default: // handle the error type  
			var syntaxErr *json.SyntaxError  
			var unmarshalTypeErr *json.UnmarshalTypeError  
  
			switch {  
			case errors.As(err, &syntaxErr):  
				offset += int(syntaxErr.Offset)  
			case errors.As(err, &unmarshalTypeErr):  
				offset += int(unmarshalTypeErr.Offset)  
			default:  
				return objs, err  
			}  
		}  
	}  
  
	return objs, nil  
}  
  
func ParseFunctionCall(llmresult string, functionConfig FunctionsConfig) []FuncCallResults {  
  
	log.Debug().Msgf("LLM result: %s", llmresult)  
  
	for _, item := range functionConfig.ReplaceFunctionResults {  
		k, v := item.Key, item.Value  
		log.Debug().Msgf("Replacing %s with %s", k, v)  
		re := regexp.MustCompile(k)  
		llmresult = re.ReplaceAllString(llmresult, v)  
	}  
	log.Debug().Msgf("LLM result(function cleanup): %s", llmresult)  
  
	functionNameKey := defaultFunctionNameKey  
	functionArgumentsKey := defaultFunctionArgumentsKey  
	if functionConfig.FunctionNameKey != "" {  
		functionNameKey = functionConfig.FunctionNameKey  
	}  
	if functionConfig.FunctionArgumentsKey != "" {  
		functionArgumentsKey = functionConfig.FunctionArgumentsKey  
	}  
  
	results := []FuncCallResults{}  
	llmResults := []string{}  
  
	returnResult := func(results []string) (result []FuncCallResults, e error) {  
		// As we have to change the result before processing, we can't stream the answer token-by-token (yet?)  
		result = make([]FuncCallResults, 0)  
  
		for _, s := range results {  
			var ss []map[string]any  
  
			s = utils.EscapeNewLines(s)  
			ss, err := ParseJSON(s)  
			//err := json.Unmarshal([]byte(s), &ss)  
			if err != nil {  
				log.Debug().Err(err).Str("escapedLLMResult", s).Msg("unable to unmarshal llm result in a single object or an array of JSON objects")  
			}  
  
			log.Debug().Msgf("Function return: %s %+v", s, ss)  
  
			for _, s := range ss {  
				// The grammar defines the function name as "function", while OpenAI returns "name"  
				func_name, ok := s[functionNameKey]  
				if !ok {  
					continue  
					//return result, fmt.Errorf("unable to find function name in result")  
				}  
				// Similarly, while here arguments is a map[string]interface{}, OpenAI actually want a stringified object  
				args, ok := s[functionArgumentsKey] // arguments needs to be a string, but we return an object from the grammar result (TODO: fix)  
				if !ok {  
					continue  
					//return result, fmt.Errorf("unable to find arguments in result")  
				}  
				d, _ := json.Marshal(args)  
				funcName, ok := func_name.(string)  
				if !ok {  
					continue  
					//return result, fmt.Errorf("unable to cast function name to string")  
				}  
  
				result = append(result, FuncCallResults{Name: funcName, Arguments: string(d)})  
			}  
		}  
  
		return result, nil  
	}  
  
	// the response is a string that we have to parse  
	result := make(map[string]string)  
	if len(functionConfig.JSONRegexMatch) != 0 {  
		for _, r := range functionConfig.JSONRegexMatch {  
			// We use a regex to extract the JSON object from the response  
			var respRegex = regexp.MustCompile(r)  
			match := respRegex.FindAllStringSubmatch(llmresult, -1)  
			var allMatches []string  
			for _, m := range match {  
				if len(m) > 1 {  
					// we match the first group  
					allMatches = append(allMatches, m[1])  
				}  
			}  
			if len(allMatches) > 0 {  
				llmResults = append(llmResults, allMatches...)  
				break  
			}  
		}  
	}  
  
	if len(functionConfig.ResponseRegex) > 0 {  
		// We use named regexes here to extract the function name and arguments  
		// obviously, this expects the LLM to be stable and return correctly formatted JSON  
		// TODO: optimize this and pre-compile it  
		for _, r := range functionConfig.ResponseRegex {  
			var respRegex = regexp.MustCompile(r)  
			matches := respRegex.FindAllStringSubmatch(llmresult, -1)  
			for _, match := range matches {  
				for i, name := range respRegex.SubexpNames() {  
					if i != 0 && name != "" && len(match) > i {  
						result[name] = match[i]  
					}  
				}  
  
				functionName := result[functionNameKey]  
				if functionName == "" {  
					return results  
				}  
				results = append(results, FuncCallResults{Name: result[functionNameKey], Arguments: result[functionArgumentsKey]})  
			}  
		}  
	} else {  
		if len(llmResults) == 0 {  
			llmResults = append(llmResults, llmresult)  
		}  
		results, _ = returnResult(llmResults)  
	}  
  
	return results  
}  
```  
  
