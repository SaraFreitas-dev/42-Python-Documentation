# 🔍 Python Traceback & Debugging Guide

> Beginner-friendly documentation about Python tracebacks, exception debugging, and useful debugging patterns.

This document explains:
- what a traceback is
- how exceptions work
- `traceback` module
- extracting function names
- debugging helpers
- useful debugging patterns

---

# 📚 Table of Contents

- [📖 What is a Traceback?](#-what-is-a-traceback)
- [⚠️ Basic Exception Output](#️-basic-exception-output)
- [🧠 Using traceback](#-using-traceback)
- [📦 extract_tb()](#-extract_tb)
- [🎯 Getting the Failed Function Name](#-getting-the-failed-function-name)
- [🛠️ Useful Debugging Patterns](#️-useful-debugging-patterns)
- [🎮 Real 42 Examples](#-real-42-examples)
- [⚠️ Common Beginner Mistakes](#️-common-beginner-mistakes)
- [📚 Best Practices](#-best-practices)
- [📚 Final Notes](#-final-notes)

---

# 📖 What is a Traceback?

A traceback is:
- Python's error report

It shows:
- where the error happened
- which functions were called
- the exception type
- the error message

---

# Example Traceback

```text
Traceback (most recent call last):
  File "main.py", line 10, in <module>
    divide(10, 0)
  File "main.py", line 5, in divide
    return a / b

ZeroDivisionError: division by zero
```

---

# ⚠️ Basic Exception Output

```python
try:
    value = 10 / 0

except Exception as e:
    print(type(e).__name__)
    print(e)
```

---

# Output

```text
ZeroDivisionError
division by zero
```

---

# 🧠 Using traceback

```python
import traceback
```

Useful for:
- advanced debugging
- custom error displays
- logging

---

# 📦 extract_tb()

`traceback.extract_tb()` extracts traceback information.

---

# Example

```python
import traceback

try:
    value = 10 / 0

except Exception as e:

    tb = traceback.extract_tb(e.__traceback__)

    print(tb)
```

---

# 🎯 Getting the Failed Function Name

```python
import traceback

try:
    value = 10 / 0

except Exception as e:

    tb = traceback.extract_tb(e.__traceback__)

    print(f"Failed on the function: {tb[-1].name}")
    print(f"{type(e).__name__}: {e}")
```

---

# Example Output

```text
Failed on the function: divide
ZeroDivisionError: division by zero
```

---

# 🛠️ Useful Debugging Patterns

## Showing the Exception Type

```python
print(type(e).__name__)
```

---

## Showing the Exception Message

```python
print(e)
```

---

## Combining Both

```python
print(f"{type(e).__name__}: {e}")
```

---

# 🎮 Real 42 Examples

## Parser Errors

```python
except Exception as e:

    tb = traceback.extract_tb(e.__traceback__)

    print(f"Failed on function: {tb[-1].name}")
```

---

## Invalid Config

```text
ValueError: Invalid WIDTH
```

---

## Invalid Coordinates

```text
MapParserError: Invalid coordinates
```

---

# ⚠️ Common Beginner Mistakes

## ❌ Using Bare except

Bad:

```python
except:
```

---

## ❌ Ignoring Tracebacks

The traceback usually shows:
- the REAL problem location

---

## ❌ Printing Huge Tracebacks Everywhere

Too much debugging output can:
- clutter logs
- make debugging harder

---

# 📚 Best Practices

- Read traceback lines carefully
- Look at the LAST function call first
- Print exception types clearly
- Use meaningful error messages
- Add debugging progressively

---

# 📚 Final Notes

Understanding tracebacks is essential for Python development.

Good traceback handling helps:
- locate bugs quickly
- improve error reporting
- simplify debugging

---
