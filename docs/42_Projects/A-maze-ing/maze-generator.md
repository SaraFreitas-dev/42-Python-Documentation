# Maze Generator

This document explains the reusable maze generation module used in the **A-Maze-ing** project.

The maze generator is the core system responsible for:
- creating the maze structure
- applying entry and exit points
- validating maze rules
- solving the maze
- exporting the final result
- providing a reusable package through `.whl`

---

# 📁 Main Files

```text
mazegen/
├── __init__.py
├── Maze.py
├── MazeGenerator.py
├── generator.py
└── solver.py
```

---

# 🧱 Maze Class

Main file:

```text
mazegen/Maze.py
```

The `Maze` class stores the basic maze data.

```python
class Maze:
```

It contains:
- maze width
- maze height
- entry coordinates
- exit coordinates
- expanded grid
- expanded grid width
- expanded grid height

---

# 🧩 Expanded Grid

The project uses an expanded grid representation.

For a maze with logical size:

```text
WIDTH x HEIGHT
```

the internal grid size becomes:

```text
(2 * WIDTH + 1) x (2 * HEIGHT + 1)
```

This allows the maze to store:
- logical cells
- walls between cells
- external borders
- opened doors

Example:

```text
logical cell -> expanded position

(x, y) -> (x * 2 + 1, y * 2 + 1)
```

---

# 🏗️ MazeGenerator Class

Main file:

```text
mazegen/MazeGenerator.py
```

The `MazeGenerator` class controls the full maze generation pipeline.

```python
class MazeGenerator:
```

It receives:

```python
width: int
height: int
entry: tuple[int, int]
exit: tuple[int, int]
perfect: bool
seed: int | None
```

---

# ⚙️ Generation Pipeline

The main method is:

```python
def generate_maze(self) -> Maze:
```

This method coordinates the helper functions from `generator.py` and `solver.py`.

Pipeline:

```text
MazeGenerator.generate_maze()
        │
        ▼
Create Maze object
        │
        ▼
dfs_generator()
        │
        ▼
apply_entry_exit()
        │
        ▼
check_open_areas()
        │
        ▼
add_42_logo()
        │
        ▼
break_walls() if PERFECT=False
        │
        ▼
bfs_solve_maze()
        │
        ▼
Return valid Maze
```

---

# 🌱 DFS Integration

The generator calls:

```python
dfs_generator(maze)
```

from:

```text
mazegen/generator.py
```

This function:
- carves the maze paths
- visits reachable cells
- creates the base perfect maze structure

The DFS details are documented in:

```text
algorithms.md
```

---

# 🚪 Entry and Exit

The generator calls:

```python
apply_entry_exit(maze)
```

This function:
- converts logical entry and exit coordinates into expanded grid coordinates
- opens the internal entry and exit cells
- validates that entry and exit are placed on maze borders
- opens the external door for each one

---

# 🛡️ Maze Validation

The generator calls:

```python
check_open_areas(maze)
```

This function prevents invalid open spaces by rejecting mazes containing a fully open `3x3` area.

If the generated maze does not pass validation, the generator tries again.

---

# 4️⃣ 42 Logo Integration

The generator calls:

```python
add_42_logo(maze)
```

This function adds the visible `42` pattern to the maze.

The logo is represented inside the expanded grid using closed cells.

If the maze is too small for the logo, the function returns an empty list and the program prints a warning message.

---

# 🧱 Perfect and Imperfect Mazes

The project supports two maze types through the `PERFECT` configuration value.

---

## PERFECT=True

The maze keeps the original DFS-generated structure.

This means:
- the maze is fully connected
- the maze has a single main route structure
- no extra walls are removed after generation

---

## PERFECT=False

The generator calls:

```python
break_walls(maze, logo_pos)
```

This function:
- removes additional walls
- creates extra routes
- makes the maze imperfect
- protects the `42` logo cells from being overwritten

---

# 🔎 BFS Integration

After generation, the maze is solved using:

```python
bfs_solve_maze(maze)
```

from:

```text
mazegen/solver.py
```

This confirms that:
- the maze is solvable
- a valid path exists between entry and exit

If BFS cannot find a path, the generation attempt is rejected and the generator tries again.

---

# 🧭 Solver Method

The class also provides:

```python
def solve(self, algorithm: str) -> tuple[list[tuple[int, int]], list[tuple[int, int]]]:
```

Currently supported algorithm:

```text
bfs
```

The method returns:
- the final shortest path
- explored cells used for pathfinding animation

Example:

```python
path, explored = generator.solve("bfs")
```

---

# 📤 Export Method

The class provides:

```python
def export(self, output_file: str) -> None:
```

This method calls the export system:

```python
export_maze(self.maze, self.path, output_file)
```

The export logic is documented in:

```text
export-system.md
```

---

# 📦 Reusable Package

The `mazegen` directory is designed to work as a reusable Python package.

This allows the maze generation logic to be used outside the main graphical application.

The reusable package contains:
- `Maze`
- `MazeGenerator`
- DFS generation helpers
- BFS solving helpers

The MLX rendering, menu system, assets, and gameplay logic are not part of the reusable package.

---

# 📄 Package Public API

The package exposes its main classes through:

```text
mazegen/__init__.py
```

Example:

```python
from .MazeGenerator import MazeGenerator
from .Maze import Maze

__all__ = [
    "MazeGenerator",
    "Maze"
]
```

This allows external projects to import:

```python
from mazegen import MazeGenerator
```

---

# 🛠️ Building the `.whl`

The package can be built using Python build tools.

Install build:

```bash
pip install build
```

Build the package:

```bash
python -m build
```

This creates:

```text
dist/
├── mazegen-1.0.0.tar.gz
└── mazegen-1.0.0-py3-none-any.whl
```

---

# 📥 Installing the Package

The generated wheel can be installed with:

```bash
pip install dist/mazegen-1.0.0-py3-none-any.whl
```

or force reinstalled with:

```bash
pip install --force-reinstall dist/mazegen-1.0.0-py3-none-any.whl
```

---

# 🧪 Example Usage

After installing the wheel:

```python
from mazegen import MazeGenerator

generator = MazeGenerator(
    width=20,
    height=20,
    entry=(0, 0),
    exit=(19, 19),
    perfect=True,
    seed=42
)

maze = generator.generate_maze()
path, explored = generator.solve("bfs")
```

---

# 🎯 Purpose of the Maze Generator Module

The maze generator module was designed to:
- keep the maze logic separate from rendering
- make the project easier to maintain
- support reuse in future projects
- provide a clean API
- satisfy the reusable module requirement
- allow package distribution through `.whl`

This separation makes the project more modular by keeping the core maze engine independent from the visual MLX application.