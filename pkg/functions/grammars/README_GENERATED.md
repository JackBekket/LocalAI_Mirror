## grammars

### Imports
```
import (
	"testing"

	. "github.com/mudler/LocalAI/pkg/functions"

	. "github.com/onsi/ginkgo/v2"
	. "github.com/onsi/gomega"
)
```

### External Data, Input Sources
None

### TestGrammar
```
func TestGrammar(t *testing.T) {
	RegisterFailHandler(Fail)
	RunSpecs(t, "Grammar test suite")
}
```
This function is a test function that registers a fail handler and runs a test suite called "Grammar test suite".

### createFunction
```
func createFunction(field1 string, field2 string, name string, properties map[string]interface{}) map[string]interface{} {
	property := map[string]interface{}{}
	property[field1] = FunctionName{Const: name}
	property[field2] = Argument{
		Type:       "object",
		Properties: properties,
	}
	return property
}
```
This function creates a map of properties for a function. It takes four arguments: field1, field2, name, and properties. It returns a map containing the properties for the function.



pkg/functions/grammars/json_schema_test.go
## grammars_test

### Imports
```
package grammars_test

import (
	"strings"

	. "github.com/mudler/LocalAI/pkg/functions"
	. "github.com/mudler/LocalAI/pkg/functions/grammars"
	. "github.com/onsi/ginkgo/v2"
	. "github.com/onsi/gomega"
)

var testFunctions = []Item{
	{
		Type: "object",
		Properties: createFunction(
			"function",
			"arguments",
			"create_event",
			map[string]interface{}{
				"title": map[string]string{"type": "string"},
				"date":  map[string]string{"type": "string"},
				"time":  map[string]string{"type": "string"},
			},
		),
	},
	{
		Type: "object",
		Properties: createFunction(
			"function",
			"arguments",
			"search",
			map[string]interface{}{
				"query": map[string]string{"type": "string"},
			}),
	},
}

var testFunctionsName = []Item{
	{
		Type: "object",
		Properties: createFunction(
			"name",
			"arguments",
			"create_event",
			map[string]interface{}{
				"title": map[string]string{"type": "string"},
				"date":  map[string]string{"type": "string"},
				"time":  map[string]string{"type": "string"},
			},
		),
	},
	{
		Type: "object",
		Properties: createFunction(
			"name",
			"arguments",
			"search",
			map[string]interface{}{
				"query": map[string]string{"type": "string"},
			}),
	},
}

func rootResult(s string) string {
	return `root-0-name ::= "\"create_event\""
