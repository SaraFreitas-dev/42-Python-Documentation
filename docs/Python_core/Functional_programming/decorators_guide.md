# Decorators Guide in Python

# What Is a Decorator?

A decorator is a function that modifies another function.

Decorators allow adding extra behavior:
- logging
- timing
- validation
- retry systems
- permissions
- caching

Without changing the original function.

---

# Basic Decorator Structure

```python
def decorator(func):

    def wrapper():
        print("Before function")

        result = func()

        print("After function")

        return result

    return wrapper
```

---

# Applying the Decorator

```python
@decorator
def greet():
    print("Hello")
```

Equivalent to:

```python
greet = decorator(greet)
```

---

# Output

```text
Before function
Hello
After function
```

---

# Why Are Decorators Useful?

They separate concerns:
- business logic stays clean
- reusable behavior stays outside functions

---

# Decorator With Arguments

Most decorators need `*args` and `**kwargs`.

---

# Example

```python
def logger(func):

    def wrapper(*args, **kwargs):
        print("Function called")

        return func(*args, **kwargs)

    return wrapper
```

---

# Timing Decorator

```python
import time


def timer(func):

    def wrapper(*args, **kwargs):
        start = time.time()

        result = func(*args, **kwargs)

        end = time.time()

        print(f"Executed in {end - start:.3f}s")

        return result

    return wrapper
```

---

# Example Usage

```python
@timer
def slow_spell():
    time.sleep(1)
```

---

# Validation Decorator

```python
def validate_power(min_power):

    def decorator(func):

        def wrapper(power):
            if power < min_power:
                return "Insufficient power"

            return func(power)

        return wrapper

    return decorator
```

---

# Example Usage

```python
@validate_power(10)
def cast_spell(power):
    return f"Spell cast with {power}"
```

---

# Decorator Factories

Decorators with parameters are called:
# → Decorator Factories

Because they create decorators dynamically.

---

# Retry Decorator

```python
def retry(max_attempts):

    def decorator(func):

        def wrapper(*args, **kwargs):

            for attempt in range(max_attempts):

                try:
                    return func(*args, **kwargs)

                except Exception:
                    print("Retrying...")

            return "Failed"

        return wrapper

    return decorator
```

---

# Example Usage

```python
@retry(3)
def unstable_spell():
    raise ValueError("Boom")
```

---

# Stacking Decorators

Multiple decorators can be combined.

---

# Example

```python
@timer
@logger
def fireball():
    print("Fireball!")
```

Equivalent to:

```python
fireball = timer(logger(fireball))
```

---

# Execution Order

Decorators execute from bottom to top.

---

# Why Use `functools.wraps`?

Without `wraps`, decorators replace:
- function name
- docstring
- metadata

---

# Problem Example

```python
def decorator(func):

    def wrapper():
        return func()

    return wrapper
```

```python
print(greet.__name__)
```

## Output

```text
wrapper
```

The original name is lost.

---

# Correct Version With `wraps`

```python
from functools import wraps


def decorator(func):

    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Now metadata is preserved.

---

# Decorators on Methods

Decorators also work on class methods.

---

# Example

```python
class Mage:

    @staticmethod
    def validate(name):
        return len(name) > 2
```

---

# Common Mistakes

## Forgetting to Return the Wrapper

Wrong:

```python
return func
```

Correct:

```python
return wrapper
```

---

# Forgetting `*args` and `**kwargs`

Wrong:

```python
def wrapper():
```

Correct:

```python
def wrapper(*args, **kwargs):
```

---

# Real Use Cases

Decorators are heavily used in:
- Flask
- FastAPI
- Django
- testing frameworks
- logging systems
- caching systems
- authentication
- retry systems

---

# Key Takeaways

- Decorators modify function behavior
- They help separate concerns
- `@decorator` is syntactic sugar
- `wraps` preserves metadata
- Decorators can accept parameters
- Multiple decorators can be stacked
- Decorators are extremely common in modern Python
