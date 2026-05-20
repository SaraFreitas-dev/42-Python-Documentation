# 🚨 Python Exceptions & Try/Except Guide

> Beginner-friendly documentation about Python exceptions, error handling, try/except blocks, and debugging practices.

This document explains:
- exceptions
- errors
- `try`
- `except`
- `finally`
- `else`
- custom exceptions
- common debugging patterns

---

# 📚 Table of Contents

- [📖 What are Exceptions?](#-what-are-exceptions)
- [⚠️ Common Python Errors](#️-common-python-errors)
- [🛠️ try / except](#️-try--except)
- [📦 Catching Specific Exceptions](#-catching-specific-exceptions)
- [🔄 else](#-else)
- [🧹 finally](#-finally)
- [🚨 Raising Exceptions](#-raising-exceptions)
- [🏗️ Custom Exceptions](#️-custom-exceptions)
- [🎮 Real 42 Examples](#-real-42-examples)
- [⚠️ Common Beginner Mistakes](#️-common-beginner-mistakes)
- [📚 Best Practices](#-best-practices)
- [📚 Final Notes](#-final-notes)

---

# 📖 What are Exceptions?

Exceptions are:
- runtime errors
- unexpected problems during execution

When an exception happens:
- Python stops normal execution

---

# Example

```python
print(10 / 0)
```

Output:

```text
ZeroDivisionError
```

---

# Why Exceptions Matter

Exceptions help:
- detect problems
- avoid crashes
- improve debugging
- handle invalid data safely

Very important in:
- parsers
- APIs
- MLX projects
- file systems
- user input validation

---

# ⚠️ Common Python Errors

| Exception | Meaning |
|---|---|
| ValueError | Invalid value |
| TypeError | Wrong type |
| ZeroDivisionError | Division by zero |
| FileNotFoundError | File missing |
| KeyError | Missing dictionary key |
| IndexError | Invalid list index |

---

# 🛠️ try / except

`try` allows Python to:
- attempt risky code

`except` handles:
- the error safely

---

# Example

```python
try:
    number = int(input("Number: "))
except ValueError:
    print("Invalid number")
```

---

# How It Works

Python:
1. runs the `try` block
2. checks for exceptions
3. jumps to `except` if an error happens

---

# 📦 Catching Specific Exceptions

Good practice:
- catch specific errors

---

# Example

```python
try:
    value = int("abc")

except ValueError:
    print("Conversion failed")
```

---

# Avoid Catching Everything

Bad:

```python
except:
    print("Error")
```

This hides:
- debugging information
- unexpected problems

---

# Better Version

```python
except ValueError:
```

---

# 🔄 else

`else` runs ONLY if:
- no exception happened

---

# Example

```python
try:
    value = int("42")

except ValueError:
    print("Invalid")

else:
    print("Success")
```

---

# 🧹 finally

`finally` ALWAYS runs.

Even if:
- exceptions happen

---

# Example

```python
try:
    file = open("test.txt")

except FileNotFoundError:
    print("Missing file")

finally:
    print("Finished")
```

---

# Why finally is Useful

Useful for:
- closing files
- cleanup
- freeing resources

---

# 🚨 Raising Exceptions

You can manually create exceptions using:

```python
raise
```

---

# Example

```python
age = -5

if age < 0:
    raise ValueError("Age cannot be negative")
```

---

# Why raise is Useful

Useful for:
- validation
- parsers
- enforcing rules
- debugging

---

# 🏗️ Custom Exceptions

You can create your own exception classes.

---

# Example

```python
class MapParserError(Exception):
    pass
```

---

# Raising the Custom Exception

```python
raise MapParserError("Invalid map format")
```

---

# Why Custom Exceptions Matter

They help:
- organize errors
- improve debugging
- create cleaner architecture

Very common in:
- parsers
- APIs
- game engines

---

# 🎮 Real 42 Examples

## Config Validation

```python
if width <= 0:
    raise ValueError("Width must be positive")
```

---

## Missing File

```python
except FileNotFoundError:
```

---

## Parser Error

```python
class ConfigError(Exception):
    pass
```

---

## Invalid Coordinates

```python
raise MapParserError(
    "Invalid coordinates"
)
```

---

# ⚠️ Common Beginner Mistakes

## ❌ Catching All Exceptions

Bad:

```python
except:
```

Too broad.

---

## ❌ Ignoring Exceptions

Bad:

```python
except ValueError:
    pass
```

Can hide bugs.

---

## ❌ Using Exceptions for Normal Logic

Exceptions should handle:
- unexpected problems

Not:
- normal flow control

---

## ❌ Forgetting finally Cleanup

Files/resources may remain open.

---

# 📚 Best Practices

- Catch specific exceptions
- Use meaningful error messages
- Create custom exceptions for large projects
- Avoid empty `except`
- Keep exception handling readable
- Use `finally` for cleanup

---

# 🧠 Why Exceptions Matter

Good exception handling helps:
- debugging
- reliability
- maintainability
- cleaner architecture

Exceptions become essential in:
- parsers
- file systems
- APIs
- MLX projects
- validation systems

---

# 📚 Final Notes

Understanding exceptions is one of the most important Python skills.

Good error handling creates:
- safer applications
- cleaner debugging
- better architecture
- more maintainable projects

Strong exception handling becomes increasingly important in larger projects.

---
