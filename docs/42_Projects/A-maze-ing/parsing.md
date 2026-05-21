# Parsing System

This document explains the configuration parsing system used in the **A-Maze-ing** project.

The parser is responsible for:
- Reading maze configuration files
- Validating required parameters
- Converting values to proper Python types
- Detecting invalid or duplicated data
- Ensuring the maze configuration is valid before generation

---

# 📁 Main File

```text
parsing/
└── config_parser.py
```

---

# 🧠 Overview

The parser uses a simple `KEY=VALUE` configuration format.

Example:

```text
WIDTH=20
HEIGHT=20
ENTRY=0,0
EXIT=19,19
OUTPUT_FILE=output_maze.txt
PERFECT=True
SEED=42
```

The configuration is:
- read from a `.txt` file
- converted into a Python dictionary
- validated
- transformed into usable Python types

---

# ⚙️ Main Responsibilities

The parsing system performs several validation and transformation steps:

- File loading
- Syntax validation
- Required key validation
- Duplicate key detection
- Type conversion
- Semantic validation

---

# 📥 Configuration File Path

The parser first checks if a configuration file was provided through command-line arguments.

```python
def get_config_path() -> str:
```

### Behavior

- If a `.txt` file path is provided:
  - it is used as the configuration source
- If no argument is provided:
  - an exception is raised

---

# 🔑 Mandatory Keys Validation

The parser validates that all required configuration keys exist.

Required keys:

```text
WIDTH
HEIGHT
ENTRY
EXIT
OUTPUT_FILE
PERFECT
```

If one or more keys are missing:

```text
ConfigError:
Missing required key(s)
```

is raised.

---

# 📝 Configuration Parsing

The main parsing function is:

```python
def parse_config(file_path: str) -> dict[str, Any]:
```

This function:
- opens the configuration file
- reads it line by line
- validates syntax
- stores values in a dictionary

---

# ✨ Supported Features

## Empty Lines

Empty lines are ignored.

Example:

```text
WIDTH=20

HEIGHT=20
```

---

## Comments

Lines starting with `#` are ignored.

Example:

```text
# Maze configuration
WIDTH=20
```

---

## Duplicate Key Detection

If the same key appears twice:

```text
WIDTH=20
WIDTH=30
```

the parser raises:

```text
Duplicate key 'WIDTH'
```

---

## Invalid Format Detection

If a line does not contain `=`:

```text
WIDTH 20
```

the parser raises:

```text
Invalid format on line X
```

---

# 🔄 Type Conversion System

After parsing the raw strings, the values are converted into proper Python types.

Main conversion function:

```python
def convert_config(config: dict[str, Any]) -> dict[str, Any]:
```

---

# 🔢 Integer Conversion

The following keys are converted to integers:

```text
WIDTH
HEIGHT
SEED
```

Example:

```text
WIDTH=20
```

becomes:

```python
20
```

---

# 📍 Coordinate Conversion

The following keys are converted into tuples:

```text
ENTRY
EXIT
```

Example:

```text
ENTRY=0,0
```

becomes:

```python
(0, 0)
```

---

# ✅ Boolean Conversion

The `PERFECT` key is converted into a boolean value.

Example:

```text
PERFECT=True
```

becomes:

```python
True
```

Supported values:

```text
True
False
```

Invalid values raise:

```text
PERFECT must be True or False
```

---

# 🛡️ Semantic Validation

The parser also validates logical maze constraints.

---

## Positive Dimensions

Maze dimensions must be positive.

Invalid example:

```text
WIDTH=-5
```

---

## Entry and Exit Validation

The parser ensures:
- entry and exit are different
- coordinates are inside maze bounds

Invalid example:

```text
ENTRY=50,50
```

for a small maze raises an exception.

---

# 🚨 Error Handling

The parsing system uses a custom exception:

```python
class ConfigError(Exception):
```

This provides:
- cleaner error reporting
- easier debugging
- centralized configuration validation errors

---

# 🧩 Parsing Pipeline

```text
config.txt
     │
     ▼
parse_config()
     │
     ▼
Dictionary Creation
     │
     ▼
Mandatory Key Validation
     │
     ▼
convert_config()
     │
     ▼
Type Conversion
     │
     ▼
Semantic Validation
     │
     ▼
Validated Configuration
```

---

# 📦 Final Output Example

After parsing and conversion:

```python
{
    "WIDTH": 20,
    "HEIGHT": 20,
    "ENTRY": (0, 0),
    "EXIT": (19, 19),
    "OUTPUT_FILE": "output_maze.txt",
    "PERFECT": True,
    "SEED": 42
}
```

---

# 🎯 Goals of the Parsing System

The parsing system was designed to:
- simplify maze configuration
- provide strong validation
- prevent invalid maze generation
- improve debugging
- keep the project modular and maintainable

It acts as the first validation layer before any maze generation or rendering begins.