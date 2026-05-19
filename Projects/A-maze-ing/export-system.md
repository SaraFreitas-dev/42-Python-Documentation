# Export System

This document explains the export and utility systems used in the **A-Maze-ing** project.

The export system is responsible for:
- Converting maze data into hexadecimal values
- Exporting maze information into a `.txt` file
- Converting paths into `N`, `E`, `S`, `W` directions
- Handling logical maze cell serialization

---

# 📁 Main Files

```text
utils/
├── export_utils.py
├── hex_utils.py
└── path_utils.py
```

---

# 🧠 Overview

The export system converts the generated maze into a structured output format.

The exported file contains:
- Hexadecimal maze representation
- Entry coordinates
- Exit coordinates
- BFS solution path directions

This output format follows the project requirements and allows the maze data to be reconstructed later.

---

# 📦 Export Pipeline

```text
Generated Maze
      │
      ▼
maze_to_hex()
      │
      ▼
path_to_directions()
      │
      ▼
export_maze()
      │
      ▼
output_maze.txt
```

---

# 📝 Main Export Function

Main file:

```text
utils/export_utils.py
```

Main function:

```python
def export_maze(
    maze: Maze,
    path: list[tuple[int, int]],
    output_file: str
) -> None:
```

This function:
- converts the maze into hexadecimal values
- extracts entry and exit coordinates
- converts the BFS path into directions
- writes all information into the output file

---

# 📄 Exported File Structure

The generated output file contains:

---

## 1. Hexadecimal Maze Representation

Each logical maze cell is converted into a hexadecimal value.

Example:

```text
F F F F
9 2 A 6
D 4 1 3
```

Each hexadecimal value represents which walls are closed around the cell.

---

## 2. Entry Coordinates

Example:

```text
0,0
```

---

## 3. Exit Coordinates

Example:

```text
19,19
```

---

## 4. Path Directions

Example:

```text
EESSWWNN
```

The directions represent the shortest BFS path from entry to exit.

---

# 🔢 Hexadecimal Conversion

Main file:

```text
utils/hex_utils.py
```

Main function:

```python
def maze_to_hex(maze: Maze) -> list[list[str]]:
```

---

# 🧠 How the Hex System Works

Each logical maze cell checks its surrounding walls:

| Direction | Bit Value |
|-----------|-----------|
| North     | 1 |
| East      | 2 |
| South     | 4 |
| West      | 8 |

The values are summed and converted into hexadecimal.

---

# 📌 Example

If a cell has:
- North wall
- West wall

then:

```text
1 + 8 = 9
```

Hex value:

```text
9
```

---

# 🧩 Expanded Grid Conversion

The maze internally uses an expanded grid system.

Logical coordinates are converted into grid coordinates:

```python
(y * 2 + 1), (x * 2 + 1)
```

This allows:
- walls
- paths
- openings

to exist between logical cells.

---

# 🧭 Path Direction Conversion

Main file:

```text
utils/path_utils.py
```

Main function:

```python
def path_to_directions(
    path: list[tuple[int, int]]
) -> list[str]:
```

This function converts BFS coordinates into:
- `N`
- `E`
- `S`
- `W`

directions.

---

# 📌 Example

Coordinate path:

```text
[(1,1), (1,2), (2,2)]
```

becomes:

```text
["E", "S"]
```

---

# 🖨️ File Printing Utility

The export system also contains a helper function:

```python
def print_maze_file(output_file: str) -> None:
```

This function:
- opens the exported file
- reads its contents
- prints the result to the terminal

It is mainly used for:
- debugging
- testing
- output verification

---

# 🚨 Error Handling

The export system handles:
- missing files
- permission errors
- invalid output access

using `try/except` blocks.

Example:

```text
print_maze_file() failed -> File not found
```

---

# 🎯 Goals of the Export System

The export system was designed to:
- serialize maze data
- match project output requirements
- simplify debugging
- convert internal structures into readable formats
- provide reusable utility functions

It acts as the final conversion layer between:
- maze generation
- pathfinding
- file output
- external representation