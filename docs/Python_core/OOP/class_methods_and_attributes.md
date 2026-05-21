# 🧠 Python Class Methods & Attributes Guide

> Beginner-friendly documentation about instance methods, class methods, static methods, attributes, and common object-oriented programming patterns in Python.

This document explains:
- instance methods
- class methods
- static methods
- instance attributes
- class attributes
- `self`
- `cls`
- common use cases
- practical examples

---

# 📚 Table of Contents

- [📖 What are Methods?](#-what-are-methods)
- [📌 self](#-self)
- [📦 Instance Attributes](#-instance-attributes)
- [🏛️ Class Attributes](#️-class-attributes)
- [⚙️ Instance Methods](#️-instance-methods)
- [🏷️ Class Methods](#️-class-methods)
- [🔗 cls](#-cls)
- [🧩 Static Methods](#-static-methods)
- [📊 Method Comparison Table](#-method-comparison-table)
- [🏗️ Factory Methods](#️-factory-methods)
- [🎮 Real 42 Examples](#-real-42-examples)
- [⚠️ Common Beginner Mistakes](#️-common-beginner-mistakes)
- [📚 Best Practices](#-best-practices)
- [📚 Final Notes](#-final-notes)

---

# 📖 What are Methods?

Methods are:
- functions inside classes

Methods help objects:
- perform actions
- manipulate data
- organize behavior

---

# 📌 self

`self` refers to:
- the current object instance

Used to access:
- object attributes
- object methods

---

# Example

```python
class Player:

    def __init__(self, name):
        self.name = name
```

---

# 📦 Instance Attributes

Instance attributes belong to:
- individual objects

Each object stores its own values.

---

# Example

```python
class Drone:

    def __init__(self, battery):
        self.battery = battery
```

---

# Creating Objects

```python
drone1 = Drone(100)
drone2 = Drone(50)
```

Each object has:
- different battery values

---

# 🏛️ Class Attributes

Class attributes belong to:
- the class itself

Shared by ALL objects.

---

# Example

```python
class Drone:

    max_speed = 10
```

---

# Accessing Class Attributes

```python
print(Drone.max_speed)
```

or:

```python
drone = Drone()

print(drone.max_speed)
```

---

# Why Class Attributes Are Useful

Useful for:
- constants
- shared settings
- counters
- global configuration

---

# ⚙️ Instance Methods

Instance methods work with:
- object data

They receive:
- `self`

---

# Example

```python
class Player:

    def jump(self):
        print("Jump!")
```

---

# Using the Method

```python
player = Player()

player.jump()
```

---

# When to Use Instance Methods

Use instance methods when:
- behavior depends on object data

---

# 🏷️ Class Methods

Class methods work with:
- the class itself

They receive:
- `cls`

instead of:
- `self`

---

# Example

```python
class Player:

    total_players = 0

    @classmethod
    def show_total(cls):
        print(cls.total_players)
```

---

# 🔗 cls

`cls` refers to:
- the class itself

Similar to how:
- `self` refers to the object

---

# Using the Class Method

```python
Player.show_total()
```

---

# Why Class Methods Are Useful

Useful for:
- shared class logic
- object counters
- factory methods
- alternative constructors

---

# 🧩 Static Methods

Static methods belong to the class but do NOT use:
- `self`
- `cls`

They behave like utility/helper functions.

---

# Example

```python
class MathUtils:

    @staticmethod
    def add(a, b):
        return a + b
```

---

# Using the Static Method

```python
print(MathUtils.add(2, 3))
```

Output:

```text
5
```

---

# Why Static Methods Are Useful

Useful for:
- utility functions
- calculations
- helper logic
- formatting functions

---

# 📊 Method Comparison Table

| Method Type | Uses self | Uses cls | Common Usage |
|---|---|---|---|
| Instance Method | ✅ | ❌ | Object behavior |
| Class Method | ❌ | ✅ | Shared class logic |
| Static Method | ❌ | ❌ | Utility/helper functions |

---

# 🏗️ Factory Methods

Factory methods are one of the most common uses of:
- `@classmethod`

---

# Example

```python
class Drone:

    def __init__(self, battery):
        self.battery = battery

    @classmethod
    def default_drone(cls):
        return cls(100)
```

---

# Using the Factory Method

```python
drone = Drone.default_drone()
```

This creates:
- a default object configuration

---

# 🎮 Real 42 Examples

## Shared Configuration

```python
class GameConfig:

    TILE_SIZE = 64
```

---

## Parser Helpers

```python
class ConfigParser:

    @staticmethod
    def clean_line(line):
        ...
```

---

## Object Counter

```python
class Drone:

    total_drones = 0
```

---

## Factory Method

```python
class Maze:

    @classmethod
    def default_maze(cls):
        ...
```

---

# ⚠️ Common Beginner Mistakes

## ❌ Forgetting self

Wrong:

```python
def jump():
```

Correct:

```python
def jump(self):
```

---

## ❌ Using self in Static Methods

Wrong:

```python
@staticmethod
def test():
    print(self.name)
```

Static methods do not have:
- `self`

---

## ❌ Confusing Class and Instance Attributes

Wrong:

```python
class Enemy:
    hp = []
```

Mutable shared attributes may create bugs.

---

## ❌ Using Class Methods Like Instance Methods

Class methods work with:
- class-level data

Not:
- object-specific state

---

# 📚 Best Practices

- Use instance methods for object behavior
- Use static methods for utilities
- Use class methods for shared class logic
- Keep classes focused
- Avoid giant classes
- Use meaningful method names

---

# 🧠 When to Use Each Method

## Instance Method

Use when:
- logic depends on object data

---

## Class Method

Use when:
- logic depends on class-wide data

---

## Static Method

Use when:
- no object/class state is required

---

# 📚 Final Notes

Understanding methods and attributes is one of the most important parts of Python OOP.

These concepts help:
- organize projects
- improve readability
- structure large applications
- reduce duplicated logic

They become especially useful in:
- MLX projects
- parsers
- APIs
- games
- collaborative repositories

---
