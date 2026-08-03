# 🛡️ Pydantic — Data Validation for Call Me Maybe

> **Why this doc exists:** The subject explicitly requires that **all classes use pydantic for validation**. This guide explains what pydantic is, how to use `BaseModel`, how to validate the project's JSON schemas, and how to integrate pydantic with the LLM output pipeline.

---

## Table of Contents

1. [What is Pydantic?](#what-is-pydantic)
2. [Installation & Setup](#installation--setup)
3. [BaseModel — The Foundation](#basemodel--the-foundation)
4. [Type Hints in Pydantic](#type-hints-in-pydantic)
5. [Validators — Custom Validation Logic](#validators--custom-validation-logic)
6. [Models for This Project](#models-for-this-project)
7. [Validating Input Files](#validating-input-files)
8. [Validating LLM Output](#validating-llm-output)
9. [Error Handling with Pydantic](#error-handling-with-pydantic)
10. [Common Mistakes](#common-mistakes)

---

## What is Pydantic?

Pydantic is a Python library for **data validation using type hints**. You define the shape of your data as a class, and pydantic automatically:

- Checks that all required fields are present
- Validates that values match their declared types
- Converts compatible types where possible (`"42"` → `42` if you declared `int`)
- Gives you clear error messages when validation fails

Without pydantic:

```python
# Manual, fragile, verbose
def load_function_def(data: dict) -> dict:
    if "name" not in data:
        raise ValueError("Missing 'name'")
    if not isinstance(data["name"], str):
        raise TypeError("'name' must be a string")
    if "parameters" not in data:
        raise ValueError("Missing 'parameters'")
    # ... 20 more lines of this
    return data
```

With pydantic:

```python
from pydantic import BaseModel
from typing import Dict

class ParameterSchema(BaseModel):
    type: str

class FunctionDefinition(BaseModel):
    name: str
    description: str
    parameters: Dict[str, ParameterSchema]
    returns: ParameterSchema

# One line — all validation happens automatically
fn = FunctionDefinition(**data)
```

---

## Installation & Setup

Pydantic is listed as a required dependency in this project. It's installed via uv:

```bash
uv add pydantic
```

This project uses **Pydantic v2** (the current major version). The import is the same as v1, but some internals differ. Always check which version you have:

```python
import pydantic
print(pydantic.__version__)   # should be 2.x.x
```

All examples in this guide use Pydantic v2.

---

## BaseModel — The Foundation

Every pydantic model inherits from `BaseModel`. You declare fields as class attributes with type annotations:

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
    active: bool = True   # default value — field is optional
```

### Creating instances

```python
# From keyword arguments
p = Person(name="Alice", age=30)
print(p.name)    # "Alice"
print(p.age)     # 30
print(p.active)  # True  ← default

# From a dictionary (most common in this project — loading JSON)
data = {"name": "Bob", "age": 25, "active": False}
p = Person(**data)

# From a dictionary using model_validate (pydantic v2 preferred way)
p = Person.model_validate(data)
```

### Accessing and serialising

```python
p = Person(name="Alice", age=30)

# Access fields like regular attributes
print(p.name)   # "Alice"

# Convert back to dict
print(p.model_dump())
# {"name": "Alice", "age": 30, "active": True}

# Convert to JSON string
print(p.model_dump_json())
# '{"name":"Alice","age":30,"active":true}'
```

### Immutability

By default, pydantic models are **immutable** after creation — you can't change field values. This is intentional: it prevents accidental mutation of validated data.

```python
p = Person(name="Alice", age=30)
p.name = "Bob"   # ❌ raises ValidationError in pydantic v2
```

If you need mutable models, configure them explicitly:

```python
class MutableModel(BaseModel):
    model_config = {"frozen": False}   # allows mutation

    value: int
```

---

## Type Hints in Pydantic

### Basic types

```python
from pydantic import BaseModel

class Example(BaseModel):
    a: str            # string
    b: int            # integer
    c: float          # float (also accepts int — coerces automatically)
    d: bool           # boolean
    e: bytes          # raw bytes
```

### Optional fields

```python
from pydantic import BaseModel
from typing import Optional

class Example(BaseModel):
    required_field: str
    optional_field: Optional[str] = None      # can be str or None, defaults to None
    optional_with_default: int = 42           # has a default value
```

### Collections

```python
from pydantic import BaseModel
from typing import Dict, List, Any

class Example(BaseModel):
    tags: List[str]                    # list of strings
    scores: List[float]                # list of floats
    metadata: Dict[str, str]           # dict with string keys and string values
    parameters: Dict[str, Any]         # dict with string keys and any values
    nested: Dict[str, List[int]]       # dict of lists
```

### Union types (Python 3.10+ syntax)

```python
from pydantic import BaseModel

class Example(BaseModel):
    value: int | float | str           # any of these types
    maybe_string: str | None = None    # optional string (Python 3.10+)
```

Or with older syntax:

```python
from typing import Union, Optional

class Example(BaseModel):
    value: Union[int, float, str]
    maybe_string: Optional[str] = None
```

### Nested models

This is where pydantic really shines — you can nest models inside other models:

```python
from pydantic import BaseModel
from typing import Dict

class ParameterSchema(BaseModel):
    type: str

class FunctionDefinition(BaseModel):
    name: str
    description: str
    parameters: Dict[str, ParameterSchema]   # values are ParameterSchema instances
    returns: ParameterSchema

# Pydantic handles the nested validation automatically:
data = {
    "name": "fn_add_numbers",
    "description": "Add two numbers.",
    "parameters": {
        "a": {"type": "number"},             # ← automatically becomes ParameterSchema
        "b": {"type": "number"}
    },
    "returns": {"type": "number"}            # ← automatically becomes ParameterSchema
}

fn = FunctionDefinition(**data)
print(fn.parameters["a"].type)   # "number"
print(type(fn.parameters["a"]))  # <class 'ParameterSchema'>
```

---

## Validators — Custom Validation Logic

Sometimes type checking alone isn't enough. Pydantic lets you add custom validation logic with the `@field_validator` decorator.

### Field validator

```python
from pydantic import BaseModel, field_validator

class FunctionDefinition(BaseModel):
    name: str
    description: str
    parameters: Dict[str, ParameterSchema]
    returns: ParameterSchema

    @field_validator("name")
    @classmethod
    def name_must_start_with_fn(cls, v: str) -> str:
        if not v.startswith("fn_"):
            raise ValueError(f"Function name '{v}' must start with 'fn_'")
        return v

    @field_validator("parameters")
    @classmethod
    def parameters_must_not_be_empty(cls, v: Dict) -> Dict:
        if not v:
            raise ValueError("Function must have at least one parameter")
        return v
```

### Model validator (cross-field validation)

When validation depends on multiple fields at once:

```python
from pydantic import BaseModel, model_validator
from typing import Any

class FunctionCall(BaseModel):
    prompt: str
    name: str
    parameters: Dict[str, Any]

    @model_validator(mode="after")
    def check_parameters_not_empty_for_non_void(self) -> "FunctionCall":
        # Example: if name suggests a function with args, params shouldn't be empty
        if self.name != "fn_no_args" and not self.parameters:
            raise ValueError(
                f"Function '{self.name}' should have parameters but got none"
            )
        return self
```

### Validator for type coercion

```python
from pydantic import BaseModel, field_validator
from typing import Any, Dict

class FunctionCall(BaseModel):
    prompt: str
    name: str
    parameters: Dict[str, Any]

    @field_validator("parameters", mode="before")
    @classmethod
    def coerce_numbers(cls, v: Dict[str, Any]) -> Dict[str, Any]:
        """Ensure number values are floats, not ints."""
        # Useful when LLM outputs 2 instead of 2.0
        return {
            k: float(val) if isinstance(val, int) else val
            for k, val in v.items()
        }
```

---

## Models for This Project

Here are the complete pydantic models you'll need, directly matching the project's data structures.

### Input models

```python
from pydantic import BaseModel, field_validator
from typing import Dict, List, Any


class ParameterSchema(BaseModel):
    """Schema for a single function parameter."""
    type: str   # "number", "string", "boolean"

    @field_validator("type")
    @classmethod
    def type_must_be_valid(cls, v: str) -> str:
        allowed = {"number", "string", "boolean", "integer", "array", "object"}
        if v not in allowed:
            raise ValueError(f"Unknown parameter type '{v}'. Expected one of {allowed}")
        return v


class FunctionDefinition(BaseModel):
    """A single function definition from functions_definition.json."""
    name: str
    description: str
    parameters: Dict[str, ParameterSchema]
    returns: ParameterSchema

    @field_validator("name")
    @classmethod
    def name_must_be_valid_identifier(cls, v: str) -> str:
        if not v.replace("_", "").isalnum():
            raise ValueError(f"Function name '{v}' must be alphanumeric with underscores")
        return v

    def get_param_type(self, param_name: str) -> str:
        """Get the type of a specific parameter."""
        if param_name not in self.parameters:
            raise KeyError(f"Parameter '{param_name}' not in function '{self.name}'")
        return self.parameters[param_name].type

    def param_names(self) -> List[str]:
        """Return ordered list of parameter names."""
        return list(self.parameters.keys())


class TestPrompt(BaseModel):
    """A single test prompt from function_calling_tests.json."""
    prompt: str

    @field_validator("prompt")
    @classmethod
    def prompt_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Prompt must not be empty or whitespace only")
        return v.strip()
```

### Output models

```python
class FunctionCall(BaseModel):
    """A single result entry in the output JSON."""
    prompt: str
    name: str
    parameters: Dict[str, Any]

    def validate_against_definition(
        self, fn_def: FunctionDefinition
    ) -> bool:
        """
        Check that this function call matches its definition.
        Returns True if valid, raises ValueError if not.
        """
        # Check function name matches
        if self.name != fn_def.name:
            raise ValueError(
                f"Function name mismatch: got '{self.name}', expected '{fn_def.name}'"
            )

        # Check all required parameters are present
        for param_name in fn_def.param_names():
            if param_name not in self.parameters:
                raise ValueError(
                    f"Missing required parameter '{param_name}' "
                    f"for function '{self.name}'"
                )

        # Check parameter types
        for param_name, value in self.parameters.items():
            if param_name not in fn_def.parameters:
                raise ValueError(
                    f"Unexpected parameter '{param_name}' "
                    f"in function '{self.name}'"
                )
            expected_type = fn_def.get_param_type(param_name)
            self._check_type(param_name, value, expected_type)

        return True

    def _check_type(self, name: str, value: Any, expected: str) -> None:
        """Validate a single parameter value against its expected type."""
        if expected == "number":
            if not isinstance(value, (int, float)):
                raise ValueError(
                    f"Parameter '{name}': expected number, got {type(value).__name__}"
                )
        elif expected == "string":
            if not isinstance(value, str):
                raise ValueError(
                    f"Parameter '{name}': expected string, got {type(value).__name__}"
                )
        elif expected == "boolean":
            if not isinstance(value, bool):
                raise ValueError(
                    f"Parameter '{name}': expected boolean, got {type(value).__name__}"
                )


class OutputFile(BaseModel):
    """The complete output file — a list of function calls."""
    results: List[FunctionCall]

    def to_json_file(self, path: str) -> None:
        """Write results to a JSON file in the required format."""
        import json
        output = [r.model_dump() for r in self.results]
        with open(path, "w", encoding="utf-8") as f:
            json.dump(output, f, indent=2, ensure_ascii=False)
```

---

## Validating Input Files

This is where you use the models above to safely load and validate the project's input files.

```python
import json
import sys
from pathlib import Path
from pydantic import ValidationError


def load_function_definitions(path: str) -> List[FunctionDefinition]:
    """
    Load and validate functions_definition.json.
    Exits with a clear error message if the file is missing or invalid.
    """
    file_path = Path(path)

    # Check file exists
    if not file_path.exists():
        print(f"Error: functions definition file not found: '{path}'")
        sys.exit(1)

    # Parse JSON
    try:
        with open(file_path, encoding="utf-8") as f:
            raw = json.load(f)
    except json.JSONDecodeError as e:
        print(f"Error: invalid JSON in '{path}': {e}")
        sys.exit(1)

    # Validate with pydantic
    try:
        return [FunctionDefinition.model_validate(item) for item in raw]
    except ValidationError as e:
        print(f"Error: invalid function definition schema in '{path}':")
        for error in e.errors():
            location = " → ".join(str(loc) for loc in error["loc"])
            print(f"  [{location}] {error['msg']}")
        sys.exit(1)


def load_test_prompts(path: str) -> List[TestPrompt]:
    """
    Load and validate function_calling_tests.json.
    """
    file_path = Path(path)

    if not file_path.exists():
        print(f"Error: test prompts file not found: '{path}'")
        sys.exit(1)

    try:
        with open(file_path, encoding="utf-8") as f:
            raw = json.load(f)
    except json.JSONDecodeError as e:
        print(f"Error: invalid JSON in '{path}': {e}")
        sys.exit(1)

    try:
        return [TestPrompt.model_validate(item) for item in raw]
    except ValidationError as e:
        print(f"Error: invalid prompt schema in '{path}':")
        for error in e.errors():
            location = " → ".join(str(loc) for loc in error["loc"])
            print(f"  [{location}] {error['msg']}")
        sys.exit(1)
```

---

## Validating LLM Output

After constrained decoding produces a JSON string, you validate it with pydantic before writing to the output file:

```python
import json
from pydantic import ValidationError


def parse_and_validate_output(
    raw_json: str,
    original_prompt: str,
    fn_defs: List[FunctionDefinition]
) -> FunctionCall:
    """
    Parse the raw JSON string produced by constrained decoding,
    validate its structure and schema, and return a FunctionCall.
    """
    # Step 1: parse JSON (should always succeed after constrained decoding,
    # but be defensive)
    try:
        data = json.loads(raw_json)
    except json.JSONDecodeError as e:
        raise ValueError(f"Constrained decoding produced invalid JSON: {e}\n{raw_json}")

    # Step 2: add the original prompt to the data
    data["prompt"] = original_prompt

    # Step 3: validate structure with pydantic
    try:
        call = FunctionCall.model_validate(data)
    except ValidationError as e:
        raise ValueError(f"LLM output failed schema validation: {e}")

    # Step 4: find the matching function definition
    fn_def = next((fn for fn in fn_defs if fn.name == call.name), None)
    if fn_def is None:
        raise ValueError(
            f"LLM selected unknown function '{call.name}'. "
            f"Available: {[fn.name for fn in fn_defs]}"
        )

    # Step 5: validate parameters against the function definition
    call.validate_against_definition(fn_def)

    return call
```

---

## Error Handling with Pydantic

### What ValidationError looks like

```python
from pydantic import BaseModel, ValidationError

class ParameterSchema(BaseModel):
    type: str

try:
    p = ParameterSchema(type=42)   # wrong type — int instead of str
except ValidationError as e:
    print(e)
    # 1 validation error for ParameterSchema
    # type
    #   Input should be a valid string [type=string_type, ...]
```

### Iterating over errors

```python
try:
    fn = FunctionDefinition(**bad_data)
except ValidationError as e:
    for error in e.errors():
        print(f"Field:   {error['loc']}")    # e.g. ('parameters', 'a', 'type')
        print(f"Message: {error['msg']}")    # e.g. "Field required"
        print(f"Type:    {error['type']}")   # e.g. "missing"
        print()
```

### Graceful error messages for the user

The subject requires that all errors be handled gracefully with clear messages. Never let a `ValidationError` propagate unhandled:

```python
def safe_load(data: dict, model_class: type) -> BaseModel | None:
    """Attempt to validate data, returning None on failure with a printed error."""
    try:
        return model_class.model_validate(data)
    except ValidationError as e:
        print(f"Validation failed for {model_class.__name__}:")
        for err in e.errors():
            loc = " → ".join(str(l) for l in err["loc"])
            print(f"  {loc}: {err['msg']}")
        return None
```

---

## Common Mistakes

### ❌ Using plain `dict` instead of a model

```python
# Bad — no validation, no type safety
def process(fn_data: dict) -> None:
    name = fn_data["name"]   # KeyError if missing
    ...

# Good — validated at creation time
def process(fn_def: FunctionDefinition) -> None:
    name = fn_def.name       # always a str, always present
    ...
```

---

### ❌ Forgetting `model_validate` for dict input

```python
data = {"type": "number"}

# Bad in pydantic v2 — may not trigger all validators
p = ParameterSchema(type=data["type"])

# Good — use model_validate when input is a dict
p = ParameterSchema.model_validate(data)
```

---

### ❌ Catching the wrong exception

```python
# Bad — json.JSONDecodeError is not a pydantic error
try:
    fn = FunctionDefinition(**data)
except json.JSONDecodeError:   # ❌ will never catch pydantic errors
    ...

# Good
from pydantic import ValidationError
try:
    fn = FunctionDefinition(**data)
except ValidationError as e:   # ✅
    ...
```

---

### ❌ Using `dict()` instead of `model_dump()`

```python
fn_call = FunctionCall(prompt="...", name="fn_greet", parameters={"name": "john"})

# Bad — deprecated in pydantic v2
output = fn_call.dict()        # ⚠️ works but shows deprecation warning

# Good — pydantic v2 method
output = fn_call.model_dump()  # ✅
```

---

### ❌ Not validating LLM output

Even with constrained decoding guaranteeing valid JSON structure, always run the output through pydantic. Constrained decoding ensures JSON syntax — pydantic ensures semantic correctness (right keys, right types, right function name):

```python
# Bad — trusting raw LLM output directly
result = json.loads(raw_json)
output_list.append(result)   # no validation

# Good — validate every output
try:
    call = parse_and_validate_output(raw_json, prompt, fn_defs)
    output_list.append(call.model_dump())
except ValueError as e:
    print(f"Warning: could not validate output for prompt '{prompt}': {e}")
    # handle gracefully — don't crash
```

---

*See also: [`FUNCTION_CALLING.md`](./FUNCTION_CALLING.md) for the data structures these models represent.*
*See also: [`CONSTRAINED_DECODING.md`](./CONSTRAINED_DECODING.md) for where validation fits in the generation pipeline.*
*Official pydantic v2 docs: [docs.pydantic.dev](https://docs.pydantic.dev)*
