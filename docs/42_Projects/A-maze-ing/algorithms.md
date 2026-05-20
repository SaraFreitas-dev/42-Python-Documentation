# Algorithms

This document explains the core algorithms used in the **A-Maze-ing** project.

The project mainly relies on:
- Depth-First Search (DFS) for maze generation
- Breadth-First Search (BFS) for maze solving

These algorithms are responsible for:
- creating valid mazes
- generating paths
- solving the maze
- visualizing exploration and traversal

---

# 📁 Main Files

```text
mazegen/
├── generator.py
└── solver.py
```

---

# 🧠 Overview

The project uses two different graph traversal algorithms:

| Algorithm | Purpose |
|-----------|----------|
| DFS | Maze generation |
| BFS | Shortest path solving |

Both algorithms operate on the maze's expanded grid representation.

---

# 🌱 DFS Maze Generation

Main function:

```python
def dfs_generator(maze: Maze) -> None:
```

Main file:

```text
mazegen/generator.py
```

---

# 🧩 Goal of DFS

DFS is used to:
- carve paths through walls
- visit every logical cell
- create a valid maze structure
- generate a perfect maze

A perfect maze means:
- every cell is reachable
- there is only one valid path between cells
- no loops exist

---

# 🧠 Expanded Grid System

The maze internally uses an expanded grid.

Logical cells are separated by walls.

Example:

```text
# # # # #
# . # . #
# # # # #
# . # . #
# # # # #
```

Logical coordinates are converted using:

```python
(y * 2 + 1), (x * 2 + 1)
```

This allows the algorithm to:
- carve walls
- connect cells
- preserve maze structure

---

# 🔄 DFS Traversal Logic

The DFS algorithm:

1. Starts at the first logical cell
2. Marks the current cell as visited
3. Randomly selects a direction
4. Moves two cells away
5. Breaks the wall between cells
6. Recursively continues traversal

---

# 🎲 Randomized Directions

The directions are shuffled using:

```python
random.sample(directions, len(directions))
```

This guarantees:
- randomized mazes
- different layouts every execution
- procedural generation behavior

---

# 🪓 Wall Breaking

When DFS moves to a new cell:

```text
Current Cell -> Wall -> Next Cell
```

the wall between both cells is removed.

This creates:
- corridors
- connected paths
- valid maze routes

---

# 📌 DFS Characteristics

| Property | Behavior |
|----------|-----------|
| Traversal Type | Recursive |
| Path Type | Deep exploration |
| Maze Style | Long corridors |
| Randomness | High |
| Guarantees | Fully connected maze |

---

# 🚪 Entry and Exit Integration

After DFS generation:

```python
apply_entry_exit(maze)
```

opens:
- the entry cell
- the exit cell
- external maze doors

The project validates that:
- entry and exit are located on maze borders
- coordinates are valid
- doors connect correctly with the expanded grid

---

# 🛡️ Maze Validation

The generation system validates:
- invalid open spaces
- malformed structures
- impossible layouts

Main validation:

```python
check_open_areas()
```

This prevents:
- large empty spaces
- invalid 3x3 open blocks

---

# 🧱 Imperfect Maze Generation

If the maze is configured as:

```text
PERFECT=False
```

the algorithm uses:

```python
break_walls()
```

to:
- remove extra walls
- create loops
- generate multiple possible paths

This transforms the perfect DFS maze into an imperfect maze.

---

# 🔎 BFS Maze Solving

Main function:

```python
def bfs_solve_maze(maze: Maze)
```

Main file:

```text
mazegen/solver.py
```

---

# 🎯 Goal of BFS

BFS is used to:
- find the shortest path
- solve the maze
- reconstruct the optimal route
- generate exploration animations

---

# 📦 BFS Data Structures

The solver uses:

| Structure | Purpose |
|-----------|----------|
| Queue (`deque`) | Exploration order |
| Set (`visited`) | Prevent revisits |
| Dictionary (`parent`) | Path reconstruction |
| List (`explored`) | Animation visualization |

---

# 🌊 BFS Traversal Logic

The BFS algorithm:

1. Starts at the entry point
2. Explores neighboring cells
3. Expands layer by layer
4. Continues until the exit is found
5. Reconstructs the shortest path

Unlike DFS:
- BFS explores breadth-first
- guarantees the shortest path

---

# 🧭 Valid Movement Rules

The solver ignores:
- walls (`1`)
- logo cells (`2`)
- already visited cells
- out-of-bounds positions

This ensures:
- safe traversal
- valid pathfinding
- no infinite loops

---

# 🧩 Path Reconstruction

Once the exit is found:

```python
parent[(new_y, new_x)] = (y, x)
```

stores:
- where each cell came from

The algorithm then reconstructs the path:
- backwards from exit
- until the entry point is reached

Finally:
- the path is reversed
- producing the final solution order

---

# 🎞️ Explored Cells Animation

BFS also stores:

```python
explored.append((new_y, new_x))
```

This allows the renderer to:
- visualize exploration
- animate the solver
- display visited cells progressively

---

# 📌 BFS Characteristics

| Property | Behavior |
|----------|-----------|
| Traversal Type | Iterative |
| Exploration Style | Layer-by-layer |
| Path Guarantee | Shortest path |
| Memory Usage | Higher than DFS |
| Animation Friendly | Yes |

---

# ⚖️ DFS vs BFS

| DFS | BFS |
|-----|-----|
| Generates mazes | Solves mazes |
| Recursive | Iterative |
| Deep traversal | Breadth traversal |
| Randomized | Deterministic |
| Creates paths | Finds shortest path |

---

# 🎯 Algorithm Goals

The algorithm system was designed to:
- generate valid procedural mazes
- guarantee solvable layouts
- provide visual algorithm demonstrations
- support animation systems
- separate generation and solving logic cleanly

The combination of DFS and BFS creates:
- randomized maze structures
- reliable shortest-path solving
- interactive visualization systems
- reproducible procedural generation