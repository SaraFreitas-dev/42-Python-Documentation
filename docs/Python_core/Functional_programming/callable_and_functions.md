# Callable and Functions in Python

# Functions Are First-Class Objects

In Python, functions are treated like any other object.

This means functions can:
- be stored in variables
- be passed as arguments
- be returned from other functions
- be stored inside lists/dictionaries

---

# Function Object vs Function Call

This is one of the most important concepts in functional programming.

---

# Function Object

```python
def greet():
    return "Hello"


print(greet)
```

## Output

```text
<function greet at 0x...>
```

Here we are referencing the function itself.

No execution happens.

---

# Function Call

```python
def greet():
    return "Hello"


print(greet())
```

## Output

```text
Hello
```

The parentheses `()` execute the function.

---

# Visual Difference

| Code | Meaning |
|---|---|
| greet | Function object |
| greet() | Function call / execution |

---

# Storing Functions in Variables

```python
def fireball():
    return "Fireball cast!"


spell = fireball

print(spell())
```

## Output

```text
Fireball cast!
```

The variable stores the function itself.

---

# Passing Functions as Arguments

Functions can be passed into other functions.

---

# Example

```python
def greet(name):
    return f"Hello {name}"


def execute(func, value):
    return func(value)


print(execute(greet, "Sara"))
```

## Output

```text
Hello Sara
```

---

# Why Is This Powerful?

This allows:
- reusable logic
- decorators
- callbacks
- flexible systems
- functional programming patterns

---

# Returning Functions

Functions can also return other functions.

---

# Example

```python
def multiplier(x):

    def inner(y):
        return x * y

    return inner


double = multiplier(2)

print(double(5))
```

## Output

```text
10
```

---

# Visual Representation

```text
multiplier(2)
│
└── returns inner()
        │
        └── remembers x = 2
```

This is the basis of closures.

---

# What Is `Callable`?

`Callable` is a type hint used for functions.

It comes from:

```python
from collections.abc import Callable
```

---

# Basic Callable Example

```python
from collections.abc import Callable


def execute(func: Callable, value: int) -> int:
    return func(value)
```

---

# Callable With Parameters and Return Types

You can specify:
- argument types
- return type

---

# Syntax

```python
Callable[[arg1_type, arg2_type], return_type]
```

---

# Example

```python
from collections.abc import Callable


def operation(func: Callable[[int, int], int]) -> int:
    return func(2, 3)
```

---

# What Does `callable()` Do?

`callable()` checks if an object can be called like a function.

---

# Example

```python
def greet():
    pass


print(callable(greet))
print(callable(42))
```

## Output

```text
True
False
```

---

# Objects Can Also Be Callable

Classes using `__call__()` become callable.

---

# Example

```python
class Spell:

    def __call__(self):
        return "Magic!"


spell = Spell()

print(callable(spell))
print(spell())
```

## Output

```text
True
Magic!
```

---

# Functions Inside Data Structures

Functions can be stored inside:
- lists
- tuples
- dictionaries

---

# Example

```python
def fire():
    return "Fire"


def ice():
    return "Ice"


spells = {
    "fire": fire,
    "ice": ice
}

print(spells["fire"]())
```

## Output

```text
Fire
```

---

# Common Mistakes

## Returning a Function Call Instead of the Function

Wrong:

```python
return greet()
```

Correct:

```python
return greet
```

---

# Forgetting Parentheses

Wrong:

```python
print(greet)
```

Correct:

```python
print(greet())
```

---

# Real Use Cases

These concepts are heavily used in:
- decorators
- callbacks
- event systems
- retry systems
- Flask/FastAPI routes
- functional programming
- ML libraries

---

# Key Takeaways

- Functions are first-class objects in Python
- Functions can be passed, stored, and returned
- `Callable` is used for function type hints
- `callable()` checks if an object can be executed
- `function` and `function()` are very different
- Closures and decorators rely heavily on these concepts
