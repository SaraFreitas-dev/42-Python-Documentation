# 🤗 LLM SDK & Hugging Face — What's Running Under the Hood

> **Why this doc exists:** You never write `import transformers` or `import torch` directly in your `src/` code — but the `llm_sdk` package does, and understanding what it's doing makes debugging (and the first slow run!) much less mysterious. This doc explains Hugging Face, PyTorch basics, and walks through the actual `llm_sdk/__init__.py` line by line.

---

## Table of Contents

1. [What is Hugging Face?](#what-is-hugging-face)
2. [The Hugging Face Hub](#the-hugging-face-hub)
3. [AutoModelForCausalLM & AutoTokenizer](#automodelforcausallm--autotokenizer)
4. [PyTorch — The Bare Minimum](#pytorch--the-bare-minimum)
5. [Walking Through llm_sdk/__init__.py](#walking-through-llm_sdk__init__py)
6. [Why the First Run Is Slow](#why-the-first-run-is-slow)
7. [Where Files Get Cached](#where-files-get-cached)
8. [What You're Forbidden From Using Directly](#what-youre-forbidden-from-using-directly)

---

## What is Hugging Face?

**Hugging Face** is a company and open-source ecosystem that hosts trained AI models, datasets, and the Python libraries used to load and run them. Think of it like GitHub, but specifically for AI models.

Three things Hugging Face provides that matter for this project:

| Thing | What it is |
|-------|-----------|
| **The Hub** | A website + API where models are hosted (huggingface.co) |
| **`transformers`** | The Python library that loads and runs models like Qwen3 |
| **`huggingface_hub`** | The Python library that downloads files from the Hub |

You don't interact with any of these directly in your `src/` code — they're forbidden by the subject. But `llm_sdk` uses all three internally.

---

## The Hugging Face Hub

The Hub is where **Qwen/Qwen3-0.6B** actually lives: [huggingface.co/Qwen/Qwen3-0.6B](https://huggingface.co/Qwen/Qwen3-0.6B)

### What "Qwen/Qwen3-0.6B" means

This string is a **repo ID** — it identifies a specific model repository on the Hub, in the format `organization/model-name`:

```
Qwen/Qwen3-0.6B
 │      │
 │      └── model name
 └── organization that published it (Alibaba's Qwen team)
```

### What's inside a model repo

A typical Hugging Face model repo contains several files:

```
Qwen/Qwen3-0.6B/
├── config.json              ← model architecture settings (layers, hidden size, etc.)
├── model.safetensors        ← the actual trained weights (the "brain")
├── tokenizer.json           ← fast tokenizer data
├── tokenizer_config.json    ← tokenizer settings
├── vocab.json               ← the vocabulary file you load with get_path_to_vocab_file()
├── merges.txt               ← BPE merge rules (used during tokenizer training)
└── special_tokens_map.json  ← maps special token names to their string forms
```

This is why `get_path_to_vocab_file()` and `get_path_to_merges_file()` in `llm_sdk` exist — they're downloading specific files out of this repo for you to inspect directly.

---

## AutoModelForCausalLM & AutoTokenizer

These are the two main classes from the `transformers` library that `llm_sdk` uses to load the model. You'll see them in the SDK's `__init__.py`.

### `AutoTokenizer`

Loads the **tokenizer** for a given model — the component that converts text ↔ token IDs (covered in detail in [`TOKENIZATION.md`](./TOKENIZATION.md)).

```python
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen3-0.6B")
```

The word "Auto" means it automatically figures out *which kind* of tokenizer to load based on the model's config — you don't need to know if it's BPE, SentencePiece, or something else.

### `AutoModelForCausalLM`

Loads the **model weights** for causal language modelling — the "predict the next token" task that LLMs do.

```python
model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen3-0.6B")
```

"Causal" means the model only looks at *previous* tokens to predict the next one — it can't see into the future of the sequence. This is the standard setup for text-generation models like Qwen, GPT, and Llama.

### Why "Auto"?

Both classes are smart wrappers — given just the repo ID, they read the `config.json` from the Hub and instantiate the correct underlying Python class automatically. This is why `llm_sdk` can support "other models as long as your project works with Qwen3-0.6B" (per the bonus part) — swapping the `model_name` string is often enough.

---

## PyTorch — The Bare Minimum

You don't write PyTorch code in this project, but `llm_sdk` is built on it, and a few concepts will help you read its source and debug issues.

### Tensors

A `torch.Tensor` is PyTorch's array type — similar to a numpy array, but capable of running on GPUs and tracking gradients (not relevant here since we're only doing inference).

This is why `model.encode()` doesn't return a plain Python list:

```python
ids = model.encode("hello")
print(type(ids))   # <class 'torch.Tensor'>
print(ids.shape)   # torch.Size([1, 2])  ← 2D! batch dimension + sequence dimension
```

To get a plain Python list of ints, you convert it:

```python
ids_list = ids[0].tolist()   # [0] removes the batch dimension, tolist() converts
```

> **This is a common bug source.** If you forget `[0].tolist()` and try to use the tensor directly as a list of IDs, you'll get confusing errors or wrong behaviour. Always convert immediately after calling `encode()`.

### Device: cpu / cuda / mps

PyTorch can run computations on different hardware:

| Device | Meaning |
|--------|---------|
| `"cpu"` | Regular processor — always available, slower |
| `"cuda"` | NVIDIA GPU — much faster, requires NVIDIA drivers + CUDA installed |
| `"mps"` | Apple Silicon GPU (M1/M2/M3 Macs) — fast on Mac |

The `llm_sdk` auto-detects which is available:

```python
if torch.backends.mps.is_available():
    device = "mps"
elif torch.cuda.is_available():
    device = "cuda"
else:
    device = "cpu"
```

If you saw a warning like *"CUDA initialization: The NVIDIA driver on your system is too old"* — that's PyTorch trying `cuda`, failing gracefully, and falling back to `cpu`. It's just a warning, not an error; the model still runs, just slower.

### `torch.no_grad()`

```python
with torch.no_grad():
    out = self._model(input_ids=input_tensor)
```

During training, PyTorch tracks every operation so it can compute gradients later (used to update the model's weights). Since this project only runs **inference** — using the model, not training it — gradient tracking is unnecessary overhead. `torch.no_grad()` turns it off, making inference faster and using less memory.

### dtype: float16 vs float32

```python
dtype = torch.float16 if self._device in ["cuda", "mps"] else torch.float32
```

This controls the numerical precision used for the model's weights:

- `float32` — standard precision, more memory, used on CPU for compatibility
- `float16` — half precision, less memory, faster on GPU, tiny accuracy trade-off

You don't need to change this — `llm_sdk` picks the right one automatically based on your device.

---

## Walking Through llm_sdk/__init__.py

Here's what each part of the provided SDK actually does, mapped to the concepts above.

### Initialization

```python
def __init__(self, model_name: str = "Qwen/Qwen3-0.6B", ...):
    self._tokenizer = AutoTokenizer.from_pretrained(model_name, ...)
    self._model = AutoModelForCausalLM.from_pretrained(model_name, ...)
    self._model.to(self._device)
    self._model.eval()
```

1. Downloads (or loads from cache) the tokenizer for `Qwen/Qwen3-0.6B`
2. Downloads (or loads from cache) the model weights
3. Moves the model onto the selected device (cpu/cuda/mps)
4. Switches to `eval()` mode — disables training-specific behaviours like dropout, ensuring consistent output

### `encode()`

```python
def encode(self, text: str) -> torch.Tensor:
    ids = self._tokenizer.encode(text, add_special_tokens=False)
    return torch.tensor([ids], device=self._device, dtype=torch.long)
```

- `add_special_tokens=False` — doesn't automatically insert things like `<|im_start|>` for you; you control the prompt format yourself
- `[ids]` — wraps the list in another list, creating the 2D shape `(1, sequence_length)` — the `1` is the **batch size** (you're only ever processing one prompt at a time, hence batch size 1)

### `get_logits_from_input_ids()`

```python
def get_logits_from_input_ids(self, input_ids: list[int]) -> list[float]:
    input_tensor = torch.tensor([input_ids], device=self._device, dtype=torch.long)
    with torch.no_grad():
        out = self._model(input_ids=input_tensor)
    logits = out.logits[0, -1].tolist()
    return [float(x) for x in logits]
```

Breaking this down:

- `torch.tensor([input_ids], ...)` — same batch-dimension wrapping as `encode()`
- `self._model(input_ids=input_tensor)` — runs a forward pass through the neural network
- `out.logits` — shape is `(batch, sequence_length, vocab_size)` — logits for *every position* in the sequence, for *every token* in the vocabulary
- `out.logits[0, -1]` — `[0]` takes the first (only) batch item, `[-1]` takes the **last position** — i.e. the prediction for what comes *next*, after everything you fed in
- `.tolist()` — converts the tensor back to a plain Python list

This is the method you call at every step of your constrained decoding loop.

### `get_path_to_vocab_file()`

```python
def get_path_to_vocab_file(self) -> str:
    vocab_file_name = self._tokenizer.vocab_files_names.get('vocab_file', "vocab.json")
    vocab_path = hf_hub_download(repo_id=self._model_name, filename=vocab_file_name)
    return vocab_path
```

`hf_hub_download` is a function from `huggingface_hub` that downloads a specific file from a specific repo on the Hub — and **caches it locally**, so subsequent calls don't re-download.

---

## Why the First Run Is Slow

The first time you run anything that creates a `Small_LLM_Model()`, it needs to download:

- The model weights (`model.safetensors`) — roughly 1.2GB for Qwen3-0.6B
- The tokenizer files (`tokenizer.json`, `vocab.json`, `merges.txt`)
- The config files

This can take a few minutes depending on your connection. **Subsequent runs are fast** because everything is cached locally — the SDK won't re-download files it already has.

```
First run:   uv run python3 test.py    →  ⏳ downloading... (1-5 minutes)
Second run:  uv run python3 test.py    →  ⚡ instant (loads from cache)
```

If you see no output for a while on the first run, that's expected — it's downloading in the background.

---

## Where Files Get Cached

By default, Hugging Face caches downloaded files in:

```
Linux/macOS:  ~/.cache/huggingface/hub/
Windows:      C:\Users\<you>\.cache\huggingface\hub\
```

Inside, you'll find a folder named after the repo:

```
~/.cache/huggingface/hub/models--Qwen--Qwen3-0.6B/
```

If something seems broken (corrupted download, wrong version), you can safely delete this folder and let it re-download on the next run. This is also useful to know for your `.gitignore` — these files should **never** be committed to your repository; they're large binary downloads, not your code.

---

## What You're Forbidden From Using Directly

The subject is explicit about this — re-stating it here because it's directly tied to everything above:

> *"The use of dspy (or any similar package) is completely forbidden including pytorch, huggingface package, transformers, outlines, etc."*

This means in your `src/` code:

```python
# ❌ Forbidden in src/
import torch
import transformers
from transformers import AutoModelForCausalLM

# ✅ Allowed — going through the provided SDK
from llm_sdk import Small_LLM_Model
model = Small_LLM_Model()
model.encode(...)
model.get_logits_from_input_ids(...)
```

You only ever touch `llm_sdk`'s public methods. Everything in this document is **background knowledge** to help you understand what's happening when you call those methods — not something you implement or import yourself.

> Also forbidden: any private methods or attributes from `llm_sdk` (anything starting with `_`, like `self._model` or `self._tokenizer`). Stick to the public interface: `encode`, `decode`, `get_logits_from_input_ids`, `get_path_to_vocab_file`, `get_path_to_merges_file`, `get_path_to_tokenizer_file`.

---

*See also: [`LLM_GUIDE.md`](./LLM_GUIDE.md) for the conceptual explanation of LLMs, tokens, and logits.*
*See also: [`TOKENIZATION.md`](./TOKENIZATION.md) for how the vocabulary file is structured and used.*
*Official docs: [huggingface.co/docs/transformers](https://huggingface.co/docs/transformers), [pytorch.org/docs](https://pytorch.org/docs)*
