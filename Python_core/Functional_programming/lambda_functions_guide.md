# 🧩 Python Lambda Functions Guide

> Beginner-friendly documentation about Python lambda functions, anonymous functions, and common functional programming patterns.

This document explains:
- what lambda functions are
- lambda syntax
- common use cases
- `map()`
- `filter()`
- `sorted()`
- practical examples
- common mistakes

---

# 📚 Table of Contents

- [📖 What is a Lambda Function?](#-what-is-a-lambda-function)
- [🧱 Basic Syntax](#-basic-syntax)
- [⚙️ Lambda vs Normal Functions](#️-lambda-vs-normal-functions)
- [📦 Returning Values](#-returning-values)
- [🔗 Using Multiple Arguments](#-using-multiple-arguments)
- [🗂️ Lambda with sorted()](#️-lambda-with-sorted)
- [🧪 Lambda with map()](#-lambda-with-map)
- [🔍 Lambda with filter()](#-lambda-with-filter)
- [🎮 Real 42 Examples](#-real-42-examples)
- [⚠️ Common Beginner Mistakes](#️-common-beginner-mistakes)
- [📚 Best Practices](#-best-practices)
- [📚 Final Notes](#-final-notes)

---

# 📖 What is a Lambda Function?

A lambda function is:
- a small anonymous function
- created in a single line
- often used for quick operations

Unlike normal functions:
- lambda functions usually do not have names

---

# 🧱 Basic Syntax

```python
lambda arguments: expression
```

---

# Simple Example

```python
square = lambda x: x * x

print(square(5))
```

Output:

```text
25
```

---

# Equivalent Normal Function

```python
def square(x):
    return x * x
```

Both functions behave the same way.

---

# ⚙️ Lambda vs Normal Functions

## Lambda Function

```python
lambda x: x + 1
```

---

## Normal Function

```python
def add_one(x):
    return x + 1
```

---

# 📦 Returning Values

Lambda functions automatically return the expression result.

---

# 🔗 Using Multiple Arguments

```python
add = lambda a, b, c: a + b + c
```

---

# 🗂️ Lambda with sorted()

```python
players.sort(key=lambda player: player[1])
```

---

# 🧪 Lambda with map()

```python
numbers = [1, 2, 3]

result = list(map(lambda x: x * 2, numbers))
```

---

# 🔍 Lambda with filter()

```python
numbers = [1, 2, 3, 4]

evens = list(filter(lambda x: x % 2 == 0, numbers))
```

---

# 🎮 Real 42 Examples

## Sorting Coordinates

```python
positions.sort(key=lambda pos: pos[0])
```

---

## Filtering Valid Zones

```python
valid_zones = list(filter(lambda zone: zone.active, zones))
```

---

# ⚠️ Common Beginner Mistakes

## ❌ Trying to Use Multiple Lines

Wrong:

```python
lambda x:
    x + 1
```

Lambda functions must stay on ONE line.

---

## ❌ Forgetting list()

`map()` and `filter()` return iterators.

Usually convert them:

```python
list(map(...))
```

---

# 📚 Best Practices

- Use lambdas for short operations
- Prefer normal functions for complex logic
- Keep lambdas readable
- Avoid nested lambdas

---

# 📚 Final Notes

Lambda functions are extremely common in modern Python.

They are especially useful for:
- quick transformations
- sorting
- filtering
- concise operations

---
