## Package: grammars

### Imports:

- encoding/json
- regexp

### External Data:

- PRIMITIVE_RULES: A map of primitive data types and their corresponding regular expressions.
- INVALID_RULE_CHARS_RE: A regular expression to match invalid characters in rule names.
- GRAMMAR_LITERAL_ESCAPE_RE: A regular expression to match literal escape characters in grammar literals.
- GRAMMAR_LITERAL_ESCAPES: A map of literal escape characters and their corresponding escaped representations.

### Code Summary:

The `grammars` package provides a set of functions and data structures for working with grammars. It includes a set of primitive rules for various data types, a regular expression to match invalid characters in rule names, and a regular expression and map for handling literal escape characters in grammar literals.

#### Primitive Rules:

The `PRIMITIVE_RULES` map defines regular expressions for various primitive data types, such as boolean, number, integer, string, and null. These regular expressions are used to parse and validate the input data.

#### Invalid Rule Characters:

The `INVALID_RULE_CHARS_RE` regular expression is used to match invalid characters in rule names. This helps ensure that the rule names are valid and can be used in the grammar.

#### Grammar Literal Escapes:

The `GRAMMAR_LITERAL_ESCAPE_RE` regular expression is used to match literal escape characters in grammar literals. The `GRAMMAR_LITERAL_ESCAPES` map provides the escaped representations of these characters, which are used to ensure that the grammar literals are correctly parsed.

#### Space Rule:

The `SPACE_RULE` constant defines a regular expression for a space character. This is used to ensure that there is a space between elements in the grammar.

#### Array Rules:

The `arrayNewLines` and `array` constants define regular expressions for arrays with and without newlines, respectively. These rules are used to parse and validate array data.

#### JSON String Function:

The `jsonString` function takes an interface{} as input and returns a string representation of the input data, along with an error if any occurs during the process. This function is used to convert data into a JSON string format.

pkg/functions/grammars/bnf_rules.go
## Package: grammars

### Imports:

- encoding/json
- regexp

### External Data:

- PRIMITIVE_RULES: A map of primitive data types and their corresponding regular expressions.
- INVALID_RULE_CHARS_RE: A regular expression to match invalid characters in rule names.
- GRAMMAR_LITERAL_ESCAPE_RE: A regular expression to match literal escape characters in grammar literals.
- GRAMMAR_LITERAL_ESCAPES: A map of literal escape characters and their corresponding escaped representations.

### Code Summary:

The `bnf_rules` file contains the implementation of the grammar rules for various data types and structures. It defines a set of functions and data structures that can be used to generate a grammar for a given schema.

#### Primitive Rules:

The `PRIMITIVE_RULES` map defines regular expressions for various primitive data types, such as boolean, number, integer, string, and null. These regular expressions are used to parse and validate the input data.

#### Invalid Rule Characters:

The `INVALID_RULE_CHARS_RE` regular expression is used to match invalid characters in rule names. This helps ensure that the rule names are valid and can be used in the grammar.

#### Grammar Literal Escapes:

The `GRAMMAR_LITERAL_ESCAPE_RE` regular expression is used to match literal escape characters in grammar literals. The `GRAMMAR_LITERAL_ESCAPES` map provides the escaped representations of these characters, which are used to ensure that the grammar literals are correctly parsed.

#### Space Rule:

The `SPACE_RULE` constant defines a regular expression for a space character. This is used to ensure that there is a space between elements in the grammar.

#### Array Rules:

The `arrayNewLines` and `array` constants define regular expressions for arrays with and without newlines, respectively. These rules are used to parse and validate array data.

#### JSON String Function:

The `jsonString` function takes an interface{} as input and returns a string representation of the input data, along with an error if any occurs during the process. This function is used to convert data into a JSON string format.

pkg/functions/grammars/json_schema.go
## Package: grammars

This package provides a JSON Schema converter that generates a grammar for a given JSON Schema.

### Imports:

- encoding/json
- fmt
- sort
- strings

### External Data, Input Sources:

- JSON Schema (map[string]interface{})
- JSON Schema bytes ( []byte)

### Code Summary:

1. **JSONSchemaConverter struct:**
   - Holds the property order and rules for the grammar generation.
   - Provides methods for formatting literals, adding rules, and visiting the schema.

2. **NewJSONSchemaConverter function:**
   - Creates a new JSONSchemaConverter instance with the given property order.
   - Initializes the rules map with the default space rule.

3. **formatLiteral method:**
   - Formats a given literal into a string representation suitable for the grammar.
   - Escapes special characters using the GRAMMAR_LITERAL_ESCAPE_RE and GRAMMAR_LITERAL_ESCAPES maps.

4. **addRule method:**
   - Adds a new rule to the rules map.
   - Handles potential conflicts by appending a unique identifier to the rule name.

5. **visit method:**
   - Recursively visits the schema and generates the grammar rules.
   - Handles different schema types, including oneOf, anyOf, $ref, const, enum, properties, and items.
   - Calls the appropriate methods for each schema type and recursively visits sub-schemas.

6. **resolveReference method:**
   - Resolves a reference to a definition in the schema.
   - Extracts the definition key from the reference and retrieves the definition from the $defs map.

7. **Grammar method:**
   - Generates the grammar for the given schema.
   - Calls the visit method to generate the rules and returns the grammar string.

8. **GrammarFromBytes method:**
   - Generates the grammar from a given byte array containing the JSON Schema.
   - Unmarshals the byte array into a map and calls the Grammar method.

pkg/functions/grammars/llama31_schema.go
## Package: grammars

