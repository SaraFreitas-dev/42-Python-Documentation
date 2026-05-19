# ⚙️ Python Makefile Guide

> Beginner-friendly documentation about Makefiles for Python projects, automation workflows, linting, virtual environments, and common development commands.

📎 A complete example Makefile is provided alongside this guide for reference and practical implementation examples.

---

# 📚 Table of Contents

- [📖 What is a Makefile?](#-what-is-a-makefile)
- [🐍 Why Use Makefiles in Python?](#-why-use-makefiles-in-python)
- [🛠️ Common Python Makefile Commands](#️-common-python-makefile-commands)
- [🧱 Basic Makefile Structure](#-basic-makefile-structure)
- [🔀 Conditional Statements in Makefiles](#-conditional-statements-in-makefiles)
- [🎯 Targets](#-targets)
- [📦 Variables](#-variables)
- [🚀 Default Targets](#-default-targets)
- [🔗 Dependencies](#-dependencies)
- [🧪 Virtual Environment Example](#-virtual-environment-example)
- [📥 Installing Requirements](#-installing-requirements)
- [🔍 Linting Example](#-linting-example)
- [🧹 Cleaning Cache Files](#-cleaning-cache-files)
- [🧨 Full Clean Example](#-full-clean-example)
- [♻️ Rebuild Example](#️-rebuild-example)
- [📌 .PHONY](#-phony)
- [⚠️ Common Beginner Mistakes](#️-common-beginner-mistakes)
- [🧠 Recommended Workflow](#-recommended-workflow)
- [📚 Additional Notes](#-additional-notes)
- [📚 Final Notes](#-final-notes)

---


# 📖 What is a Makefile?

A Makefile is a file used to automate terminal commands.

Instead of typing long commands repeatedly, you can run:

```bash
make
```

or:

```bash
make run
```

and execute predefined workflows automatically.

---


# 🐍 Why Use Makefiles in Python?

Makefiles help:
- automate repetitive tasks
- standardize project workflows
- simplify setup
- improve teamwork
- reduce mistakes

Very useful for:
- 42 projects
- game projects
- parsers
- APIs
- collaborative repositories

---

# 🛠️ Common Python Makefile Commands

| ⚙️ Target | 📖 Purpose |
|---|---|
| `make` | 🚀 Default workflow |
| `make run` | ▶️ Run project |
| `make install` | 📦 Install dependencies |
| `make lint` | 🔍 Run flake8/mypy |
| `make clean` | 🧹 Remove cache |
| `make re` | ♻️ Rebuild project |

---

# 🧱 Basic Makefile Structure

## Simple Example

```make
run:
	python3 main.py
```

Execute with the command:

```bash
make run
```

# 🔀 Conditional Statements in Makefiles

Makefiles can also use shell conditionals such as:

```make
@if [ ! -d "$(VENV)" ]; then \
	python3 -m venv $(VENV); \
fi
```

---

# What This Does

This checks if the virtual environment folder already exists.

## Explanation

| Part | Meaning |
|---|---|
| `if` | Starts the condition |
| `[ ! -d "$(VENV)" ]` | Checks if the directory does NOT exist |
| `then` | Executes the commands below |
| `fi` | Ends the condition |

---

# Why This is Useful

This prevents:
- recreating the virtual environment every time
- reinstalling dependencies unnecessarily
- accidentally overwriting environments

Very common in Python project setup workflows.

---

# Common Example

```make
venv:
	@if [ ! -d "$(VENV)" ]; then \
		python3 -m venv $(VENV); \
	fi
```

This means:
- if `venv/` does not exist → create it
- otherwise → do nothing


---

# 🎯 Targets

A target is a command section inside the Makefile.

## Example

```make
install:
	pip install -r requirements.txt
```

---

# 📦 Variables

To Makefiles cleaner and reusable.

Variables are useful to:
- reduce repetition
- simplify updates
- improve readability


## Example

```make
PYTHON = python3
```

Used like:

```make
$(PYTHON) main.py
```

---

# 🚀 Default Targets

The first target is usually the default target.

## Example

```make
all: run
```

Running:

```bash
make
```

automatically runs:

```bash
make run
```

---

# 🔗 Dependencies

Targets can depend on other targets.

## Example

```make
run: install
	python3 main.py
```

Meaning:
- `install` runs first
- then `run`

---

# 🧪 Virtual Environment Example

```make
venv:
	python3 -m venv venv
```

---

## Using the venv Python

```make
run:
	venv/bin/python3 main.py
```

This guarantees:
- correct packages
- correct environment

---

# 📥 Installing Requirements

```make
install:
	venv/bin/pip install -r requirements.txt
```

---

# 🔍 Linting Example

```make
lint:
	flake8 .
	mypy .
```

Useful for:
- formatting checks
- typing checks

---

# 🧹 Cleaning Cache Files

```make
clean:
	find . -name "__pycache__" -exec rm -rf {} +
```

---

# 🧨 Full Clean Example

```make
fclean: clean
	rm -rf venv
```

Removes:
- cache files
- virtual environment

---

# ♻️ Rebuild Example

```make
re: fclean install
```

```text
clean everything
→ reinstall everything
```

---

# 📌 .PHONY

`.PHONY` prevents Make from confusing targets with real files.

---

# Example

```make
.PHONY: run clean install
```

Very common in professional Makefiles.

---

# ⚠️ Common Beginner Mistakes

## ❌ Using Spaces Instead of Tabs

Wrong:

```make
run:
 python3 main.py
```

Correct:

```make
run:
	python3 main.py
```

---

## ❌ Missing .PHONY

Without `.PHONY`, Make may behave incorrectly if files share target names.

---

## ❌ Using Global Python

Bad:

```make
python3 main.py
```

Better:

```make
venv/bin/python3 main.py
```

---

# 🧠 Recommended Workflow

1. Create virtual environment
2. Install requirements
3. Run project
4. Run linting
5. Clean cache files when needed

---

# 📚 Additional Notes

An example Makefile is included alongside this document for reference and practical usage examples.

---

# 📚 Final Notes

Understanding Makefiles is an important skill for Python development.

Even simple Makefiles can:
- save time
- automate repetitive tasks
- standardize workflows
- improve project organization

---
