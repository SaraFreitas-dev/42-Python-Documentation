# 🤖 LLMs & Function Calling — How Call Me Maybe Works

> **Why this doc exists:** Call Me Maybe is not a chatbot. It's a **function calling system** — a pipeline that reads a natural language prompt and outputs a structured JSON function call. This guide explains what LLMs are, how they generate text token by token, and why this project's approach (constrained decoding) exists and matters.

---

## Table of Contents

1. [What is an LLM?](#what-is-an-llm)
2. [How Text Generation Actually Works](#how-text-generation-actually-works)
3. [Tokens — The Building Blocks](#tokens--the-building-blocks)
4. [Logits — The Model's Raw Output](#logits--the-models-raw-output)
5. [What is Function Calling?](#what-is-function-calling)
6. [The Problem: Small Models Are Unreliable](#the-problem-small-models-are-unreliable)
7. [The Solution: Constrained Decoding](#the-solution-constrained-decoding)
8. [The Model Used: Qwen3-0.6B](#the-model-used-qwen3-06b)
9. [The llm_sdk Package](#the-llm_sdk-package)
10. [The Full Pipeline in This Project](#the-full-pipeline-in-this-project)
11. [Glossary](#glossary)

---

## What is an LLM?

A **Large Language Model** is an AI trained on enormous amounts of text. During training, it adjusted billions of internal parameters (weights) to learn one core skill: **predicting the most likely next token given everything that came before.**

That's the whole trick. From that single ability, you get something that can answer questions, write code, summarise text — and in this project, decide which function to call and with what arguments.

### 🏛️ Some well-known LLMs

| Model | Creator | Parameters | Notes |
|-------|---------|-----------|-------|
| GPT-4o | OpenAI | ~1 trillion (est.) | Powers ChatGPT |
| Claude 3.5 Sonnet | Anthropic | Unknown | Strong reasoning |
| Llama 3.3 | Meta | 70B | Open source |
| **Qwen3-0.6B** | **Alibaba** | **600 million** | **Used in this project** |

> **In this project** we use **Qwen/Qwen3-0.6B** — a small, local model that runs entirely on your machine via the provided `llm_sdk` package. No API calls, no internet required.

---

## How Text Generation Actually Works

Understanding this is essential for this project. The generation loop is what you'll be hooking into with constrained decoding.

### The generation loop

Every time an LLM produces text, it runs this cycle repeatedly:

```
1. Take all tokens so far (prompt + already generated tokens)
2. Feed them through the neural network
3. Get back a probability score for every possible next token (logits)
4. Pick the next token
5. Append it to the sequence
6. Repeat from step 1 until done
```

Here's what that looks like for a simple input:

```
Input:  "What is the sum of 2 and 3?"

Step 1: model sees → ["What", " is", " the", " sum", " of", " 2", " and", " 3", "?"]
Step 3: logits → { "{": 0.72, "The": 0.04, "5": 0.19, ... }   ← "{" wins
Step 5: sequence → ["What", ..., "?", "{"]

Step 1: model sees → ["What", ..., "?", "{"]
Step 3: logits → { '"': 0.91, " ": 0.03, "f": 0.02, ... }     ← '"' wins
Step 5: sequence → ["What", ..., "?", "{", '"']

... and so on until the full JSON is generated
```

Each token is chosen one at a time. The model has no concept of "the whole output" — it only ever decides: *given everything so far, what comes next?*

---

## Tokens — The Building Blocks 🧩

LLMs don't process characters or words — they process **tokens**. A token is a chunk of text, typically 3–4 characters on average, defined by the model's **tokenizer**.

### Why not just use words?

Tokenizers use algorithms like **BPE (Byte Pair Encoding)** that split text into statistically efficient chunks. Common words become single tokens; rare words get split.

```
"hello"          → ["hello"]                   = 1 token
"pyproject"      → ["py", "project"]           = 2 tokens
"fn_add_numbers" → ["fn", "_add", "_numbers"]  = 3 tokens
"Qwen"           → ["Qwen"]                    = 1 token
```

### The vocabulary

Every model has a fixed **vocabulary** — a list of every possible token and its integer ID. Qwen3-0.6B's vocabulary contains ~150,000 entries. At each generation step, the model assigns a score to all ~150,000 tokens.

> **This matters for constrained decoding:** the vocabulary file (`get_path_to_vocab_file()`) is how you know which token IDs correspond to characters like `{`, `"`, `1`, `2`, `.`, etc. — and therefore which tokens are valid JSON at each point in the generation.

### Tokens have leading spaces

Tokenizers preserve spacing. The token for ` the` (with a leading space) is different from `the` (without). When you inspect the vocabulary file, you'll see tokens like `Ġthe`, `Ġsum`, `Ġof` — where `Ġ` represents a preceding space.

```
"the sum"  →  ["the", "Ġsum"]    ← "Ġsum" includes the space before it
```

This matters when matching tokens to partial JSON strings during constrained decoding.

---

## Logits — The Model's Raw Output 📊

After processing the input, the model outputs **logits** — one raw score per vocabulary token. These are not probabilities yet; they're raw numbers that can be positive or negative.

```python
# Simplified example: vocabulary of 5 tokens
logits = {
    "{":    2.8,    # high → likely
    "The":  0.1,    # low → unlikely
    "5":    1.2,    # medium
    "null": -3.4,   # very low → very unlikely
    "fn":   0.6,
}
```

To convert logits to probabilities, you apply **softmax**:

```
probability(token) = exp(logit) / sum(exp(all logits))
```

The token with the highest logit wins (greedy decoding), or you can sample proportionally.

### The key insight for this project

**You can modify logits before the token is selected.** This is constrained decoding. If you set a token's logit to `-inf` (negative infinity), it effectively has zero probability of being chosen:

```python
import math

logits[" The"] = -math.inf   # this token can never be chosen
logits["5"]    = -math.inf   # this token can never be chosen
# now only valid JSON tokens remain in the distribution
```

---

## What is Function Calling? 📞

LLMs are great at generating natural language — but programs need structured, machine-readable output. **Function calling** bridges this gap.

The idea: given a natural language request and a set of available functions, the LLM determines which function to call and with what arguments — outputting structured JSON instead of a prose answer.

```
User: "What is the sum of 40 and 2?"

❌ Traditional LLM output:
   "The sum of 40 and 2 is 42."          ← useful for humans, useless for code

✅ Function calling output:
   {
     "name": "fn_add_numbers",
     "parameters": {"a": 40.0, "b": 2.0}
   }                                       ← your code can actually use this
```

Another example:

```
User: "Greet John"

Output:
{
  "name": "fn_greet",
  "parameters": {"name": "John"}
}
```

The model doesn't execute the function. It just figures out *which* function to call and *what arguments* to pass. Your Python code does the actual execution.

### Why is this powerful?

Function calling lets LLMs interact with real systems — call APIs, query databases, trigger actions — by producing output that software can parse and act on. It's how modern AI assistants ("check my calendar", "send this email", "set a reminder") actually work under the hood.

---

## The Problem: Small Models Are Unreliable ⚠️

Here's the honest truth about small models when asked to produce JSON:

```
Qwen3-0.6B prompted to output JSON: succeeds ~30% of the time
GPT-4o prompted to output JSON:     succeeds ~95% of the time
```

Small models (like the 0.6B you're using) frequently:
- Forget to close brackets
- Add prose explanations after the JSON
- Use wrong types (`"42"` instead of `42.0`)
- Hallucinate argument names that don't exist in the schema
- Output partial JSON before giving up

```
# What you asked for:
{"name": "fn_add_numbers", "parameters": {"a": 40.0, "b": 2.0}}

# What a small model might actually produce:
{"name": "fn_add_numbers", "parameters": {"a": "40", "b": "2"}}
# Wrong types! ↑                                      ↑

# Or even:
{"name": "fn_add_numbers", "parameters": {"a": 40, "b": 2}
# Missing closing brace! ↑

# Or:
The function to call is fn_add_numbers with arguments a=40 and b=2.
# Not JSON at all!
```

**The subject is explicit about this:** your solution must NOT rely on prompting the model and hoping it produces correct JSON. That's not reliable, and it's not the skill being tested.

---

## The Solution: Constrained Decoding 🔒

Constrained decoding solves the reliability problem at the **generation level** — not at the prompting level. Instead of hoping the model produces valid JSON, you **guarantee it** by controlling which tokens it's allowed to choose at each step.

### The core idea

At every token generation step:

1. Get logits for all ~150,000 tokens from the model
2. Determine which tokens would produce **valid JSON at this position** given the schema
3. Set all invalid token logits to `-inf`
4. Sample from the remaining valid tokens

The model still drives the content (which function name, which argument values), but it physically cannot produce invalid JSON.

### A concrete walkthrough

Say you're generating the output for `"Greet John"`. The schema requires:

```json
{"name": "...", "parameters": {"name": "..."}}
```

**Step 1: Must start with `{`**

```
Valid tokens:   ["{"]
Invalid tokens: everything else → set to -inf
Chosen token:   "{"
```

**Step 2: After `{`, must have `"`**

```
Current partial: "{"
Valid tokens:    ['"']
Chosen token:   '"'
```

**Step 3: The key field must be `name`**

```
Current partial: '{"'
Valid tokens:   ["n"]   ← only 'n' continues a valid field name here
Chosen token:   "n"
```

**Step 4: Filling in the function name**

Once you're inside the `"name"` value, only tokens that continue valid function names are allowed:

```
Current partial: '{"name": "'
Valid tokens: ["fn_add_numbers", "fn_greet", "fn_reverse_string", ...]
              ← restricted to actual functions from functions_definition.json
```

The model assigns higher probability to `fn_greet` given the prompt "Greet John" — and since only valid function names are available, it correctly picks `fn_greet`.

**Step 5: For numeric arguments, only numbers allowed**

If the schema says a parameter is of type `number`:

```
Current partial: '{"name": "fn_add_numbers", "parameters": {"a": '
Valid tokens:    ["0","1","2","3","4","5","6","7","8","9","-"]
                 ← strings, booleans, null are all set to -inf
```

### Result

```
Final output: {"name": "fn_greet", "parameters": {"name": "John"}}
```

100% valid JSON. 100% schema-compliant. Every time.

---

## The Model Used: Qwen3-0.6B 🏎️

**Qwen3-0.6B** is a small open-source language model from Alibaba. The `0.6B` means it has approximately **600 million parameters** — tiny by modern standards (GPT-4 is estimated at ~1 trillion), but more than enough for function calling when guided by constrained decoding.

### Why a small model?

- Runs locally on standard hardware (no GPU required, though it helps)
- No API calls, no costs, no internet dependency
- The project is specifically designed to show that **structural guidance matters more than raw model size**
- With constrained decoding, a 0.6B model can achieve reliability comparable to models 100x larger

### What the model can and cannot do

| ✅ Can do | ❌ Cannot do |
|----------|-------------|
| Understand natural language intent | Reliably produce structured JSON without guidance |
| Choose the right function from context | Remember things between separate calls |
| Extract argument values from text | Know things after its training cutoff |
| Follow semantic patterns | Guarantee valid output format on its own |

The model is strong at **understanding**. Constrained decoding handles the **formatting**.

---

## The llm_sdk Package 📦

The project provides a `llm_sdk` package with a `Small_LLM_Model` class. This is your interface to the Qwen model. You copy it into your project directory and use it directly.

### Available methods

```python
from llm_sdk import Small_LLM_Model

model = Small_LLM_Model()
```

**`get_logits_from_input_ids(input_ids: List[int]) -> List[float]`**

The core method. Pass a list of token IDs (your prompt + tokens generated so far), get back logits for every token in the vocabulary.

```python
# Encode your prompt to token IDs first
input_ids = [892, 318, 262, 4771, 286, 16, 290, 17, 30]  # "What is the sum of 2 and 3?"

logits = model.get_logits_from_input_ids(input_ids)
# logits[i] = score for vocabulary token i
# len(logits) == vocabulary size (~150,000)
```

**`get_path_to_vocab_file() -> str`**

Returns the path to a JSON file mapping token IDs to their string representations. You need this to know which token ID corresponds to `{`, `"`, `1`, `2`, etc.

```python
import json

vocab_path = model.get_path_to_vocab_file()
with open(vocab_path) as f:
    vocab = json.load(f)

# vocab might look like:
# {"0": "!", "1": '"', "2": "#", ..., "5476": "{", ...}
# Use this to find which IDs are valid at each JSON position
```

**`encode(text: str) -> Tensor`**

Converts a text string into token IDs. Use this to tokenise your prompt before passing it to `get_logits_from_input_ids`.

```python
token_ids = model.encode("What is the sum of 2 and 3?")
# Returns a tensor of integer IDs
```

**`decode(token_ids: List[int]) -> str`** *(optional)*

Converts token IDs back to text. Useful for debugging or the bonus part.

```python
text = model.decode([892, 318, 262])
# → "What is the"
```

> **Important:** The subject forbids using any private methods or attributes from `llm_sdk` (those starting with `_`). Only use the public methods listed above.

---

## The Full Pipeline in This Project 🔄

Here's how all the pieces connect for a single prompt:

```
1. READ INPUT
   Load functions_definition.json → know available functions & their schemas
   Load function_calling_tests.json → list of natural language prompts

2. FOR EACH PROMPT (e.g. "What is the sum of 2 and 3?"):

   a. CONSTRUCT PROMPT
      Build a text prompt that includes the function definitions
      and the user's natural language request

   b. TOKENISE
      token_ids = model.encode(prompt)

   c. CONSTRAINED GENERATION LOOP
      partial_json = ""

      while not complete:
          logits = model.get_logits_from_input_ids(token_ids)

          valid_token_ids = get_valid_tokens(partial_json, schema)
          # ↑ your constrained decoding logic:
          # which tokens can legally follow what we've generated so far?

          for i, token in enumerate(vocabulary):
              if i not in valid_token_ids:
                  logits[i] = -inf

          next_token_id = argmax(logits)   # pick highest remaining score
          token_ids.append(next_token_id)
          partial_json += vocabulary[next_token_id]

   d. PARSE OUTPUT
      result = json.loads(partial_json)
      # always valid because constrained decoding guaranteed it

3. WRITE OUTPUT
   Write all results to data/output/function_calling_results.json
```

### Example end-to-end

```
Input prompt:  "What is the sum of 2 and 3?"
Available functions: [fn_add_numbers(a: number, b: number), fn_greet(name: string), ...]

Constrained generation produces:
{"name": "fn_add_numbers", "parameters": {"a": 2.0, "b": 3.0}}

Final output entry:
{
  "prompt": "What is the sum of 2 and 3?",
  "name": "fn_add_numbers",
  "parameters": {"a": 2.0, "b": 3.0}
}
```

---

## Glossary 📖

| Term | Definition |
|------|-----------|
| **LLM** | Large Language Model — AI trained on text to predict/generate language |
| **Token** | A chunk of text (~3–4 chars) that the model processes as a single unit |
| **Tokenizer** | The component that splits text into tokens and assigns them integer IDs |
| **Vocabulary** | The complete list of all tokens the model knows (~150,000 for Qwen3) |
| **Logits** | Raw scores the model outputs for every token in the vocabulary at each step |
| **Softmax** | Mathematical function that converts logits into probabilities (sum to 1) |
| **Greedy decoding** | Always picking the token with the highest logit (deterministic) |
| **Constrained decoding** | Forcing invalid tokens to -inf before selection, guaranteeing valid structure |
| **Function calling** | Using an LLM to output structured function call JSON instead of prose |
| **Schema** | The definition of a function's name, parameter names, and parameter types |
| **BPE** | Byte Pair Encoding — the algorithm most modern tokenizers use |
| **Parameters (model)** | The billions of learned weights that make up the neural network |
| **Parameters (function)** | The arguments a function takes (different meaning — context matters!) |
| **Hallucination** | When a model confidently generates plausible but incorrect content |
| **Inference** | Running the model to generate output (vs. training, which sets the weights) |

---

*Official Qwen model page: [huggingface.co/Qwen/Qwen3-0.6B](https://huggingface.co/Qwen/Qwen3-0.6B)*
*Constrained decoding deep dive: [arxiv.org/abs/2307.09702](https://arxiv.org/abs/2307.09702)*