This package provides a way to convert a JSON schema into a grammar. It uses a schema converter to visit the schema and generate a grammar based on the schema's structure.

### Imports:

- encoding/json
- fmt
- regexp
- sort
- strings

### External Data, Input Sources:

- JSON schema: The package expects a JSON schema as input, which can be provided as a map or a byte array.

### Code Summary:

1. **LLama31SchemaConverter struct:** This struct is responsible for converting a JSON schema into a grammar. It has two fields:
    - `fnName`: The name of the function to be used in the grammar.
    - `rules`: A map of rules that will be used to generate the grammar.

2. **NewLLama31SchemaConverter function:** This function creates a new instance of the LLama31SchemaConverter struct. It takes the function name as input and initializes the rules map with a default rule for the "space" type.

3. **GRAMMAR_LITERAL_ESCAPESLlama and GRAMMAR_LITERAL_ESCAPE_RELlama variables:** These variables define the literal escapes and the regular expression to escape literal characters in the grammar.

4. **formatLiteral and formatLiteralQuoted functions:** These functions format a given literal value into a string representation suitable for the grammar.

5. **addRule function:** This function adds a new rule to the rules map. It takes the rule name and the rule value as input and returns the key of the added rule.

6. **visit function:** This function recursively visits the schema and generates the grammar rules. It takes the schema, the current rule name, and the root schema as input and returns the generated rule and any errors encountered.

7. **resolveReference function:** This function resolves a reference to a definition in the schema. It takes the reference string and the root schema as input and returns the definition as a map and any errors encountered.

8. **Grammar and GrammarFromBytes functions:** These functions provide a way to generate the grammar from a given schema. The Grammar function takes the schema and a list of options as input, while the GrammarFromBytes function takes a byte array containing the schema and a list of options. Both functions return the generated grammar and any errors encountered.

pkg/functions/grammars/options.go
## Package: grammars

### Imports:

- `SchemaConverterType` (from a package, not provided in the code)

### External Data, Input Sources:

- `SchemaConverterType` (from a package, not provided in the code)

### GrammarOption Struct:

The `GrammarOption` struct is used to configure various aspects of a grammar. It has the following fields:

- `PropOrder`: String representing the order of properties in the output.
- `Prefix`: String representing the prefix to be added to the output.
- `MaybeArray`: Boolean indicating whether the output should be a list or array.
- `DisableParallelNewLines`: Boolean indicating whether to disable parallel new lines in the output.
- `MaybeString`: Boolean indicating whether the output should be a string.
- `NoMixedFreeString`: Boolean indicating whether to disable mixed free strings in the output.
- `ExpectStringsAfterJSON`: Boolean indicating whether to expect strings after JSON in the output.
- `FunctionName`: String representing the function name to be used in the output.
- `SchemaType`: `SchemaConverterType` representing the schema type to be used in the output.

### Apply Method:

The `Apply` method takes a variable number of functions as input and applies them to the `GrammarOption` struct. This allows for chaining of configuration options.

### Helper Functions:

The code provides several helper functions to set specific configuration options for the `GrammarOption` struct:

- `EnableMaybeArray`: Sets the `MaybeArray` field to true.
- `DisableParallelNewLines`: Sets the `DisableParallelNewLines` field to true.
- `EnableMaybeString`: Sets the `MaybeString` field to true.
- `NoMixedFreeString`: Sets the `NoMixedFreeString` field to true.
- `ExpectStringsAfterJSON`: Sets the `ExpectStringsAfterJSON` field to true.
- `SetPrefix`: Sets the `Prefix` field to the provided suffix.
- `SetPropOrder`: Sets the `PropOrder` field to the provided order.
- `WithSchemaType`: Sets the `SchemaType` field to the provided schema type.
- `WithFunctionName`: Sets the `FunctionName` field to the provided name.

### Summary:

The `grammars` package provides a `GrammarOption` struct and helper functions to configure various aspects of a grammar. The `GrammarOption` struct can be customized using the provided helper functions, allowing for flexible configuration of the grammar. The package also includes an `Apply` method to chain configuration options together.

pkg/functions/grammars/rules.go
## Package: grammars

### Imports:

- fmt
- strings
- github.com/mudler/LocalAI/pkg/utils

### External Data, Input Sources:

- Rules: A map of strings representing grammar rules.

### Summary:

The `grammars` package provides a function called `ToGrammar` that takes a map of grammar rules and an optional set of options to customize the output grammar. The function returns a string representation of the grammar.

The `ToGrammar` function first applies the provided options to a `GrammarOption` struct. Then, it iterates through the provided rules and constructs a list of strings representing the grammar rules. The function also handles special cases for the root rule, array rules, and free string rules based on the provided options. Finally, it joins the list of strings with newline characters and returns the resulting grammar string.

pkg/functions/grammars/types.go
## Package: grammars

### Imports:

None

### External Data and Input Sources:

None

### SchemaConverterType

The `SchemaConverterType` is an integer type that represents the type of schema converter. It has two possible values: `JSONSchema` and `LLama31Schema`.

### Constants

The package defines two constants:

- `JSONType`: A string representing the JSON schema type.
- `LlamaType`: A string representing the Llama 3.1 schema type.

### String() Method

The `String()` method is implemented for the `SchemaConverterType` type. It returns a string representation of the schema converter type based on the value of the `SchemaConverterType` variable.

### NewType() Function

The `NewType()` function takes a string argument `t` and returns a `SchemaConverterType` value based on the input string. It supports two input types: `JSONType` and `LlamaType`. If the input string is not recognized, it returns the default value, which is `JSONSchema`.

