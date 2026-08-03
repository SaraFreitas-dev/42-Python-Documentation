# ⚡ UV — Modern Python Package & Environment Management

> **Why this doc exists:** Previous projects at 42 used plain `venv` + `pip`. This project uses **UV** — a faster, more complete replacement. 

## 🎯 Purpose

This guide focuses on:

-   What UV is
-   Why UV is becoming popular
-   How UV compares to venv
-   Understanding `pyproject.toml`
-   Understanding `uv.lock`
-   Typical UV workflows
-   Using UV through a Makefile
---

## Table of Contents

1. [What is UV?](#what-is-uv)
2. [Why not just use venv + pip?](#why-not-just-use-venv--pip)
3. [Installing UV](#installing-uv)
4. [Key Concepts](#key-concepts)
   - [pyproject.toml](#pyprojecttoml)
   - [uv.lock](#uvlock)
5. [Getting Started with this Project](#getting-started-with-this-project)
6. [Common UV Commands](#common-uv-commands)
7. [UV vs venv + pip — Side by Side](#uv-vs-venv--pip--side-by-side)
8. [Using the Makefile](#using-the-makefile)
9. [Project Structure](#project-structure)
10. [FAQ](#faq)

---

## What is UV?

UV is a Python package and environment manager built by [Astral](https://astral.sh) (the same team behind the `ruff` linter). It's written in **Rust**, which is a big part of why it's so fast.

Think of UV as a single tool that replaces:

| Old Tool | UV Equivalent |
|----------|--------------|
| `python -m venv` | `uv venv` |
| `pip install` | `uv add` / `uv sync` |
| `pip freeze` | automatic via `uv.lock` |
| `pip-tools` / `pip-compile` | built-in |

The goal: **one tool, reproducible environments, no surprises.**

---

## Why not just use venv + pip?

If you've worked on other 42 Python projects, you're probably used to this flow:

```bash
python3 -m venv venv
source venv/bin/activate
pip install requests
pip freeze > requirements.txt
```

This works for small solo scripts. But it has real problems in team or multi-machine scenarios.

### The "works on my machine" problem

Imagine this:

```
You add `requests` to your project in January.
A friend clones the repo in April.
pip installs requests==2.32.0 for you, requests==2.33.1 for them.
Subtle differences in behaviour. Debugging nightmare.
```

With UV and a lock file, both of you get **exactly the same versions, every time.**

### pip is slow

UV resolves and installs packages significantly faster than pip. On a fresh install of a project with many dependencies, the difference is very noticeable.

### requirements.txt is fragile

`pip freeze` dumps every installed package — including things that got pulled in indirectly. The file isn't structured, doesn't track dev vs runtime deps, and drifts over time.

UV uses `pyproject.toml` + `uv.lock` instead — structured, reliable, and version-controlled properly.

---

## Installing UV

### Linux / macOS

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Then restart your terminal (or `source ~/.bashrc` / `source ~/.zshrc`) and verify:

```bash
uv --version
# uv 0.x.x (...)
```

### Windows

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

> **42 School machines (Linux):** The `curl` command above is the one to use. UV installs to `~/.local/bin/` by default, which should already be in your PATH.

---

## Key Concepts

### `pyproject.toml`

This is the **declaration** of your project. It's the single source of truth for:

- Project name and version
- Runtime dependencies (things the code needs to run)
- Dev dependencies (things you need during development, like test runners)
- Python version constraints

Example from this project:

```toml
[project]
name = "call-me-maybe"
version = "0.1.0"
description = "42 Python project — socket programming"
requires-python = ">=3.10"

dependencies = [
    "requests>=2.28.0",
]

[tool.uv]
dev-dependencies = [
    "pytest>=7.0",
]
```

**You edit `pyproject.toml` directly, or let UV update it when you run `uv add`.** Think of it like a contract: "this project needs at least these things."

---

### `uv.lock`

This is the **resolved snapshot** of your environment. UV generates and updates it automatically — you never edit it by hand.

It records the **exact** versions of every package (including transitive dependencies — packages that your packages depend on):

```
# Example excerpt from uv.lock
[[package]]
name = "requests"
version = "2.32.3"
source = { registry = "https://pypi.org/simple" }

[[package]]
name = "urllib3"
version = "2.5.0"

[[package]]
name = "certifi"
version = "2026.01.01"
```

#### Why does this matter?

```
pyproject.toml says:  "requests>=2.28.0"   ← flexible requirement
uv.lock says:         "requests==2.32.3"   ← exact pinned version
```

When you run `uv sync`, UV reads the lock file and installs those exact versions — not whatever happens to be latest today. This means:

- You, your teammate, and the CI pipeline all get identical environments
- No more debugging version-related differences across machines
- Rolling back is easy — just revert the lock file in git

**Both `pyproject.toml` and `uv.lock` should be committed to git.**

---

## Getting Started with this Project

Clone the repo and set up your environment in three commands:

```bash
git clone <repo-url>
cd call-me-maybe
uv sync
```

That's it. UV will:
1. Read `pyproject.toml` and `uv.lock`
2. Create a `.venv/` directory
3. Install all dependencies at the exact pinned versions

Then run the project:

```bash
uv run python main.py
```

Or run tests:

```bash
uv run pytest
```

> **Note:** You don't need to activate the virtual environment manually. `uv run` handles that for you. If you prefer the traditional `source .venv/bin/activate`, that still works too.

---

## Common UV Commands

### Adding a dependency

```bash
uv add requests
```

UV will:
- Install the package
- Add it to `pyproject.toml` under `[project] dependencies`
- Update `uv.lock` with the resolved versions

Adding a dev-only dependency (e.g. a testing library):

```bash
uv add --dev pytest
```

### Removing a dependency

```bash
uv remove requests
```

### Syncing your environment

After pulling new changes from git (someone else may have added dependencies):

```bash
uv sync
```

This reads the lock file and makes your local environment match it exactly. It's the equivalent of `pip install -r requirements.txt`, but more reliable.

### Running code

```bash
uv run python main.py
uv run pytest
uv run python -c "import sys; print(sys.version)"
```

### Upgrading a dependency

```bash
uv add requests --upgrade
```

This upgrades `requests` to the latest compatible version and updates `uv.lock`.

### Updating all dependencies

```bash
uv lock --upgrade
uv sync
```

### Checking what's installed

```bash
uv pip list
```

---

## UV vs venv + pip — Side by Side

Here's the same task done both ways:

### Setting up a fresh project

**Old way (venv + pip):**
```bash
python3 -m venv venv
source venv/bin/activate
pip install requests pytest
pip freeze > requirements.txt
# requirements.txt now has ~20 packages, including ones you didn't ask for
```

**UV way:**
```bash
uv init my_project
cd my_project
uv add requests
uv add --dev pytest
# pyproject.toml is clean, uv.lock has the full resolution
```

---

### Cloning someone else's project

**Old way:**
```bash
git clone <repo>
cd repo
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
# Hope the versions in requirements.txt are still installable...
```

**UV way:**
```bash
git clone <repo>
cd repo
uv sync
# Done. Exact same environment as everyone else.
```

---

### Feature comparison

| Feature | venv + pip | UV |
|---------|-----------|-----|
| Virtual environment | ✅ | ✅ |
| Package installation | ✅ | ✅ |
| Lock file | ❌ (manual with pip-tools) | ✅ automatic |
| Reproducible installs | ⚠️ limited | ✅ guaranteed |
| Dev vs runtime deps | ❌ manual convention | ✅ built-in |
| Speed | normal | very fast (Rust) |
| Single tool | ❌ | ✅ |
| Works without activating venv | ❌ | ✅ (via `uv run`) |

---

## Using the Makefile

The `Makefile` in this project is just a shortcut layer on top of UV commands. You don't need it, but it makes the most common tasks quick to type:

```makefile
install:
	uv sync

run:
	uv run python main.py

test:
	uv run pytest

clean:
	rm -rf .venv __pycache__ .pytest_cache
```

Usage:

```bash
make install   # set up / sync the environment
make run       # run the project
make test      # run tests
make clean     # remove generated files
```

---

## Project Structure

```
call-me-maybe/
├── .venv/              ← created by UV, do NOT commit (in .gitignore)
├── src/
│   └── *.py
├── tests/
│   └── test_*.py
├── docs/
│   └── UV_GUIDE.md     ← you are here
├── pyproject.toml      ← commit this ✅
├── uv.lock             ← commit this ✅
├── Makefile
└── README.md
```

> **`.venv/` is in `.gitignore`.** Each developer generates their own local environment via `uv sync`. You never commit the virtual environment itself.

---

## FAQ

**Q: Do I still need to activate the virtual environment?**

You don't have to. `uv run <command>` automatically uses the project's `.venv`. If you prefer the traditional workflow, `source .venv/bin/activate` still works exactly as before.

---

**Q: Someone added a new package. How do I get it?**

```bash
git pull
uv sync
```

UV will detect that `uv.lock` changed and install the new package.

---

**Q: Can I still use `pip` inside the project?**

Technically yes (if you activate `.venv` first), but you shouldn't — pip won't update `uv.lock`, so your changes won't be tracked. Use `uv add` instead.

---

**Q: Should I commit `uv.lock`?**

**Yes, always.** The lock file is what makes installs reproducible. Without it, two people running `uv sync` on different days might get different package versions.

---

**Q: What's the difference between `uv add` and `uv sync`?**

- `uv add` — installs a new package and records it in `pyproject.toml` + `uv.lock`
- `uv sync` — makes your environment match the current `uv.lock` (used after pulling from git, or setting up a fresh clone)

---

**Q: UV isn't found after installing (`command not found`)**

Make sure `~/.local/bin` is in your PATH:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Add that line to your `~/.bashrc` or `~/.zshrc` to make it permanent.

---

*For the official UV docs: [docs.astral.sh/uv](https://docs.astral.sh/uv)*