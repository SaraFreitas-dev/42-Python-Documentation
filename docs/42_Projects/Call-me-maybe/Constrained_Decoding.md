# 🔒 Constrained Decoding — Guaranteeing Valid JSON Output

> **What this doc covers:** The token-by-token generation loop, how to track JSON state at each step, how to use the vocabulary file to identify valid tokens, and how to enforce schema types (number, string, boolean). This is the core algorithm of the project.

---

## Table of Contents

1. [Why Constrained Decoding Exists](#why-constrained-decoding-exists)
2. [The Generation Loop](#the-generation-loop)
3. [The Vocabulary File](#the-vocabulary-file)
4. [Tracking JSON State](#tracking-json-state)
5. [Valid Tokens at Each JSON Position](#valid-tokens-at-each-json-position)
6. [Enforcing Schema Types](#enforcing-schema-types)
7. [The -inf Trick](#the--inf-trick)
8. [Complete Walkthrough](#complete-walkthrough)
9. [Putting It All Together](#putting-it-all-together)
10. [Common Mistakes](#common-mistakes)

---

## Why Constrained Decoding Exists

At each generation step, the model outputs a probability score (logit) for every token in its vocabulary — roughly 151,643 tokens for Qwen3. Normally, you just pick the highest one.

The problem: **nothing stops the model from picking a token that breaks your JSON.**

```
Partial output so far:  '{"name": "'
Model's top candidates:
  "fn_add_numbers"   logit: 3.2   ✅ valid function name
  "fn_greet"         logit: 2.9   ✅ valid function name
  "The"              logit: 1.4   ❌ breaks JSON — not a function name
  "42"               logit: 0.8   ❌ breaks schema — name must be a string
  " "                logit: 0.3   ❌ breaks JSON — unexpected whitespace here
```

Without intervention, the model might pick `"The"` and produce broken output. With constrained decoding, you set all invalid token logits to `-inf` before selection, so the model can only ever pick a valid token.

```
After constraint:
  "fn_add_numbers"   logit: 3.2   ✅ stays
  "fn_greet"         logit: 2.9   ✅ stays
  "The"              logit: -inf  ❌ eliminated
  "42"               logit: -inf  ❌ eliminated
  " "                logit: -inf  ❌ eliminated

Selected token: "fn_add_numbers"  ← correct, guaranteed
```

**Result:** 100% valid, schema-compliant JSON every time — regardless of model size.

---

## The Generation Loop

This is the fundamental loop you implement. Everything else in the project plugs into it.

```python
import math
import json
from typing import List

def generate_constrained(
    model,
    prompt: str,
    fn_def: FunctionDefinition,
    all_fn_names: List[str],
    str_to_id: dict[str, int],   # token_string → token_id  (real vocab format)
    id_to_str: dict[int, str],   # token_id → token_string  (reverse lookup)
    max_tokens: int = 200
) -> dict:
    """Generate a constrained function call JSON for a given prompt."""

    # Step 1: tokenise the prompt — encode() returns a 2D tensor, use [0].tolist()
    input_ids: List[int] = model.encode(prompt)[0].tolist()

    generated_ids: List[int] = []
    partial_json: str = ""

    for _ in range(max_tokens):
        # Step 2: get logits for all vocabulary tokens
        all_ids = input_ids + generated_ids
        logits: List[float] = list(model.get_logits_from_input_ids(all_ids))

        # Step 3: determine valid tokens at this position
        valid_ids = get_valid_token_ids(partial_json, fn_def, all_fn_names, str_to_id)

        # Step 4: mask all invalid tokens
        for i in range(len(logits)):
            if i not in valid_ids:
                logits[i] = -math.inf

        # Step 5: pick the highest-scoring valid token
        next_id = int(max(range(len(logits)), key=lambda i: logits[i]))

        # Step 6: append and update state
        generated_ids.append(next_id)
        token_str = id_to_str[next_id]   # use the reverse lookup: id → string
        partial_json += token_str

        # Step 7: check if generation is complete
        if is_complete(partial_json, fn_def):
            break

    return json.loads(partial_json)
```

The key function is `get_valid_token_ids` — it answers the question: *given what we've generated so far, which tokens are legal next?*

---

## The Vocabulary File

`model.get_path_to_vocab_file()` returns the path to a JSON file. The **real format** maps **token strings to integer IDs**:

```python
import json

vocab_path = model.get_path_to_vocab_file()
with open(vocab_path, "r", encoding="utf-8") as f:
    str_to_id: dict[str, int] = json.load(f)

# Real structure:
# {"!": 0, '"': 1, "#": 2, ..., "{": 5476, "}": 9313, ...}
# key   = token string
# value = integer ID

# Build the reverse lookup for use after token selection
id_to_str: dict[int, str] = {v: k for k, v in str_to_id.items()}
```

> **⚠️ The key is the token string, the value is the ID** — not the other way around. Always build the reverse `id_to_str` lookup so you can go in both directions.

### What to look for in the vocabulary

You need to find the token IDs for specific characters and strings. Build these lookup sets once at startup, not inside the generation loop:

```python
def build_vocab_lookups(str_to_id: dict[str, int]) -> dict:
    """Pre-compute useful token sets from the vocabulary."""

    # Normalize helper — handles Ġ (leading space) and other markers
    def norm(s: str) -> str:
        return s.replace("Ġ", " ").replace("Ċ", "\n")

    # Single structural characters — find all variants (with/without leading space)
    open_brace   = {tid for s, tid in str_to_id.items() if norm(s) == "{"}
    close_brace  = {tid for s, tid in str_to_id.items() if norm(s) == "}"}
    open_quote   = {tid for s, tid in str_to_id.items() if norm(s) == '"'}
    colon        = {tid for s, tid in str_to_id.items() if norm(s).strip() == ":"}
    comma        = {tid for s, tid in str_to_id.items() if norm(s).strip() == ","}

    # Numeric tokens — digits, decimal point, minus, scientific notation
    numeric      = {tid for s, tid in str_to_id.items()
                    if norm(s) and all(c in "0123456789.-+eE" for c in norm(s))}

    # Whitespace tokens
    whitespace   = {tid for s, tid in str_to_id.items()
                    if norm(s).strip() == "" and norm(s) != ""}

    return {
        "{": open_brace,
        "}": close_brace,
        '"': open_quote,
        ":": colon,
        ",": comma,
        "numeric": numeric,
        "whitespace": whitespace,
    }
```

### The leading-space problem

Tokenizers attach spaces to the **beginning** of the following token, not the end of the previous one. A space before `"` is part of the `"` token, not a separate space token.

```
Text:    '{"name": "fn_greet"}'
Tokens:  ['{', '"', 'name', '"', ':', 'Ġ"', 'fn', '_greet', '"', '}']
                                         ↑
                              space is part of this token: 'Ġ"'
```

This matters when you search the vocabulary for `:`. There may be tokens for `:` and `Ġ:` (with leading space). You need to account for all variants:

```python
colon_variants = {tid for s, tid in str_to_id.items() if s.replace("Ġ", " ").strip() == ":"}
```

---

## Tracking JSON State

To know which tokens are valid at each position, you need to know **where you are in the JSON structure**. The simplest approach is a state machine.

### The JSON structure you're generating

```
{
  "name": "<function_name>",
  "parameters": {
    "<param1>": <value1>,
    "<param2>": <value2>
  }
}
```

### States

```python
from enum import Enum, auto

class JSONState(Enum):
    START               = auto()   # haven't written anything yet → expect '{'
    AFTER_OPEN_BRACE    = auto()   # wrote '{' → expect '"name"'
    IN_NAME_KEY         = auto()   # writing the literal key "name"
    AFTER_NAME_KEY      = auto()   # wrote '"name"' → expect ':'
    IN_NAME_VALUE       = auto()   # writing the function name string
    AFTER_NAME_VALUE    = auto()   # wrote function name → expect ','
    IN_PARAMS_KEY       = auto()   # writing '"parameters"'
    AFTER_PARAMS_KEY    = auto()   # wrote '"parameters"' → expect ':'
    IN_PARAMS_OPEN      = auto()   # wrote ':' → expect '{'
    IN_ARG_KEY          = auto()   # writing an argument name key
    AFTER_ARG_KEY       = auto()   # wrote key → expect ':'
    IN_ARG_VALUE_NUM    = auto()   # writing a numeric argument value
    IN_ARG_VALUE_STR    = auto()   # writing a string argument value
    IN_ARG_VALUE_BOOL   = auto()   # writing a boolean argument value
    AFTER_ARG_VALUE     = auto()   # wrote value → expect ',' or '}'
    COMPLETE            = auto()   # wrote final '}' → done
```

### Parsing current state from partial output

Rather than tracking state with a variable, you can **infer it from what's been generated so far**. This is more robust and easier to debug:

```python
def get_current_state(partial: str, fn_def: FunctionDefinition) -> JSONState:
    """Determine current JSON generation state from partial output."""
    partial = partial.strip()

    if partial == "":
        return JSONState.START

    if partial == "{":
        return JSONState.AFTER_OPEN_BRACE

    if not partial.startswith('{"name"'):
        return JSONState.IN_NAME_KEY

    if ',' not in partial:
        if partial.count('"') < 4:
            return JSONState.IN_NAME_VALUE
        return JSONState.AFTER_NAME_VALUE

    if '"parameters"' not in partial:
        return JSONState.IN_PARAMS_KEY

    # We're inside the parameters object — expand this logic for your implementation
    return JSONState.IN_ARG_KEY
```

> **Tip:** For reliability, track state as a variable that you update token by token, rather than re-parsing the full partial string each time. The walkthrough section below shows this approach.

---

## Valid Tokens at Each JSON Position

Here's what tokens are valid at each stage of generating the output JSON:

### Stage 1: Start → `{`

```python
# Only the open brace is valid
valid = {tid for s, tid in str_to_id.items() if s.replace("Ġ", " ").strip() == "{"}
```

### Stage 2: After `{` → the key `"name"`

```python
# Next tokens must spell out exactly '"name"'
already_written = len(partial) - 1  # chars after '{'
target = '"name"'
remaining = target[already_written:]
valid = {tid for s, tid in str_to_id.items() if remaining.startswith(s.replace("Ġ", " "))}
```

### Stage 3: Inside function name value → restricted to valid function names

```python
# partial ends with '{"name": "'
# next tokens must continue one of the valid function names
written_so_far = extract_name_value(partial)  # e.g. "" or "fn_" or "fn_greet"

valid = set()
for fn_name in all_function_names:
    if fn_name.startswith(written_so_far):
        remaining = fn_name[len(written_so_far):]
        for s, tid in str_to_id.items():
            norm = s.replace("Ġ", " ")
            if remaining.startswith(norm) and norm:
                valid.add(tid)
```

### Stage 4: Argument value → depends on schema type

```python
param_type = current_fn.parameters[current_param].type

if param_type == "number":
    valid = get_numeric_tokens(str_to_id, partial_value)

elif param_type == "string":
    valid = get_string_tokens(str_to_id, in_string, partial_value)

elif param_type == "boolean":
    valid = get_boolean_tokens(str_to_id, partial_value)
```

---

## Enforcing Schema Types

This is where you use `fn_def.parameters[param_name].type` to decide which tokens are valid inside the argument value.

### `type: "number"`

```python
def get_valid_number_tokens(
    str_to_id: dict[str, int],
    partial_value: str
) -> set[int]:
    """Returns token IDs that can legally continue a JSON number value."""
    valid = set()

    for s, tid in str_to_id.items():
        norm = s.replace("Ġ", " ").strip()
        candidate = partial_value + norm
        if is_valid_number_prefix(candidate):
            valid.add(tid)

    return valid


def is_valid_number_prefix(s: str) -> bool:
    """Return True if s is a valid prefix of a JSON number."""
    import re
    pattern = r'^-?(?:\d+(?:\.\d*)?(?:[eE][+-]?\d*)?)?$'
    return bool(re.match(pattern, s))
```

### `type: "string"`

```python
def get_valid_string_tokens(
    str_to_id: dict[str, int],
    in_string: bool,
    string_content: str
) -> set[int]:

    if not in_string:
        return {tid for s, tid in str_to_id.items() if s == '"'}

    valid = set()
    for s, tid in str_to_id.items():
        if s == '"':
            valid.add(tid)   # closing quote
            continue
        if '"' not in s and '\\' not in s:
            valid.add(tid)   # any token without unescaped quote
        elif '\\"' in s:
            valid.add(tid)   # escaped quote is fine

    return valid
```

### `type: "boolean"`

```python
def get_valid_boolean_tokens(
    str_to_id: dict[str, int],
    partial_value: str
) -> set[int]:
    """Tokens that continue 'true' or 'false' from partial_value."""
    valid = set()

    for target in ["true", "false"]:
        if target.startswith(partial_value):
            remaining = target[len(partial_value):]
            for s, tid in str_to_id.items():
                norm = s.replace("Ġ", " ")
                if remaining.startswith(norm) and norm:
                    valid.add(tid)

    return valid
```

---

## The `-inf` Trick

Negative infinity in Python's float system:

```python
import math
NEGATIVE_INF = -math.inf
```

When you set a logit to `-math.inf`, the softmax calculation gives it a probability of exactly 0 — it can **never** be selected.

```python
def apply_constraints(
    logits: list[float],
    valid_token_ids: set[int]
) -> list[float]:
    """Mask all tokens not in valid_token_ids to -inf."""
    constrained = logits.copy()

    for i in range(len(constrained)):
        if i not in valid_token_ids:
            constrained[i] = -math.inf

    return constrained
```

> **Watch out:** if `valid_token_ids` is empty, all logits become `-inf` and `argmax` will behave unpredictably. Always assert before applying constraints:

```python
assert len(valid_token_ids) > 0, (
    f"No valid tokens found at position '{partial_json}' — "
    f"check your state machine logic"
)
```

---

## Complete Walkthrough

Let's trace through generating the output for `"Greet john"` step by step.

**Function definition:**
```json
{"name": "fn_greet", "parameters": {"name": {"type": "string"}}}
```

**Expected output:**
```json
{"name": "fn_greet", "parameters": {"name": "john"}}
```

---

```
partial: ""          state: START
valid:   {tid for '{'}
model picks: "{"     (id=5476 in Qwen3 vocab)
→ partial: "{"

partial: "{"         state: AFTER_OPEN_BRACE
valid:   {tid for '"'}
model picks: '"'     (id=1)
→ partial: '{"'

partial: '{"'        state: IN_NAME_KEY
valid:   tokens continuing '"name"' from position 1
         → tids for "n", "na", "nam", "name"
model picks: token for "name"
→ partial: '{"name'

partial: '{"name'    state: IN_NAME_KEY
valid:   {tid for '"'}   ← closing the key
model picks: '"'
→ partial: '{"name"'

partial: '{"name"'   state: AFTER_NAME_KEY
valid:   {tids for ":", "Ġ:", ": "}
model picks: token for ": "
→ partial: '{"name": '

partial: '{"name": ' state: BEFORE_NAME_VALUE
valid:   {tid for '"'}
model picks: '"'
→ partial: '{"name": "'

partial: '{"name": "' state: IN_NAME_VALUE
valid: tokens continuing any valid function name from ""
  "fn_add_numbers" → tids for "fn", "fn_", "fn_a", ...
  "fn_greet"       → tids for "fn", "fn_", "fn_g", ...
  "fn_reverse_string" → ...

model sees "Greet john" → assigns highest logit to "fn_greet" prefix
model picks: token for "fn_greet"  (or "fn" then "_greet")
→ partial: '{"name": "fn_greet'

partial: '{"name": "fn_greet'   state: IN_NAME_VALUE (complete)
valid:   {tid for '"'}
model picks: '"'
→ partial: '{"name": "fn_greet"'

partial: '{"name": "fn_greet"'   state: AFTER_NAME_VALUE
valid:   {tid for ','}
model picks: ","
→ partial: '{"name": "fn_greet",'

... (similar process for '"parameters"' key, ':', '{') ...

→ partial: '{"name": "fn_greet", "parameters": {"'

state: IN_ARG_KEY
valid: tokens continuing '"name"'  ← only param for fn_greet is "name"
model picks: token for "name"
→ partial: '{"name": "fn_greet", "parameters": {"name'

→ partial: '{"name": "fn_greet", "parameters": {"name"'
→ partial: '{"name": "fn_greet", "parameters": {"name": '
→ partial: '{"name": "fn_greet", "parameters": {"name": "'

state: IN_ARG_VALUE_STR (type is "string")
valid: all printable non-quote tokens + closing '"'

model extracts "john" from "Greet john" → picks token for "john"
→ partial: '{"name": "fn_greet", "parameters": {"name": "john'

valid: {tid for '"'}   ← must close the string
model picks: '"'
→ partial: '{"name": "fn_greet", "parameters": {"name": "john"'

state: AFTER_ARG_VALUE
no more params → valid: {tid for '}'}
model picks: "}"
→ partial: '{"name": "fn_greet", "parameters": {"name": "john"}'

state: AFTER_PARAMS_CLOSE
valid: {tid for '}'}
model picks: "}"
→ partial: '{"name": "fn_greet", "parameters": {"name": "john"}}'

state: COMPLETE → stop generation

json.loads('{"name": "fn_greet", "parameters": {"name": "john"}}')
→ {"name": "fn_greet", "parameters": {"name": "john"}}  ✅
```

---

## Putting It All Together

```python
def get_valid_token_ids(
    partial: str,
    fn_def: FunctionDefinition,
    all_fn_names: list[str],
    str_to_id: dict[str, int],   # token_string → token_id
    state: JSONState,
    context: dict
) -> set[int]:
    """
    Central dispatch: given the current state and partial output,
    return the set of token IDs that are valid next tokens.
    """

    def find(target: str) -> set[int]:
        """Find token IDs whose normalized string equals target."""
        return {tid for s, tid in str_to_id.items()
                if s.replace("Ġ", " ") == target}

    def find_strip(target: str) -> set[int]:
        """Find token IDs whose stripped normalized string equals target."""
        return {tid for s, tid in str_to_id.items()
                if s.replace("Ġ", " ").strip() == target}

    if state == JSONState.START:
        return find_strip("{")

    if state == JSONState.AFTER_OPEN_BRACE:
        return find('"')

    if state == JSONState.IN_NAME_KEY:
        return tokens_continuing(str_to_id, '"name"', context["name_key_written"])

    if state == JSONState.AFTER_NAME_KEY:
        return find_strip(":")

    if state == JSONState.BEFORE_NAME_VALUE:
        return find('"')

    if state == JSONState.IN_NAME_VALUE:
        written = context["name_value_written"]
        return get_valid_name_tokens(str_to_id, all_fn_names, written)

    if state == JSONState.AFTER_NAME_VALUE:
        return find('"')   # closing quote of name value

    # ... continue for all states ...

    if state == JSONState.IN_ARG_VALUE_NUM:
        return get_valid_number_tokens(str_to_id, context["current_value"])

    if state == JSONState.IN_ARG_VALUE_STR:
        return get_valid_string_tokens(
            str_to_id=str_to_id,
            in_string=True,
            string_content=context["current_value"]
        )

    if state == JSONState.IN_ARG_VALUE_BOOL:
        return get_valid_boolean_tokens(str_to_id, context["current_value"])

    raise ValueError(f"Unhandled state: {state}")
```

---

## Common Mistakes

### ❌ Using `vocab[str(next_id)]` or `vocab[next_id]` to get the token string

The vocab format is `str → int`, not `int → str`. After picking `next_id`, you need the **reverse lookup**:

```python
# ❌ Wrong — vocab keys are strings, not IDs
token_str = vocab[str(next_id)]
token_str = vocab[next_id]

# ✅ Correct — use the pre-built reverse lookup
token_str = id_to_str[next_id]
```

---

### ❌ Forgetting `.tolist()` on encode output

`model.encode()` returns a **2D tensor**, not a list of ints:

```python
# ❌ Wrong — returns a 2D tensor
input_ids = model.encode(prompt)

# ✅ Correct — flatten to a list of ints
input_ids = model.encode(prompt)[0].tolist()
```

---

### ❌ Searching the whole vocabulary in the hot loop

Pre-compute fixed sets at startup. Only do dynamic searches (function name prefixes, param name prefixes) when in those specific states.

---

### ❌ Assuming tokens are single characters

A token like `"fn_greet"` might be emitted as one token `fn_greet`, or as `fn` + `_greet`. Your prefix-matching logic must handle all cases:

```python
# ❌ Wrong: assuming 'fn_greet' is always one token
valid = {tid for s, tid in str_to_id.items() if s == "fn_greet"}

# ✅ Correct: allow any token that continues what's needed
def tokens_continuing(str_to_id, target: str, written: str) -> set[int]:
    remaining = target[len(written):]
    return {tid for s, tid in str_to_id.items()
            if remaining.startswith(s.replace("Ġ", " ")) and s.replace("Ġ", " ")}
```

---

### ❌ Not handling the closing `}` of the root object

After the parameters object closes with `}`, you still need one more `}` to close the root object. Make sure your state machine has a state for this.

---

### ❌ Hardcoding function names

Your solution must work with any `functions_definition.json`. Never hardcode `"fn_add_numbers"` — always read function names dynamically from the loaded schema.

---

*See also: [`TOKENIZATION.md`](./TOKENIZATION.md) for how the vocabulary file is structured and how to handle BPE token edge cases.*
*See also: [`FUNCTION_CALLING.md`](./FUNCTION_CALLING.md) for the full input/output format and schema structure.*
