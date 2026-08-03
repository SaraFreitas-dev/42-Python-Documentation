# 📐 Schemas — The Blueprint That Connects Everything

> **What this doc covers:** What a schema is, how `functions_definition.json` is itself a schema, and how that single schema drives three different parts of the project: prompt construction, pydantic validation, and constrained decoding. The schema is the thread that connects the entire pipeline.

---

## Table of Contents

1. [What is a Schema?](#what-is-a-schema)
2. [Data vs Schema — The Key Distinction](#data-vs-schema--the-key-distinction)
3. [The Schema in This Project](#the-schema-in-this-project)
4. [Schema → Prompt Construction](#schema--prompt-construction)
5. [Schema → Pydantic Validation](#schema--pydantic-validation)
6. [Schema → Constrained Decoding](#schema--constrained-decoding)
7. [The Schema as a Single Source of Truth](#the-schema-as-a-single-source-of-truth)
8. [Type Mapping Across the Pipeline](#type-mapping-across-the-pipeline)

---

## What is a Schema?

A **schema** is a description of the *shape* of data — not the data itself. It defines:

- What fields exist
- What type each field must be
- Which fields are required vs optional
- What values are valid

Think of it like a contract or a blueprint:

```
Blueprint of a house  ≠  The actual house
Schema of a function  ≠  The actual function call
```

You've already encountered schemas without necessarily calling them that:

| Thing you know | What it actually is |
|----------------|-------------------|
| A pydantic `BaseModel` | A schema defined in Python |
| A SQL `CREATE TABLE` statement | A schema for database rows |
| A TypeScript `interface` | A schema for objects |
| `functions_definition.json` | A schema for function calls |

In this project, the word "schema" refers specifically to the structure defined in `functions_definition.json` — the rules that describe what a valid function call looks like.

---

## Data vs Schema — The Key Distinction

This distinction is fundamental to understanding the project's pipeline.

### Schema — the description

```json
{
  "name": "fn_add_numbers",
  "description": "Add two numbers together and return their sum.",
  "parameters": {
    "a": {"type": "number"},
    "b": {"type": "number"}
  },
  "returns": {"type": "number"}
}
```

This says: *"there exists a function called `fn_add_numbers` that takes two number parameters named `a` and `b`."*

It describes the **shape** of valid calls. It does not contain any actual values.

### Data — a concrete instance

```json
{
  "prompt": "What is the sum of 2 and 3?",
  "name": "fn_add_numbers",
  "parameters": {"a": 2.0, "b": 3.0}
}
```

This is actual data — a concrete function call with real values. It must conform to the schema above.

### The relationship

```
Schema defines the rules
Data must follow the rules

Schema: parameter "a" must be a number
Data:   "a": 2.0   ✅  is a number
Data:   "a": "two" ❌  is a string — violates the schema
```

Every output entry your program produces is **data that must conform to its schema**. This is why both pydantic and constrained decoding exist — they are two different mechanisms for enforcing the same schema, at different points in the pipeline.

---

## The Schema in This Project

`functions_definition.json` defines the schema for every possible function call your system can produce. Let's look at its full structure:

```json
[
  {
    "name": "fn_add_numbers",
    "description": "Add two numbers together and return their sum.",
    "parameters": {
      "a": {"type": "number"},
      "b": {"type": "number"}
    },
    "returns": {"type": "number"}
  },
  {
    "name": "fn_greet",
    "description": "Generate a greeting message for a person by name.",
    "parameters": {
      "name": {"type": "string"}
    },
    "returns": {"type": "string"}
  },
  {
    "name": "fn_reverse_string",
    "description": "Reverse a string and return the reversed result.",
    "parameters": {
      "s": {"type": "string"}
    },
    "returns": {"type": "string"}
  }
]
```

### What each field means for your code

| Field | Used for |
|-------|---------|
| `name` | Constrained decoding — only these names are valid in the output's `"name"` field |
| `description` | Prompt construction — helps the model understand what each function does |
| `parameters` keys | Constrained decoding — only these key names are valid inside `"parameters"` |
| `parameters[x].type` | Constrained decoding — determines which tokens are valid for each value |
| `returns` | Informational — not directly used in output validation for this project |

### The schema is dynamic

**You must never hardcode function names or parameter names.** The peer reviewer will test your solution with a different `functions_definition.json`. Your code must read and adapt to whatever schema it receives:

```python
# ❌ Hardcoded — will fail with different function definitions
if "add" in prompt:
    output = {"name": "fn_add_numbers", "parameters": {"a": ..., "b": ...}}

# ✅ Dynamic — reads schema at runtime and adapts
fn_defs = load_function_definitions(args.functions_definition)
# Now your constrained decoder uses fn_defs to know what's valid
```

---

## Schema → Prompt Construction

The first place the schema is used: building the prompt that the model sees.

The `description` field is the key piece here. The model needs to understand what each function does in order to choose the right one. You inject the schema into the prompt:

```python
def build_prompt(
    user_request: str,
    fn_defs: list[FunctionDefinition]
) -> str:
    """Build a prompt that includes function descriptions for model context."""

    # Format function signatures from the schema
    fn_descriptions = []
    for fn in fn_defs:
        params = ", ".join(
            f"{name}: {schema.type}"
            for name, schema in fn.parameters.items()
        )
        fn_descriptions.append(
            f"- {fn.name}({params}): {fn.description}"
        )

    functions_block = "\n".join(fn_descriptions)

    return (
        f"You are a function calling assistant.\n\n"
        f"Available functions:\n{functions_block}\n\n"
        f'User request: "{user_request}"\n\n'
        f'Output the correct function call as JSON:\n'
        f'{{"name": "'
        # prompt ends here — constrained decoding takes over
    )
```

For the example schema above, this produces:

```
You are a function calling assistant.

Available functions:
- fn_add_numbers(a: number, b: number): Add two numbers together and return their sum.
- fn_greet(name: string): Generate a greeting message for a person by name.
- fn_reverse_string(s: string): Reverse a string and return the reversed result.

User request: "What is the sum of 2 and 3?"

Output the correct function call as JSON:
{"name": "
```

The model reads the descriptions and assigns higher probability to `fn_add_numbers` because the prompt mentions "sum" and "numbers". The constrained decoder then restricts which function names are actually selectable.

---

## Schema → Pydantic Validation

The second place the schema is used: validating data after it's loaded or generated.

Your pydantic models mirror the schema structure exactly:

```
functions_definition.json schema      →    Pydantic models
─────────────────────────────────────────────────────────
{ "type": "number" }                  →    ParameterSchema(type="number")
{ "name": ..., "parameters": {...} }  →    FunctionDefinition(name=..., parameters={...})
{ "prompt": ..., "name": ..., ... }   →    FunctionCall(prompt=..., name=..., ...)
```

Pydantic enforces the schema at **load time** and **output time**:

```python
# At load time — validate the schema file itself
fn_defs = [FunctionDefinition.model_validate(item) for item in raw_json]
# If functions_definition.json is malformed, this raises ValidationError immediately

# At output time — validate the LLM's output against the schema
call = FunctionCall.model_validate(llm_output)
call.validate_against_definition(matching_fn_def)
# If the LLM somehow produced wrong types or missing fields, this catches it
```

### Schema drives validator logic

The schema's type system feeds directly into custom pydantic validators:

```python
from pydantic import BaseModel, field_validator

class ParameterSchema(BaseModel):
    type: str

    @field_validator("type")
    @classmethod
    def type_must_be_supported(cls, v: str) -> str:
        """Validate that the type is one the constrained decoder can handle."""
        supported = {"number", "string", "boolean"}
        if v not in supported:
            raise ValueError(
                f"Unsupported parameter type '{v}'. "
                f"Constrained decoder supports: {supported}"
            )
        return v
```

This validator exists because the constrained decoder has specific logic for each type. If an unknown type appears in the schema, the decoder wouldn't know which tokens to allow — so pydantic catches it early with a clear error.

---

## Schema → Constrained Decoding

The third and most critical place the schema is used: controlling which tokens the model is allowed to generate at each step.

The schema answers three questions the constrained decoder asks constantly:

### Question 1: Which function names are valid?

```python
# Schema says these functions exist:
valid_fn_names = [fn.name for fn in fn_defs]
# → ["fn_add_numbers", "fn_greet", "fn_reverse_string"]

# Constrained decoder: when generating the "name" value,
# only allow tokens that continue one of these names
valid_tokens = get_tokens_continuing_any(vocab, valid_fn_names, written_so_far)
```

### Question 2: Which parameter names are valid?

```python
# Schema says fn_add_numbers has parameters "a" and "b"
current_fn = get_fn_def("fn_add_numbers", fn_defs)
valid_param_names = list(current_fn.parameters.keys())
# → ["a", "b"]

# Constrained decoder: when generating parameter keys,
# only allow tokens that continue one of these names
# (and exclude names already written)
remaining_params = [p for p in valid_param_names if p not in written_params]
valid_tokens = get_tokens_continuing_any(vocab, remaining_params, written_so_far)
```

### Question 3: What type of value is allowed here?

```python
# Schema says parameter "a" has type "number"
param_type = current_fn.parameters["a"].type
# → "number"

# Constrained decoder: dispatch to the right token filter
if param_type == "number":
    valid_tokens = get_numeric_tokens(vocab, partial_value)
    # only digits, '.', '-', 'e', 'E', '+'

elif param_type == "string":
    valid_tokens = get_string_tokens(vocab, in_string, partial_value)
    # any printable character token + closing '"'

elif param_type == "boolean":
    valid_tokens = get_boolean_tokens(vocab, partial_value)
    # only tokens continuing "true" or "false"
```

### The schema as a runtime constraint engine

You can think of the schema as a **constraint engine** that the decoder queries at every token step:

```
Token step N:
  partial_json = '{"name": "fn_add_numbers", "parameters": {"a": '
  
  Decoder asks schema:
    → What function did we select?          "fn_add_numbers"
    → What parameter are we filling?        "a"
    → What type does "a" have?              "number"
    → What tokens are valid for a number?   {digits, '.', '-', ...}
  
  Apply constraints → pick token → advance
```

---

## The Schema as a Single Source of Truth

This is the elegant part of the project's design. The same `functions_definition.json` file drives every stage of the pipeline without duplication:

```
functions_definition.json
         │
         ├──▶ load_function_definitions()
         │              │
         │    ┌─────────┴──────────┐─────────────────────┐
         │    ▼                    ▼                      ▼
         │  PROMPT             PYDANTIC               CONSTRAINED
         │ CONSTRUCTION        VALIDATION              DECODING
         │    │                    │                      │
         │  Injects            Validates              Restricts
         │  descriptions       input files            token choices
         │  into model         and LLM output         at every step
         │  context                │                      │
         │                        └──────────┬───────────┘
         │                                   ▼
         │                         OUTPUT JSON FILE
         │                    function_calling_results.json
         │
         └──▶ Any change to the schema file automatically
              affects all three pipeline stages — no code changes needed
```

This is why the subject says *"do not hardcode solutions"* and *"the input files may change during peer review."* If you've built your pipeline correctly around the schema, changing `functions_definition.json` to have different functions, different parameter names, or different types requires **zero changes to your code** — the schema propagates through automatically.

---

## Type Mapping Across the Pipeline

The schema defines types as strings (`"number"`, `"string"`, `"boolean"`). Here's how each type maps across the three pipeline stages:

| Schema type | Prompt text | Python/Pydantic type | Valid token characters |
|-------------|-------------|---------------------|----------------------|
| `"number"` | `"number"` | `float` | `0-9`, `.`, `-`, `+`, `e`, `E` |
| `"string"` | `"string"` | `str` | any printable + `"` to close |
| `"boolean"` | `"boolean"` | `bool` | tokens continuing `true`/`false` |

### The number → float convention

The schema says `"number"` but JSON doesn't distinguish int from float. The subject's example output uses `2.0` and `3.0` (floats) rather than `2` and `3` (ints). Follow this convention — always output numbers as floats:

```python
# In your constrained decoder, after generating a number value:
raw_value = "2"             # model generated "2"
typed_value = float("2")   # → 2.0  ✅ matches subject example

# In your pydantic coercion validator:
@field_validator("parameters", mode="before")
@classmethod
def coerce_numbers_to_float(cls, v: dict) -> dict:
    return {
        k: float(val) if isinstance(val, int) else val
        for k, val in v.items()
    }
```

---

*See also: [`FUNCTION_CALLING.md`](./FUNCTION_CALLING.md) for the full structure of `functions_definition.json`.*
*See also: [`PYDANTIC.md`](./PYDANTIC.md) for the pydantic models that mirror this schema.*
*See also: [`CONSTRAINED_DECODING.md`](./CONSTRAINED_DECODING.md) for how the schema drives token selection.*
