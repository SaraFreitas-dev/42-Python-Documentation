# 🖥️ CLI & Argument Parsing — How the Program Accepts Input

> **What this doc covers:** What `argparse` is, why it exists, how it works in this project, and how the three arguments (`--functions_definition`, `--input`, `--output`) map to the program's execution flow.

---

## Table of Contents

1. [What is argparse?](#what-is-argparse)
2. [Why Not Hardcode the Paths?](#why-not-hardcode-the-paths)
3. [How argparse Works](#how-argparse-works)
4. [The Three Arguments in This Project](#the-three-arguments-in-this-project)
5. [Default Values](#default-values)
6. [How Arguments Flow Through the Program](#how-arguments-flow-through-the-program)
7. [Running the Program](#running-the-program)

---

## What is argparse?

`argparse` is Python's standard library module for parsing command-line arguments. It lets your program accept input from the terminal when it's launched, instead of having paths or settings hardcoded inside the code.

```
uv run python -m src --input data/input/tests.json
                      ↑
                      argparse reads this
```

It handles three things automatically:
- Parsing the arguments the user passes
- Providing default values when arguments are omitted
- Generating a `--help` message that describes the program

---

## Why Not Hardcode the Paths?

You could write this inside your code:

```python
# Hardcoded — bad
fn_defs = load_function_definitions("data/input/functions_definition.json")
prompts = load_test_prompts("data/input/function_calling_tests.json")
```

This works locally, but it breaks the moment someone wants to run the program with different files — which is exactly what the peer reviewer will do. The subject explicitly states:

> *"The given input files may change during the peer review. Do not hardcode solutions based on the provided examples."*

With argparse, the paths come from the command line — the reviewer can pass any files they want without touching your code.

---

## How argparse Works

The basic pattern is always the same:

```python
import argparse

def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(description="...")

    parser.add_argument(
        "--argument-name",
        type=str,
        default="default/value",
        help="What this argument does"
    )

    return parser.parse_args()
```

Then in `main()`:

```python
def main() -> None:
    args = parse_args()
    print(args.argument_name)   # access with dot notation
```

### Key parameters of `add_argument`

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `"--name"` | The flag name (with `--`) | `"--input"` |
| `type` | Converts the string value to a Python type | `type=str` |
| `default` | Value used when the argument is not passed | `default="data/input/tests.json"` |
| `help` | Description shown in `--help` output | `help="Path to input file"` |

### Accessing values

After `parse_args()`, you access arguments with dot notation. Dashes in argument names become underscores:

```
--functions_definition  →  args.functions_definition
--input                 →  args.input
--output                →  args.output
```

---

## The Three Arguments in This Project

The subject specifies exactly how the program must be called:

```bash
uv run python -m src \
  --functions_definition data/input/functions_definition.json \
  --input data/input/function_calling_tests.json \
  --output data/output/function_calling_results.json
```

This means three arguments are needed:

### `--functions_definition`

The path to the JSON file that defines the available functions and their schemas.

```
Default: data/input/functions_definition.json
Used by: load_function_definitions() in data_loader.py
```

### `--input`

The path to the JSON file containing the natural language prompts to process.

```
Default: data/input/function_calling_tests.json
Used by: load_test_prompts() in data_loader.py
```

### `--output`

The path where the program writes the generated function calls JSON file.

```
Default: data/output/function_calling_results.json
Used by: the output writing logic in __main__.py
```

---

## Default Values

The subject says:

> *"By default, the program will read input files from the `data/input/` directory and write output to the `data/output/` directory."*

This means **all three arguments are optional** — if the user runs the program without any flags, it falls back to the default paths and still works correctly:

```bash
uv run python -m src                    # uses all defaults ✅
uv run python -m src --input other.json # overrides just --input ✅
```

This is exactly what `default=` in `add_argument` provides.

---

## How Arguments Flow Through the Program

```
Command line:
uv run python -m src --input data/input/tests.json
                                    ↓
                            parse_args()
                            returns args.input = "data/input/tests.json"
                                    ↓
                            main(args)
                                    ↓
                 load_test_prompts(args.input)      ← data_loader.py
                 load_function_definitions(args.functions_definition)
                                    ↓
                         constrained_decoder
                                    ↓
                    write output to args.output
```

The arguments are read once at startup and passed through to the functions that need them. Nothing inside `data_loader.py`, `llm_engine.py`, or `constrained_decoder.py` reads from `sys.argv` directly — they all receive paths as regular function parameters.

---

## Running the Program

### With all defaults

```bash
uv run python -m src
```

### With custom paths

```bash
uv run python -m src \
  --functions_definition data/input/functions_definition.json \
  --input data/input/function_calling_tests.json \
  --output data/output/function_calling_results.json
```

### Via the Makefile

```bash
make run
```

The `Makefile` already has the default paths configured:

```makefile
FUNCTIONS = data/input/functions_definition.json
INPUT     = data/input/function_calling_tests.json
OUTPUT    = data/output/function_calling_results.json

run:
	uv run python -m src \
		--functions_definition $(FUNCTIONS) \
		--input $(INPUT) \
		--output $(OUTPUT)
```

### Getting help

```bash
uv run python -m src --help
```

argparse generates this automatically from the `help=` strings you provide in each `add_argument` call.

---

*See also: [`UV_GUIDE.md`](./UV_GUIDE.md) for how `uv run` works with the project environment.*
*Official argparse docs: [docs.python.org/3/library/argparse.html](https://docs.python.org/3/library/argparse.html)*
