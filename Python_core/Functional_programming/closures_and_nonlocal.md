# Closures and `nonlocal` in Python

## What is a Closure?

A closure is a function that **remembers variables** from the scope where it was created, even after that outer function has finished running.

Closures allow functions to keep a private internal state without using classes or global variables.

---

# Basic Closure Example

```python
def outer():
    message = "Hello"

    def inner():
        return message

    return inner


func = outer()

print(func())
```

## Output

```text
Hello
```

---

# Why Does This Work?

Normally, local variables disappear after a function ends.

But here:

```python
message = "Hello"
```

is remembered by:

```python
inner()
```

Even after `outer()` has finished.

That remembered environment is called a:

# → Closure

---

# Lexical Scoping

Python uses **lexical scoping**.

This means:
- inner functions can access variables from outer scopes
- variable lookup depends on where the function was defined

---

# Scope Order (LEGB Rule)

Python searches variables in this order:

| Scope | Meaning |
|---|---|
| Local | Current function |
| Enclosing | Outer function |
| Global | File-level variables |
| Built-in | Python built-ins |

---

# Closure Memory Example

```python
def counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment


c = counter()

print(c())
print(c())
print(c())
```

## Output

```text
1
2
3
```

---

# Visual Representation

```text
counter()
│
├── count = 0
│
└── returns increment()
        │
        └── remembers count variable
```

The function keeps access to `count` even after `counter()` has ended.

---

# What Does `nonlocal` Do?

`nonlocal` allows modifying variables from the enclosing function scope.

Without `nonlocal`, Python creates a new local variable instead.

---

# Without `nonlocal`

```python
def counter():
    count = 0

    def increment():
        count += 1
        return count

    return increment
```

## Error

```text
UnboundLocalError:
local variable 'count' referenced before assignment
```

Python thinks `count` is local to `increment()`.

---

# Correct Version With `nonlocal`

```python
def counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

---

# `global` vs `nonlocal`

| global | nonlocal |
|---|---|
| Modifies global variables | Modifies enclosing scope variables |
| Works at file/module level | Works inside nested functions |
| Less safe | Safer for closures |
| Shared everywhere | Private to closure |

---

# Example Using `global`

```python
count = 0

def increment():
    global count
    count += 1
```

This changes a variable shared by the whole program.

---

# Example Using `nonlocal`

```python
def counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

This creates private persistent state.

---

# Persistent State Without Classes

Closures are often used instead of classes for small stateful systems.

---

# Closure Version

```python
def accumulator(start):
    total = start

    def add(value):
        nonlocal total
        total += value
        return total

    return add


acc = accumulator(100)

print(acc(20))
print(acc(30))
```

## Output

```text
120
150
```

---

# Equivalent Class Version

```python
class Accumulator:
    def __init__(self, start):
        self.total = start

    def add(self, value):
        self.total += value
        return self.total
```

Closures provide a lighter functional alternative.

---

# Real Use Cases for Closures

Closures are commonly used for:
- decorators
- counters
- caching systems
- state tracking
- callback functions
- factories
- retry systems

---

# Common Mistakes

## Forgetting `nonlocal`

```python
count += 1
```

without:

```python
nonlocal count
```

---

## Returning the Function Call Instead of the Function

Wrong:

```python
return increment()
```

Correct:

```python
return increment
```

---

# Key Takeaways

- Closures remember variables from outer scopes
- Python uses lexical scoping
- `nonlocal` modifies enclosing scope variables
- Closures can maintain persistent state
- Closures are powerful alternatives to small classes
- Closures are heavily used in decorators and functional programming
