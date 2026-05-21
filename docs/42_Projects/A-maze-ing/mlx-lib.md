# MLX Library

This document explains how the **MiniLibX (MLX)** library is used in the **A-Maze-ing** project.

MiniLibX is a small graphical library commonly used in 42 School projects. It provides basic tools to:
- open a window
- load images
- draw images to the screen
- handle keyboard input
- listen to window events
- run a graphical loop

In this project, MLX is used to display the maze, menus, themes, player movement, BFS animations, and visual feedback.

---

# 📁 Main Files

```text
render/
├── mlx_renderer.py
├── draw_maze.py
├── menu.py
├── Assets.py
├── converter.py
└── GameState.py
```

---

# 🧠 What MLX Does

MLX works as the graphical layer of the project.

It does not generate the maze and it does not solve the maze.

Instead, it receives data from the project and displays it visually.

```text
Maze generation
      │
      ▼
Maze grid / path / state
      │
      ▼
MLX renderer
      │
      ▼
Window display
```

---

# 🪟 Creating a Window

The first step in any MLX program is initializing MLX and creating a window.

In this project, this happens inside:

```python
def mlx_window(game: GameState) -> None:
```

Example from the project:

```python
mlx = Mlx()
mlx_ptr = mlx.mlx_init()

win_ptr = mlx.mlx_new_window(
    mlx_ptr,
    WINDOW_WIDTH,
    WINDOW_HEIGHT,
    "A-MAZE-ING"
)
```

This creates the main game window.

The project uses:

```python
WINDOW_WIDTH = 1600
WINDOW_HEIGHT = 900
```

---

# 🖼️ Images in MLX

MiniLibX works best with `.xpm` images.

Because of that, the project converts image assets into XPM before loading them.

Examples of loaded assets:
- walls
- floors
- trails
- ducks
- exit doors
- banners
- menu backgrounds

---

# 📦 Asset Loading

The `Assets` class loads and stores the images used by the renderer.

Main file:

```text
render/Assets.py
```

Example:

```python
self.wall, _, _ = mlx.mlx_xpm_file_to_image(
    mlx_ptr,
    f"assets/generated/{wall_file}_{tile_size}.xpm"
)
```

This loads an XPM image and stores it so it can be drawn later.

---

# 🔄 PNG / JPEG to XPM Conversion

The project stores original assets as PNG or JPEG files.

Before MLX can use them, they are:
1. resized
2. converted to XPM
3. saved in `assets/generated/`

Main file:

```text
render/converter.py
```

Main functions:

```python
generate_scaled_asset()
convert_to_xpm()
generate_all_assets()
```

Example:

```python
os.system(f"magick {png_path} {xpm_path}")
```

The project uses ImageMagick through the `magick` command to perform the conversion.

---

# 📐 Dynamic Asset Scaling

The maze can have different sizes, so the tile size is calculated dynamically.

This prevents the maze from being too large for the screen.

Example:

```python
def calculate_tile_size(game: GameState) -> int:
```

The tile size depends on:
- maze width
- maze height
- available window width
- available window height
- banner space

Once the tile size is known, assets are generated for that exact size.

---

# 🧱 Drawing Images to the Window

MLX draws images using:

```python
mlx.mlx_put_image_to_window()
```

Example from the project:

```python
mlx.mlx_put_image_to_window(
    mlx_ptr,
    win_ptr,
    base_tile,
    screen_x,
    screen_y
)
```

This places an image at a specific pixel position.

---

# 🧩 Tile-Based Rendering

The maze is drawn tile by tile.

Each value in the maze grid represents a different visual element:

```text
0 = floor
1 = wall
2 = 42 wall
3 = duck/player
4 = trail/path
```

The renderer reads the grid and chooses which image to draw.

Example logic:

```python
if cell == 3:
    base_tile = assets.floor
elif cell == 0:
    base_tile = assets.floor
elif cell == 1:
    base_tile = assets.wall
elif cell == 2:
    base_tile = assets.wall_42
else:
    base_tile = assets.trail
```

---

# 🦆 Drawing the Player

The duck is drawn as a separate image on top of the floor.

This is important because the player should not permanently modify the maze grid.

Example:

```python
if cell == 3:
    mlx.mlx_put_image_to_window(
        mlx_ptr,
        win_ptr,
        assets.duck,
        screen_x,
        screen_y
    )
```

This allows the player to move without destroying walls, floors, or trails.

---

# 🚪 Drawing Entry and Exit

The maze entry and exit are logical cells.

For rendering, they are converted to expanded grid positions.

The renderer uses:

```python
logical_to_grid()
calculate_door_position()
```

These functions are used to draw the entry and exit doors in the correct position.

---

# 🧭 Coordinate Systems

The project uses two coordinate systems:

## Logical Coordinates

Used by the configuration file and maze structure.

Example:

```text
ENTRY=0,0
EXIT=19,19
```

## Grid Coordinates

Used by the expanded internal maze grid.

Conversion:

```python
grid_x = logical_x * 2 + 1
grid_y = logical_y * 2 + 1
```

This allows the renderer to draw:
- cells
- walls
- corridors
- doors

correctly.

---

# 🎮 Keyboard Input

