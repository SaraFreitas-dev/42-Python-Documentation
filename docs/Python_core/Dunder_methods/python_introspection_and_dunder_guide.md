# 🧠 Python Introspection & Useful Dunder Methods Guide

> Beginner-friendly documentation about useful Python dunder attributes and introspection tools commonly used for debugging, classes, logging, and development workflows.

This document explains:
- `__name__`
- `__class__`
- `__dict__`
- `__module__`
- `type()`
- getting class names
- getting function names
- debugging patterns
- introspection basics

---

# 📚 Table of Contents

- [📖 What is Introspection?](#-what-is-introspection)
- [🏷️ __name__](#️-__name__)
- [🏛️ __class__](#️-__class__)
- [📦 __dict__](#-__dict__)
- [📁 __module__](#-__module__)
- [🔍 type()](#-type)
- [🎯 Getting Class Names](#-getting-class-names)
- [⚙️ Getting Function Names](#️-getting-function-names)
- [🛠️ Useful Debugging Patterns](#️-useful-debugging-patterns)
- [🎮 Real 42 Examples](#-real-42-examples)
- [⚠️ Common Beginner Mistakes](#️-common-beginner-mistakes)
- [📚 Best Practices](#-best-practices)
- [📚 Final Notes](#-final-notes)

---

# 📖 What is Introspection?

Introspection means:
- inspecting Python objects dynamically

Python allows you to inspect:
- classes
- objects
- functions
- modules
- attributes

Very useful for:
- debugging
- logging
- parsers
- large projects

---

# 🏷️ __name__

`__name__` stores:
- the name of a function
- the name of a module

---

# Function Example

```python
def move():
    pass

print(move.__name__)
```

Output:

```text
move
```

---

# Module Example

```python
print(__name__)
```

Usually outputs:

```text
__main__
```

when running the file directly.

---

# 🏛️ __class__

`__class__` gives:
- the object's class

---

# Example

```python
class Drone:
    pass

drone = Drone()

print(drone.__class__)
```

Output:

```text
<class '__main__.Drone'>
```

---

# 📦 __dict__

`__dict__` stores:
- object attributes

---

# Example

```python
class Player:

    def __init__(self):
        self.hp = 100
        self.name = "Sara"

player = Player()

print(player.__dict__)
```

---

# Output

```text
{'hp': 100, 'name': 'Sara'}
```

---

# Why __dict__ is Useful

Useful for:
- debugging
- serialization
- quick inspection

---

# 📁 __module__

`__module__` shows:
- where the class/function was defined

---

# Example

```python
print(Player.__module__)
```

---

# 🔍 type()

`type()` returns:
- the object's type

---

# Example

```python
print(type(player))
```

Output:

```text
<class '__main__.Player'>
```

---

# 🎯 Getting Class Names

Very common debugging pattern.

---

# Example

```python
print(player.__class__.__name__)
```

Output:

```text
Player
```

---

# Why This is Useful

Useful for:
- debugging
- logging
- error reporting
- game systems

---

# ⚙️ Getting Function Names

Functions also store their names.

---

# Example

```python
def validate():
    pass

print(validate.__name__)
```

Output:

```text
validate
```

---

# Using Inside Debugging

```python
print(f"Failed inside function: {validate.__name__}")
```

---

# 🛠️ Useful Debugging Patterns

## Printing Class Names

```python
print(type(obj).__name__)
```

---

# Example Output

```text
Maze
```

---

## Printing Function Names

```python
print(function.__name__)
```

---

## Printing Object Attributes

```python
print(obj.__dict__)
```

---

## Printing Exception Types

```python
print(type(e).__name__)
```

---

# 🎮 Real 42 Examples

## Parser Debugging

```python
print(type(zone).__name__)
```

---

## Maze Debugging

```python
print(maze.__dict__)
```

---

## Function Tracking

```python
print(generate_maze.__name__)
```

---

## Error Reporting

```python
print(f"{type(e).__name__}: {e}")
```

---

# ⚠️ Common Beginner Mistakes

## ❌ Confusing __name__ Types

Functions:
- have `__name__`

Objects:
- usually do not

---

## ❌ Printing Entire Objects Accidentally

```python
print(obj)
```

may not be useful without:
- `__str__`

---

## ❌ Modifying __dict__ Incorrectly

Changing `__dict__` directly may:
- create bugs
- break objects

---

# 📚 Best Practices

- Use introspection mainly for debugging
- Keep debug prints readable
- Use meaningful class names
- Avoid excessive debug output
- Prefer structured logging in large projects

---

# 🧠 Why Introspection Matters

Introspection helps:
- understand objects dynamically
- debug faster
- inspect large systems
- improve logging

Very useful in:
- parsers
- frameworks
- MLX projects
- game systems
- debugging tools

---

# 📚 Final Notes

Python introspection is one of the language's most powerful features.

Understanding these tools helps:
- debug efficiently
- inspect objects dynamically
- improve architecture
- build better development workflows

These patterns become increasingly useful in larger Python projects.

---
