# 📞 Function Calling — From Natural Language to Structured JSON

> **What this doc covers:** What function calling is in the context of this project, how `functions_definition.json` works, how the model decides which function to call, and what the full input → output flow looks like with real examples.

---

## Table of Contents

1. [The Core Idea](#the-core-idea)
2. [functions_definition.json — The Schema File](#functions_definitionjson--the-schema-file)
3. [function_calling_tests.json — The Input File](#function_calling_testsjson--the-input-file)
4. [How the Model Chooses the Right Function](#how-the-model-chooses-the-right-function)
5. [The Output Format](#the-output-format)
6. [End-to-End Examples](#end-to-end-examples)
7. [Type System](#type-system)
8. [Edge Cases to Handle](#edge-cases-to-handle)
9. [What This Project Does NOT Do](#what-this-project-does-not-do)

---

## The Core Idea

A traditional LLM answers a question with prose:

```
User:  "What is the sum of 40 and 2?"
Model: "The sum of 40 and 2 is 42."
```

That's great for humans. It's useless for software.

**Function calling** changes the goal entirely. Instead of answering the question, the model produces a structured JSON object that tells your code *what to execute* and *with what arguments*:

```json
{
  "name": "fn_add_numbers",
  "parameters": {"a": 40.0, "b": 2.0}
}
```

Your Python code then takes that JSON and can call the actual function. The model is acting as a **natural language → function call translator**.

```
Natural Language  →  LLM  →  Structured JSON  →  Your Code  →  Result
"sum of 40 and 2"     →      fn_add_numbers(40, 2)     →      42.0
```

---

## `functions_definition.json` — The Schema File

This file tells the model (and your constrained decoder) what functions exist, what arguments they take, and what types those arguments must be.

### Structure

It's a JSON array. Each element describes one available function:

```json
[
  {
    "name": "fn_add_numbers",
    "description": "Add two numbers together and return their sum.",
    "parameters": {
      "a": {"type": "number"},
      "b": {"type": "number"}
    },
    "returns": {
      "type": "number"
    }
  },
  {
    "name": "fn_greet",
    "description": "Generate a greeting message for a person by name.",
    "parameters": {
      "name": {"type": "string"}
    },
    "returns": {
      "type": "string"
    }
  },
  {
    "name": "fn_reverse_string",
    "description": "Reverse a string and return the reversed result.",
    "parameters": {
      "s": {"type": "string"}
    },
    "returns": {
      "type": "string"
    }
  }
]
```

### Fields explained

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Exact function name — must appear verbatim in output |
| `description` | string | Human-readable explanation — used in the prompt to help the model understand intent |
| `parameters` | object | Map of argument name → `{"type": ...}` |
| `returns` | object | Return type of the function (informational) |

### How your code should load this

```python
import json
from pydantic import BaseModel
from typing import Dict

class ParameterSchema(BaseModel):
    type: str

class FunctionDefinition(BaseModel):
    name: str
    description: str
    parameters: Dict[str, ParameterSchema]
    returns: ParameterSchema

def load_functions(path: str) -> list[FunctionDefinition]:
    with open(path) as f:
        data = json.load(f)
    return [FunctionDefinition(**fn) for fn in data]
```

> **Always validate with pydantic.** The subject requires all classes use pydantic for validation, and the input files may be malformed. Wrap your load logic in try/except.

---

## `function_calling_tests.json` — The Input File

This file contains the natural language prompts your system must process. It's a JSON array of objects, each with a single `"prompt"` key:

```json
[
  {"prompt": "What is the sum of 2 and 3?"},
  {"prompt": "What is the sum of 265 and 345?"},
  {"prompt": "Greet shrek"},
  {"prompt": "Greet john"},
  {"prompt": "Reverse the string 'hello'"},
  {"prompt": "Add 1000 and 0.5"},
  {"prompt": "Say hello to Alice"}
]
```

### What to expect from real test files

The subject warns that the actual test files used during peer review may differ from the examples. Your solution must handle:

- Different phrasing for the same intent (`"sum of"`, `"add"`, `"plus"`, `"total of"`)
- Different capitalisation (`"Greet John"`, `"greet john"`, `"GREET JOHN"`)
- Numbers as words or digits (`"five"` vs `"5"`) — the model should handle semantics
- Prompts with extra context (`"Can you please reverse the string 'world' for me?"`)
- Ambiguous prompts where multiple functions could apply

---

## How the Model Chooses the Right Function

This is where the LLM does the actual "thinking." The model reads the prompt and the function descriptions, and assigns higher probability to the function name that semantically matches.

### The prompt you give the model

You need to construct a prompt that includes enough context for the model to understand what functions are available. A typical structure:

```
You are a function calling assistant. Given a user request, output the 
correct function call as JSON.

Available functions:
- fn_add_numbers(a: number, b: number): Add two numbers together and return their sum.
- fn_greet(name: string): Generate a greeting message for a person by name.
- fn_reverse_string(s: string): Reverse a string and return the reversed result.

User request: "Greet john"

Output the function call as JSON:
{"name": "
```

Notice the prompt **ends mid-JSON** with `{"name": "`. This is intentional — you prime the model to continue generating inside a JSON structure, and then constrained decoding takes over to guarantee the rest is valid.

### Why the model picks the right function

The model has learned from training data that:
- `"sum"`, `"add"`, `"plus"` → arithmetic functions
- `"greet"`, `"hello"`, `"say hi to"` → greeting functions
- `"reverse"` → string reversal

When constrained decoding restricts the `"name"` field to only valid function names, the model's learned semantic associations mean it assigns the highest probability to the correct one.

```
Prompt: "Greet john"

At the name field, valid token choices are:
  "fn_add_numbers"   → logit: -2.3   (low — no math intent in prompt)
  "fn_greet"         → logit:  4.1   (high — "greet" matches directly)
  "fn_reverse_string"→ logit: -1.8   (low — no reversal intent)

Winner: "fn_greet" ✅
```

---

## The Output Format

Your program writes a single file: `data/output/function_calling_results.json`.

It's a JSON array. Each element corresponds to one input prompt, in the same order:

```json
[
  {
    "prompt": "What is the sum of 2 and 3?",
    "name": "fn_add_numbers",
    "parameters": {"a": 2.0, "b": 3.0}
  },
  {
    "prompt": "Greet john",
    "name": "fn_greet",
    "parameters": {"name": "john"}
  },
  {
    "prompt": "Reverse the string 'hello'",
    "name": "fn_reverse_string",
    "parameters": {"s": "hello"}
  }
]
```

### Validation rules (from the subject)

- ✅ Must be valid JSON (no trailing commas, no comments)
- ✅ `name` must exactly match a function name from `functions_definition.json`
- ✅ `parameters` keys must exactly match the function's parameter names
- ✅ Parameter types must match the schema (`number` → float, `string` → string, etc.)
- ✅ All required parameters must be present
- ❌ No extra keys allowed
- ❌ No prose, explanations, or markdown — pure JSON only

### Pydantic model for output validation

```python
from pydantic import BaseModel
from typing import Any, Dict

class FunctionCall(BaseModel):
    prompt: str
    name: str
    parameters: Dict[str, Any]

    def validate_against_schema(self, fn_def: FunctionDefinition) -> bool:
        """Check that parameters match the function definition schema."""
        if self.name != fn_def.name:
            return False
        for param_name, param_schema in fn_def.parameters.items():
            if param_name not in self.parameters:
                return False
            value = self.parameters[param_name]
            if param_schema.type == "number" and not isinstance(value, (int, float)):
                return False
            if param_schema.type == "string" and not isinstance(value, str):
                return False
        return True
```

---

## End-to-End Examples

### Example 1: Arithmetic

```
Input prompt:   "What is the sum of 265 and 345?"

Model reads:    fn_add_numbers(a: number, b: number) — "Add two numbers"
Model decides:  fn_add_numbers, a=265, b=345

Constrained generation:
  "{"          ← only '{' allowed at start
  "{"n"        ← only '"' then 'n' continue valid JSON key "name"
  ...
  "fn_add_numbers"  ← only valid function names allowed here
  ...
  "265.0"      ← only numeric tokens allowed (type: number)
  ...
  "345.0"      ← same

Output:
{
  "prompt": "What is the sum of 265 and 345?",
  "name": "fn_add_numbers",
  "parameters": {"a": 265.0, "b": 345.0}
}
```

---

### Example 2: String argument

```
Input prompt:   "Reverse the string 'hello'"

Model reads:    fn_reverse_string(s: string) — "Reverse a string"
Model decides:  fn_reverse_string, s="hello"

At the 's' parameter, type is string:
  → string tokens are allowed: "h", "e", "l", "l", "o"
  → numbers, booleans, null are blocked

Output:
{
  "prompt": "Reverse the string 'hello'",
  "name": "fn_reverse_string",
  "parameters": {"s": "hello"}
}
```

---

### Example 3: Greeting with name extraction

```
Input prompt:   "Say hello to Alice"

Model reads:    fn_greet(name: string) — "Generate a greeting for a person by name"
Model decides:  fn_greet, name="Alice"

Note: the model must extract "Alice" from the natural language phrasing.
      "Say hello to" provides semantic context that maps to fn_greet.
      The name argument is filled with the extracted entity "Alice".

Output:
{
  "prompt": "Say hello to Alice",
  "name": "fn_greet",
  "parameters": {"name": "Alice"}
}
```

---

### Example 4: Different phrasing, same function

```
Prompt A: "What is the sum of 2 and 3?"      → fn_add_numbers(a=2.0, b=3.0)
Prompt B: "Add 1000 and 0.5"                 → fn_add_numbers(a=1000.0, b=0.5)
Prompt C: "What's 7 plus 8?"                 → fn_add_numbers(a=7.0, b=8.0)
Prompt D: "Calculate the total of 12 and 6"  → fn_add_numbers(a=12.0, b=6.0)
```

All four prompts should produce `fn_add_numbers` — the model handles the semantic variation.

---

## Type System

The subject specifies these types in `functions_definition.json`:

### `number`

Maps to Python `float`. In the JSON output, numeric values should be floats:

```json
{"a": 2.0, "b": 3.0}   ✅
{"a": 2, "b": 3}        ⚠️  integer — check if your validator accepts this
{"a": "2", "b": "3"}   ❌  strings — wrong type, will fail validation
```

During constrained decoding, only tokens that form a valid number (digits, `.`, `-`, `e`) are allowed when filling a `number` argument.

### `string`

Maps to Python `str`. The value must be a JSON string (inside double quotes):

```json
{"name": "John"}    ✅
{"name": John}      ❌  not quoted — invalid JSON
{"name": 42}        ❌  wrong type
```

During constrained decoding, once inside a string value, all printable character tokens are valid — but you must track the closing `"` correctly.

### `boolean` (if present)

Maps to Python `bool`. JSON booleans are lowercase:

```json
{"flag": true}    ✅
{"flag": True}    ❌  Python syntax, not valid JSON
{"flag": "true"}  ❌  string, not boolean
```

---

## Edge Cases to Handle

These are explicitly mentioned in the subject or likely to appear in peer review:

### Malformed input files

```python
try:
    with open(path) as f:
        data = json.load(f)
except FileNotFoundError:
    print(f"Error: input file '{path}' not found")
    sys.exit(1)
except json.JSONDecodeError as e:
    print(f"Error: invalid JSON in '{path}': {e}")
    sys.exit(1)
```

### Empty string arguments

```
Prompt: "Reverse the string ''"
Output: {"name": "fn_reverse_string", "parameters": {"s": ""}}
```

An empty string `""` is valid — your constrained decoder must handle zero-length string values.

### Large numbers

```
Prompt: "What is the sum of 999999 and 0.000001?"
Output: {"name": "fn_add_numbers", "parameters": {"a": 999999.0, "b": 0.000001}}
```

No hardcoded number limits — let the model generate any valid numeric token sequence.

### Special characters in strings

```
Prompt: "Reverse the string 'hello world'"
Output: {"name": "fn_reverse_string", "parameters": {"s": "hello world"}}
```

Spaces and punctuation inside string arguments must be handled — they're valid string content.

### Ambiguous prompts

```
Prompt: "What is 5?"
```

This doesn't clearly map to any function. Your constrained decoder still forces a valid function name — the model picks whichever it assigns highest probability to. Don't crash; produce your best guess.

---

## What This Project Does NOT Do

To avoid confusion about scope:

**The project does NOT execute functions.** You are not implementing `fn_add_numbers`, `fn_greet`, or `fn_reverse_string`. You are only producing the JSON that *describes* which function to call and with what arguments.

**The project does NOT use an external API.** Everything runs locally via `llm_sdk`. No OpenAI, no Anthropic, no internet.

**The project does NOT use prompting alone.** Simply asking the model to output JSON is not the approach here — constrained decoding at the token level is. The subject is explicit that relying on the model to spontaneously produce correct JSON is forbidden.

**The project does NOT hardcode answers.** Your solution must work with any valid `functions_definition.json` and any `function_calling_tests.json` — not just the examples provided. The peer reviewer will test with different files.

---

*See also: [`CONSTRAINED_DECODING.md`](./CONSTRAINED_DECODING.md) for the token-by-token algorithm, [`TOKENIZATION.md`](./TOKENIZATION.md) for how the vocabulary file works.*