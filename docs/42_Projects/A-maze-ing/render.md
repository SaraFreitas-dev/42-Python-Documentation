# Render System

This document explains the rendering systems used in the **A-Maze-ing** project.

The render system is responsible for:
- graphical visualization using MiniLibX
- menu and banner rendering
- asset loading and scaling
- player visualization
- animation systems
- real-time game updates

---

# 📁 Main Files

```text
render/
├── Assets.py
├── converter.py
├── draw_maze.py
├── GameState.py
├── menu.py
└── mlx_renderer.py
```

---

# 🧠 Overview

The rendering system combines:
- MiniLibX graphical rendering
- dynamic asset loading
- tile-based visualization
- gameplay systems
- real-time animations
- menu and UI rendering

The renderer is responsible for:
- drawing the maze
- displaying gameplay elements
- animating BFS/path systems
- handling themes
- rendering player movement
- updating the graphical interface in real time

---

# 🖼️ MLX Rendering System

Main file:

```text
render/mlx_renderer.py
```

Main function:

```python
def mlx_window(game: GameState) -> None:
```

This function:
- initializes MLX
- creates the game window
- loads graphical assets
- handles events
- updates the rendering loop

---

# 🪟 Window System

The game window uses:

```python
WINDOW_WIDTH = 1600
WINDOW_HEIGHT = 900
```

The renderer dynamically scales the maze to fit inside the window.

The bottom section of the screen is reserved for the banner UI.

---

# 🎮 Game Modes

The renderer supports multiple game states.

Main states:

| Mode | Description |
|------|-------------|
| MENU | Theme selection screen |
| GAME | Main gameplay screen |

Gameplay options:

| Key | Action |
|-----|--------|
| 1 | Generate new maze |
| 2 | Show BFS solution path |
| 3 | Show BFS exploration animation |
| 4 | Player mode |
| 5 | Change theme |
| ESC / 6 | Exit game |

---

# 🧩 GameState System

Main file:

```text
render/GameState.py
```

The `GameState` class stores all runtime game information.

```python
class GameState:
```

It stores:
- current mode
- selected theme
- animation states
- player position
- current maze
- BFS path
- explored nodes
- gameplay flags

---

# 🧠 Main Stored Data

Examples:

```python
self.mode
self.theme
self.playing
self.show_path
self.animate_bfs
self.player_x
self.player_y
self.path
self.explored
```

The class acts as a centralized runtime state manager.

---

# 🎨 Asset System

Main file:

```text
render/Assets.py
```

The `Assets` class loads all XPM images used by the game.

```python
class Assets:
```

Loaded assets include:
- walls
- floors
- trails
- ducks
- exit doors
- animated exits
- banners

---

# 🖼️ Theme System

The renderer supports multiple themes.

Current themes:

```text
normal
gothic
```

The selected theme changes:
- walls
- floors
- ducks
- trails
- exits

The assets are loaded dynamically depending on:
- theme
- tile size

---

# 🔄 Asset Conversion System

Main file:

```text
render/converter.py
```

The project stores original assets as:
- PNG
- JPEG

They are dynamically converted into:
- XPM

because MiniLibX requires the XPM format.

---

# 📦 Asset Generation Pipeline

The conversion system:

1. loads the original image
2. resizes the image
3. converts the image into XPM
4. stores the generated file

Main functions:

```python
generate_scaled_asset()
convert_to_xpm()
generate_all_assets()
```

---

# 📐 Dynamic Tile Scaling

The renderer dynamically calculates tile size:

```python
def calculate_tile_size(game: GameState) -> int:
```

This allows:
- small mazes
- large mazes
- fullscreen scaling

without manually resizing assets.

---

# 🧱 Maze Drawing System

Main file:

```text
render/draw_maze.py
```

Main function:

```python
def draw_maze(...)
```

This function:
- renders the maze grid
- draws walls and floors
- draws the player
- draws BFS trails
- renders the animated exit
- centers the maze on screen

---

# 🧩 Coordinate Conversion

The renderer uses helper functions:

```python
logical_to_grid()
grid_to_logical()
```

These functions convert:
- logical maze coordinates
- expanded grid coordinates

This keeps rendering synchronized with:
- BFS
- DFS
- gameplay movement

---

# 🚪 Door Rendering

The renderer calculates visual entry and exit doors using:

```python
calculate_door_position()
```

This function:
- determines which maze edge the door belongs to
- offsets the position correctly
- renders the external maze openings

---

# 🎞️ Animation System

The project supports several animations.

---

# 🦆 Path Animation

Option 2 displays:
- the shortest BFS path
- the duck progressively moving through the maze

The animation progressively reveals:

```python
game.path[:frame]
```

---

# 🌊 BFS Exploration Animation

Option 3 visualizes:
- BFS traversal
- explored cells
- search progression

The animation uses:

```python
game.explored[:frame]
```

to progressively display explored positions.

---

# 🚪 Exit Animation

When:
- BFS finishes
- the player reaches the exit

the exit portal becomes animated by alternating between:

```python
assets.exit
assets.exit2
```

using timed frame switching.

---

# 🎮 Player Mode

Player mode allows real-time maze traversal using arrow keys.

Main function:

```python
handle_player_movement()
```

Main features:
- collision detection
- smooth movement
- trail rendering
- backtracking
- exit detection

---

# 🧱 Collision Detection

The player cannot move through:
- walls (`1`)
- logo blocks (`2`)

The movement system validates:
- boundaries
- blocked cells
- valid traversal positions

before updating the player position.

---

# 👣 Player Trail System

The renderer stores the player path:

```python
game.player_path
```

This allows:
- visual backtracking
- persistent player trails
- movement visualization

When the player walks backwards:
- the trail is removed dynamically

---

# 📜 Menu System

Main file:

```text
render/menu.py
```

The menu system handles:
- menu backgrounds
- banner rendering
- menu image loading
- dynamic image generation

---

# 🖼️ Banner Rendering

The bottom banner acts as the game's UI.

It contains:
- gameplay options
- controls
- theme information

The banner is rendered independently from the maze.

---

# ⚡ Rendering Loop

The MLX loop uses:

```python
mlx.mlx_loop_hook()
```

This continuously:
- redraws the screen
- updates animations
- refreshes gameplay state
- synchronizes rendering

The project also uses:

```python
mlx.mlx_do_sync()
```

to improve rendering consistency.

---

# 🧠 Rendering Goals

The rendering system was designed to:
- provide real-time visualization
- separate graphics from algorithms
- support gameplay interaction
- create animated algorithm demonstrations
- dynamically scale assets
- keep the project modular

The renderer acts as the visual layer between:
- maze generation
- gameplay systems
- BFS/DFS algorithms
- user interaction