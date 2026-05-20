# ✨ Python Dunder Methods Guide

> Beginner-friendly documentation about Python dunder methods (magic methods), special object behavior, and common object-oriented patterns.


---

# 📚 Table of Contents

- [📖 What are Dunder Methods?](#-what-are-dunder-methods)
- [🚀 __init__](#-__init__)
- [🖨️ __str__](#️-__str__)
- [🔍 __repr__](#-__repr__)
- [📏 __len__](#-__len__)
- [⚖️ __eq__](#️-__eq__)
- [➕ __add__](#-__add__)
- [📦 __getitem__](#-__getitem__)
- [🔄 __iter__](#-__iter__)
- [🎮 Real 42 Examples](#-real-42-examples)
- [⚠️ Common Beginner Mistakes](#️-common-beginner-mistakes)
- [📚 Best Practices](#-best-practices)
- [📚 Final Notes](#-final-notes)

---

# 📖 What are Dunder Methods?

Dunder methods are:
- special Python methods
- surrounded by double underscores

Example:

```python
__init__
```

The word:
```text
dunder
```

means:
```text
double underscore
```

---

# Why Dunder Methods Exist

They allow Python objects to:
- behave like built-in types
- customize operators
- customize printing
- customize comparisons
- support iteration

---

# 🚀 __init__

`__init__` runs automatically when an object is created.

Used for:
- initializing attributes
- preparing object state

---

# Example

```python
class Player:

    def __init__(self, name):
        self.name = name
```

---

# Creating the Object

```python
player = Player("Sara")
```

---

# 🖨️ __str__

`__str__` controls:
- readable object printing

---

# Example

```python
class Player:

    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f"Player({self.name})"
```

---

# Using __str__

```python
player = Player("Sara")

print(player)
```

Output:

```text
Player(Sara)
```

---

# Why __str__ is Useful

Without `__str__`, Python prints:

```text
<object at 0x123456>
```

which is not very readable.

---

# 🔍 __repr__

`__repr__` provides:
- developer/debug representation

---

# Example

```python
class Enemy:

    def __repr__(self):
        return "Enemy()"
```

---

# Difference Between __str__ and __repr__

| Method | Purpose |
|---|---|
| `__str__` | User-friendly output |
| `__repr__` | Developer/debug output |

---

# 📏 __len__

`__len__` controls:
- object length behavior

---

# Example

```python
class Inventory:

    def __init__(self):
        self.items = ["Sword", "Potion"]

    def __len__(self):
        return len(self.items)
```

---

# Using len()

```python
inventory = Inventory()

print(len(inventory))
```

Output:

```text
2
```

---

# ⚖️ __eq__

`__eq__` controls:
- equality comparison

---

# Example

```python
class Player:

    def __init__(self, score):
        self.score = score

    def __eq__(self, other):
        return self.score == other.score
```

---

# Using Equality

```python
player1 = Player(10)
player2 = Player(10)

print(player1 == player2)
```

Output:

```text
True
```

---

# ➕ __add__

`__add__` customizes:
- the `+` operator

---

# Example

```python
class Coins:

    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        return Coins(self.amount + other.amount)
```

---

# Using +

```python
c1 = Coins(5)
c2 = Coins(10)

result = c1 + c2
```

---

# 📦 __getitem__

`__getitem__` allows:
- indexing with `[]`

---

# Example

```python
class Inventory:

    def __init__(self):
        self.items = ["Sword", "Potion"]

    def __getitem__(self, index):
        return self.items[index]
```

---

# Using Indexing

```python
inventory = Inventory()

print(inventory[0])
```

Output:

```text
Sword
```

---

# 🔄 __iter__

`__iter__` allows:
- iteration with loops

---

# Example

```python
class Inventory:

    def __init__(self):
        self.items = ["Sword", "Potion"]

    def __iter__(self):
        return iter(self.items)
```

---

# Using Iteration

```python
for item in inventory:
    print(item)
```

---

# 🎮 Real 42 Examples

## Maze String Representation

```python
def __str__(self):
```

Useful for:
- ASCII maze printing

---

## Position Comparison

```python
def __eq__(self, other):
```

Useful for:
- coordinate comparison
- pathfinding

---

## Inventory Length

```python
def __len__(self):
```

Useful for:
- item systems

---

## Custom Containers

```python
def __getitem__(self, index):
```

Useful for:
- grid systems
- tile access

---

# ⚠️ Common Beginner Mistakes

## ❌ Returning Wrong Types

Bad:

```python
def __len__(self):
    return "five"
```

`__len__` must return:
- integer

---

## ❌ Forgetting other in __eq__

Wrong:

```python
def __eq__(self):
```

Correct:

```python
def __eq__(self, other):
```

---

## ❌ Overcomplicating Dunder Methods

Keep behavior:
- simple
- predictable
- readable

---

## ❌ Forgetting Return Values

Example:

```python
def __str__(self):
    print("Player")
```

Wrong because:
- `__str__` must RETURN a string

---

# 📚 Best Practices

- Keep dunder methods simple
- Return correct types
- Use dunder methods only when useful
- Avoid unnecessary operator overloads
- Focus on readability

---

# 🧠 Why Dunder Methods Matter

Dunder methods help objects:
- behave naturally
- integrate with Python syntax
- support built-in operations
- feel more flexible

They are heavily used in:
- frameworks
- libraries
- games
- APIs
- custom containers

---

# 📚 Final Notes

Dunder methods are one of the most powerful features of Python OOP.

Understanding them helps create:
- cleaner APIs
- more intuitive objects
- reusable systems
- better project architecture

They become increasingly useful in larger and more advanced Python projects.

---