MLX can react to keyboard input using hooks.

In this project, keyboard input is handled with:

```python
mlx.mlx_key_hook()
```

Example:

```python
mlx.mlx_key_hook(
    win_ptr,
    lambda key, param: key_hook(
        key,
        game,
        frame,
        param,
        mlx,
        mlx_ptr,
        win_ptr
    ),
    None
)
```

The `key_hook()` function controls:
- menu selection
- maze regeneration
- BFS animation
- player mode
- theme switching
- exiting the game

---

# ⌨️ Key Controls

The project uses numeric keys and arrow keys.

```text
1        Generate new maze
2        Show shortest path
3        Show BFS exploration animation
4        Player mode
5        Change theme
6 / ESC  Exit
Arrows   Move player
```

---

# 🎮 Game State and MLX

MLX itself does not know what mode the game is in.

The project stores that information inside:

```python
class GameState:
```

The renderer checks the game state every frame.

Example states:

```python
game.mode
game.theme
game.show_path
game.animate_bfs
game.playing
game.game_won
```

This allows MLX to draw different things depending on the current mode.

---

# 🔁 The MLX Loop

MLX uses a loop to keep the window alive.

In this project:

```python
mlx.mlx_loop_hook(
    mlx_ptr,
    on_loop,
    None
)

mlx.mlx_loop(mlx_ptr)
```

The loop repeatedly calls `on_loop()`.

This function updates:
- menu rendering
- maze rendering
- animations
- player movement display
- exit animation

---

# 🎞️ Animation With MLX

MLX does not provide a complex animation system by itself.

Instead, the project creates animation manually using:
- frame counters
- time checks
- partial path drawing
- repeated redraws

Example:

```python
frame = [0]
last_time = [time.time()]
```

The renderer only advances the animation when enough time has passed.

Example:

```python
if now - last_time[0] >= 0.05:
```

This prevents animations from running too fast.

---

# 🦆 Path Animation

Option 2 shows the shortest path.

The renderer progressively draws part of the path:

```python
game.path[:frame[0]]
```

This creates the effect of the duck moving through the maze.

---

# 🌊 BFS Exploration Animation

Option 3 shows how BFS explores the maze.

Instead of displaying only the final path, it displays explored cells:

```python
game.explored[:frame[0]]
```

This visualizes the search process step by step.

---

# 🚪 Exit Animation

The exit uses two images:

```python
assets.exit
assets.exit2
```

The renderer alternates between them using time:

```python
animation_frame = int(time.time() * 3.33) % 2
```

This creates a simple animated portal effect.

---

# 🧹 Clearing and Syncing the Window

Sometimes the renderer needs to clear the old screen before drawing a new state.

Example:

```python
mlx.mlx_clear_window(mlx_ptr, win_ptr)
```

The project also uses:

```python
mlx.mlx_do_sync(mlx_ptr)
```

to help synchronize visual updates.

This is useful when:
- changing themes
- regenerating the maze
- switching modes
- entering player mode

---

# ❌ Closing the Window

The project handles the window close event with:

```python
mlx.mlx_hook(
    win_ptr,
    33,
    0,
    close,
    None
)
```

The `close()` function exits the program:

```python
def close(param: Any) -> None:
    os._exit(0)
```

The user can also quit with:

```text
ESC
6
```

---

# 🎨 Themes With MLX

The renderer supports different visual themes.

Current themes:

```text
normal
gothic
```

The selected theme changes which assets are loaded.

Example:

```python
f"assets/generated/{theme}_floor_{tile_size}.xpm"
```

This allows the same maze logic to be displayed with different visual styles.

---

# 🧱 MLX Limitations

MLX is intentionally simple.

It does not provide:
- advanced UI widgets
- built-in animation tools
- sprite management
- automatic scaling
- layout systems
- scene management

Because of that, the project manually handles:
- asset conversion
- image scaling
- menu drawing
- animation timing
- player movement
- screen updates

---

# 🧠 Why MLX Was Used

MLX was used because:
- it is part of the 42 graphics ecosystem
- it teaches low-level rendering concepts
- it gives direct control over pixels and images
- it integrates well with simple game loops
- it is suitable for tile-based projects

For this project, MLX is enough to create:
- a graphical maze
- menus
- animations
- player movement
- theme switching

---

# 🏗️ Role of MLX in A-Maze-ing

MLX acts as the visual interface between the user and the maze engine.

```text
MazeGenerator
      │
      ▼
Maze + Path + Explored Cells
      │
      ▼
GameState
      │
      ▼
MLX Renderer
      │
      ▼
Window Output
```

The maze generation system stays independent from MLX.

This separation keeps the project modular:
- `mazegen/` handles maze logic
- `render/` handles graphics and interaction

---

# 🎯 Summary

In A-Maze-ing, MiniLibX is used to:
- open the game window
- load XPM images
- draw the maze
- render menus and banners
- handle keyboard input
- animate BFS and pathfinding
- display player movement
- support visual themes

The project builds a higher-level rendering system on top of MLX by combining:
- `GameState`
- dynamic assets
- tile rendering
- event hooks
- animation logic

This allows a simple graphics library to support a complete interactive maze visualizer.