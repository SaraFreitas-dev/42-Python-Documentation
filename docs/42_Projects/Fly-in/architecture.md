# 🏗️ Fly-in Architecture
> Understanding how the project components interact and why the system is separated into multiple responsibilities.

---

# 📚 Table of Contents

1. Why an Architecture?
2. High-Level Overview
3. MapParser
4. Simulator
5. PathFinder
6. Visualizer
7. ImageGenerator
8. Data Flow
9. Why Separate Classes?
10. Single Responsibility Principle
11. Benefits of the Architecture
12. Mental Model

---

# 1️⃣ Why an Architecture?

As projects grow, putting everything into a single file quickly becomes difficult to maintain.

Instead of creating one massive class responsible for:

- parsing
- pathfinding
- simulation
- visualization
- rendering

Fly-in separates each responsibility into dedicated systems.

This makes the project:

✅ easier to understand

✅ easier to debug

✅ easier to extend

✅ easier to test

---

# 2️⃣ High-Level Overview

```text
Map File (.txt)
       │
       ▼
┌─────────────────┐
│   MapParser     │
└─────────────────┘
       │
       ▼
┌─────────────────┐
│   Simulator     │
└─────────────────┘
       │
       │
       ▼
┌─────────────────┐
│   PathFinder    │
└─────────────────┘
       │
       ├─────────────► Visualizer
       │                  │
       │                  ▼
       │            Terminal Output
       ▼
┌─────────────────┐
│ ImageGenerator  │
└─────────────────┘
       │
       ▼
 Frames + GIF
```

---

# 3️⃣ MapParser

## Responsibility

The parser transforms the map file into Python objects that can be used by the simulation.

Input:

```txt
connection: A-B
```

Output:

```python
Connection("A", "B")
```

---

## What it validates

The parser verifies:

- syntax correctness
- duplicated zones
- duplicated connections
- invalid metadata
- invalid coordinates
- invalid capacities
- missing start/end zones

---

## What it creates

```python
zones
connections
start_zone
end_zone
nb_drones
```

Everything else in the project depends on this data.

---

# 4️⃣ Simulator

## Responsibility

The Simulator is the heart of the project.

It decides:

```text
Who moves
When they move
Whether movement is valid
```

---

## Tracks

```python
occupied_zones
occupied_links
current_turn
turn_states
```

---

## Handles

- drone movement
- zone capacities
- link capacities
- waiting turns
- restricted zones
- delivery tracking
- turn progression

---

## Think of it as

```text
Air Traffic Control
```

The Simulator controls movement but does not calculate routes.

---

# 5️⃣ PathFinder

## Responsibility

The PathFinder answers one question:

```text
What is the best route?
```

---

## Uses

```python
Dijkstra Algorithm
heapq Priority Queue
```

---

## Considers

- movement costs
- restricted zones
- blocked zones
- priority zones
- occupied areas

---

## Returns

```python
[
    "start",
    "hub",
    "corridor",
    "goal"
]
```

The Simulator then decides whether that route can be used.

---

# 6️⃣ Visualizer

## Responsibility

The Visualizer displays the simulation in the terminal.

It converts raw simulation data into a format that humans can easily understand.

---

## Input

The Visualizer receives:

```python
simul_result
```

Example:

```python
{
    1: ["D1-hub", "D2-hub"],
    2: ["D1-corridor", "D3-hub"]
}
```

---

## Output

```text
Turn 1
🛸 D1 ➤ hub
🛸 D2 ➤ hub

Turn 2
🛸 D1 ➤ corridor
🛸 D3 ➤ hub
```

---

## Responsibilities

The Visualizer:

- displays turn-by-turn movements
- formats terminal output
- generates reports
- improves readability
- assists debugging

---

## Final Report

Example:

```text
══════════════════════════════════════
       FLY-IN DRONE SIMULATION
══════════════════════════════════════

Total drones: 25
Delivered drones: 25
Total turns: 41
Maximum turns allowed: 45

Simulation Status: SUCCESS
```

---

## Why a Separate Class?

Displaying information is not part of simulation logic.

Separating responsibilities makes future output systems easier to create:

```text
Terminal Output
JSON Output
Web Output
GUI Output
```

---

# 7️⃣ ImageGenerator

## Responsibility

The ImageGenerator creates a visual representation of the simulation.

---

## Generates

📸 Individual Frames

```text
frame_000.png
frame_001.png
frame_002.png
...
```

---

🎬 Final GIF

```text
simulation.gif
```

---

## Uses

```python
Pillow
```

to render:

- zones
- connections
- drone distribution
- turn information
- simulation statistics

---

## Purpose

The ImageGenerator is not required for the simulation itself.

It was developed to:

- improve user experience
- provide visual debugging
- better understand drone movement
- practice Pillow outside the project's requirements

---

# 8️⃣ Data Flow

Complete execution flow:

```text
Map File
    ↓
MapParser
    ↓
Zones + Connections
    ↓
Simulator
    ├──→ Visualizer
    │         ↓
    │   Terminal Output
    │
    └──→ PathFinder
              ↓
      Movement Decisions
              ↓
        Turn States
              ↓
      ImageGenerator
              ↓
        Frames + GIF
```

---

# 9️⃣ Why Separate Classes?

A common question during evaluations is:

## Why not put everything inside Simulator?

Because it would create a massive class responsible for:

- parsing
- pathfinding
- simulation
- terminal output
- image generation

This violates the:

```text
Single Responsibility Principle
```

and makes maintenance harder.

---

## Example

If a bug exists in:

```text
Map Parsing
```

the fix belongs in:

```python
MapParser
```

not inside the Simulator.

---

# 🔟 Single Responsibility Principle

One class.

One responsibility.

---

## MapParser

```text
Reads and validates input
```

---

## Simulator

```text
Controls simulation rules
```

---

## PathFinder

```text
Calculates optimal routes
```

---

## Visualizer

```text
Displays terminal output
```

---

## ImageGenerator

```text
Creates visual output
```

---

This separation keeps the codebase clean and maintainable.

---

# 1️⃣1️⃣ Benefits of the Architecture

Because responsibilities are isolated:

✅ easier debugging

✅ easier testing

✅ easier maintenance

✅ easier feature development

✅ lower coupling between systems

---

## Example

Want to replace Dijkstra with A*?

Only:

```python
PathFinder.py
```

requires significant changes.

The rest of the project remains untouched.

---

# 1️⃣2️⃣ Mental Model

Think of the project as an airport.

---

## 🧾 MapParser

```text
Airport Construction Team
```

Builds the airport layout.

---

## 🧭 PathFinder

```text
GPS Navigation System
```

Calculates routes.

---

## 🎮 Simulator

```text
Air Traffic Control
```

Controls movement rules.

---

## 📺 Visualizer

```text
Control Tower Screens
```

Displays what is happening in real time.

---

## 🎬 ImageGenerator

```text
Security Cameras
```

Records and visualizes the entire simulation.

---

Each component has a clear purpose and communicates only the information required for the next stage of the simulation.