freestring ::= (
		[^"\\] |
		"\\" (["\\/bfnrt] | "u" [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F])
  )* space
root-0 ::= "{" space "\"arguments\"" space ":" space root-0-arguments "," space "\"name\"" space ":" space root-0-name "}" space
root-1-arguments ::= "{" space "\"query\"" space ":" space string "}" space
realvalue ::= root-0 | root-1
root ::= ` + s + `
space ::= " "?
root-0-arguments ::= "{" space "\"date\"" space ":" space string "," space "\"time\"" space ":" space string "," space "\"title\"" space ":" space string "}" space
root-1 ::= "{" space "\"arguments\"" space ":" space root-1-arguments "," space "\"name\"" space ":" space root-1-name "}" space
string ::= "\"" (
	[^"\\] |
	"\\" (["\\/bfnrt] | "u" [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F])
)* "\"" space
arr  ::=
"[\n"  (
	realvalue
    (",\n"  realvalue)*
  )? "]"
root-1-name ::= "\"search\""`
}

const (
	testInput1 = `
	{
		"oneOf": [
			{
				"type": "object",
				"properties": {
					"function": {"const": "create_event"},
					"arguments": {
						"type": "object",
						"properties": {
							"title": {"type": "string"},
							"date": {"type": "string"},
							"time": {"type": "string"}
						}
					}
				}
			},
			{
				"type": "object",
				"properties": {
					"function": {"const": "search"},
					"arguments": {
						"type": "object",
						"properties": {
							"query": {"type": "string"}
						}
					}
				}
			}
		]
	}`

	inputResult1 = `root-0-function ::= "\"create_event\""
freestring ::= (
		[^"\\] |
		"\\" (["\\/bfnrt] | "u" [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F])
  )* space
root-0 ::= "{" space "\"arguments\"" space ":" space root-0-arguments "," space "\"function\"" space ":" space root-0-function "}" space
root-1-arguments ::= "{" space "\"query\"" space ":" space string "}" space
realvalue ::= root-0 | root-1
root ::= arr | realvalue
space ::= " "?
root-0-arguments ::= "{" space "\"date\"" space ":" space string "," space "\"time\"" space ":" space string "," space "\"title\"" space ":" space string "}" space
root-1 ::= "{" space "\"arguments\"" space ":" space root-1-arguments "," space "\"function\"" space ":" space root-1-function "}" space
string ::= "\"" (
	[^"\\] |
	"\\" (["\\/bfnrt] | "u" [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F])
)* "\"" space
arr  ::=
"[\n"  (
	realvalue
    (",\n"  realvalue)*
  )? "]"
root-1-function ::= "\"search\""`

	testInput2 = `
{
	"oneOf": [
		{
			"type": "object",
			"properties": {
				"name": {"const": "create_event"},
				"arguments": {
					"type": "object",
					"properties": {
						"title": {"type": "string"},
						"date": {"type": "string"},
						"time": {"type": "string"}
					}
				}
			}
		},
		{
			"type": "object",
			"properties": {
				"name": {"const": "search"},
				"arguments": {
					"type": "object",
					"properties": {
						"query": {"type": "string"}
					}
				}
			}
		}
	]
}`

	inputResult2 = `root-0-name ::= "\"create_event\""
freestring ::= (
		[^"\\] |
		"\\" (["\\/bfnrt] | "u" [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F])
  )* space
root-0 ::= "{" space "\"arguments\"" space ":" space root-0-arguments "," space "\"name\"" space ":" space root-0-name "}" space
root-1-arguments ::= "{" space "\"query\"" space ":" space string "}" space
realvalue ::= root-0 | root-1
root ::= arr | realvalue
space ::= " "?
root-0-arguments ::= "{" space "\"date\"" space ":" space string "," space "\"time\"" space ":" space string "," space "\"title\"" space ":" space string "}" space
root-1 ::= "{" space "\"arguments\"" space ":" space root-1-arguments "," space "\"name\"" space ":" space root-1-name "}" space
string ::= "\"" (
	[^"\\] |
	"\\" (["\\/bfnrt] | "u" [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F] [0-9a-fA-F])
)* "\"" space
arr  ::=
"[\n"  (
	realvalue
    (",\n"  realvalue)*
  )? "]"
root-1-name ::= "\"search\""`
)

var _ = Describe("JSON schema grammar tests", func() {
	Context("JSON", func() {
		It("generates a valid grammar from JSON schema", func() {
			grammar, err := NewJSONSchemaConverter("").GrammarFromBytes([]byte(testInput1))
			Expect(err).To(BeNil())
			results := strings.Split(inputResult1, "\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))))
		})
		It("generates a valid grammar from JSON schema", func() {
			grammar, err := NewJSONSchemaConverter("").GrammarFromBytes([]byte(testInput2))
			Expect(err).To(BeNil())
			results := strings.Split(inputResult2, "\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))))
		})
		It("generates a valid grammar from JSON Objects", func() {

			structuredGrammar := JSONFunctionStructure{
				OneOf: testFunctions}

			grammar, err := structuredGrammar.Grammar()
			Expect(err).To(BeNil())
			results := strings.Split(inputResult1, "\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))))
		})

		It("generates a valid grammar from JSON Objects for multiple function return", func() {
			structuredGrammar := JSONFunctionStructure{
				OneOf: testFunctions}

			grammar, err := structuredGrammar.Grammar(EnableMaybeArray)
			Expect(err).To(BeNil())
			results := strings.Split(
				strings.Join([]string{
					inputResult2,
					"mixedstring ::= freestring | freestring arr | freestring realvalue"}, "\n"),
				"\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))), grammar)
		})

		It("generates a valid grammar from JSON Objects for multiple function return with a suffix and array", func() {
			structuredGrammar := JSONFunctionStructure{
				OneOf: testFunctionsName}

			grammar, err := structuredGrammar.Grammar(SetPrefix("suffix"), EnableMaybeArray)
			Expect(err).To(BeNil())
			results := strings.Split(
				strings.Join([]string{
					rootResult(`"suffix" arr | realvalue`),
					"mixedstring ::= freestring | freestring arr | freestring realvalue"}, "\n"),
				"\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))), grammar)
		})
		It("generates a valid grammar from JSON Objects with a suffix", func() {
			structuredGrammar := JSONFunctionStructure{
				OneOf: testFunctionsName}

			grammar, err := structuredGrammar.Grammar(SetPrefix("suffix"))
			Expect(err).To(BeNil())
			results := strings.Split(
				strings.Join([]string{
					rootResult(`"suffix" realvalue`),
					"mixedstring ::= freestring | freestring realvalue"}, "\n"),
				"\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))), grammar)
		})
		It("generates a valid grammar from JSON Objects with a suffix that could return text or an array of tools", func() {
			structuredGrammar := JSONFunctionStructure{
				OneOf: testFunctionsName}

			grammar, err := structuredGrammar.Grammar(SetPrefix("suffix"), EnableMaybeString, EnableMaybeArray)
			Expect(err).To(BeNil())
			results := strings.Split(
				strings.Join([]string{
					rootResult(`( "suffix" realvalue | mixedstring )`),
					"mixedstring ::= freestring | freestring realvalue"}, "\n"),
				"\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))), grammar)
		})

		It("generates a valid grammar from JSON Objects without a suffix that could return text or an array of tools or just string", func() {
			structuredGrammar := JSONFunctionStructure{
				OneOf: testFunctionsName}

			grammar, err := structuredGrammar.Grammar(EnableMaybeString, EnableMaybeArray)
			Expect(err).To(BeNil())
			results := strings.Split(
				strings.Join([]string{
					rootResult(`mixedstring | arr | realvalue`),
					"mixedstring ::= freestring | freestring arr | freestring realvalue"}, "\n"),
				"\n")
			for _, r := range results {
				if r != "" {
					Expect(grammar).To(ContainSubstring(r))
				}
			}
			Expect(len(results)).To(Equal(len(strings.Split(grammar, "\n"))), grammar)
		})

		It("generates a valid grammar from JSON Objects without

