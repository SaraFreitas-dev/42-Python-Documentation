# Simulator System
> Understanding how the simulation engine coordinates drones, turns and movement rules in Fly-in.

---

# Table of Contents

1. What is the Simulator?
2. Main Responsibilities
3. Simulation State
4. Drone Lifecycle
5. Turn System
6. Occupancy Tracking
7. Capacity Management
8. Waiting States
9. Movement Validation
10. Path Updates
11. End Conditions
12. Simulator vs PathFinder
13. Mental Model

---

# 1️⃣ What is the Simulator?

The Simulator is the component responsible for:

```text
coordinating all drone activity
```

While other systems may:

- load map data
- validate inputs
- calculate routes

the Simulator controls:

```text
what happens each turn
```

Think of it as the system that applies the project rules and keeps the simulation progressing.

---

# 2️⃣ Main Responsibilities

The Simulator typically manages:

- 🛩️ drone movement
- ⏱️ turn progression
- 📦 occupancy tracking
- 🚦 rule enforcement
- 📄 output generation

---

# 3️⃣ Simulation State

During execution, the Simulator keeps track of information such as:

```python
current_turn = 7
active_drones = [...]
occupied_zones = {...}
occupied_links = {...}
```

The exact implementation may differ, but the idea remains the same:

```text
store everything needed to make decisions
```

---

# 4️⃣ Drone Lifecycle

A drone normally moves through several states.

```text
Created
   ↓
Waiting
   ↓
Moving
   ↓
Waiting (if required)
   ↓
Moving again
   ↓
Delivered
```

Not every drone will follow exactly the same route, but they all follow the same simulation rules.

---

# 5️⃣ Turn System

Fly-in is a:

```text
turn-based simulation
```

Each turn represents a small step of time.

Example:

```text
Turn 1
D1 -> A
D2 -> B

Turn 2
D1 -> C
D2 -> D
```

The Simulator evaluates every active drone and decides what can happen during that turn.

A simulation turn usually follows:

```text
Check drones
    ↓
Validate movement
    ↓
Move drones
    ↓
Update occupancy
    ↓
Store turn result

The exact implementation may vary.

The important concept is:

all rules are applied consistently
```

---

# 6️⃣ Occupancy Tracking

To avoid conflicts, the Simulator usually tracks:

- occupied zones
- occupied connections
- active movements

Example:

```text
Zone A
 ├─ D1
 └─ D2
```

Without occupancy tracking, capacity rules cannot be enforced.

---

# 7️⃣ Capacity Management

Many simulations include limits such as:

```text
Maximum drones per zone
```

or

```text
Maximum drones per connection
```

Before a movement happens, those limits may need to be checked.

---

# 8️⃣ Waiting States

Sometimes a drone cannot move immediately.

Possible reasons:

- destination unavailable
- connection unavailable
- special movement rules
- temporary congestion

In these situations the Simulator may decide:

```text
wait this turn
```

Waiting is a normal part of scheduling.

---

# 9️⃣ Movement Validation

Before moving a drone, several questions may need to be answered.

✅ Is the destination valid?

✅ Is there enough capacity?

✅ Can the connection be used?

✅ Does a valid route still exist?

Only after validation can the movement be applied.

---

# 🔟 Path Updates

Routes are not always permanent.

As the simulation changes:

- occupancy changes
- capacities change
- available routes may change

Because of this, a route may need to be recalculated during execution.

---

# 1️⃣1️⃣ End Conditions

The simulation ends when:

```text
all drones have been delivered
```

Example:

```text
D1 ✓
D2 ✓
D3 ✓
D4 ✓
```

No active drones remain.

---

# 1️⃣2️⃣ Simulator vs PathFinder

These systems solve different problems.

## 🧭 PathFinder

Answers:

```text
Where should the drone go?
```

---

## 🎮 Simulator

Answers:

```text
Can the drone move right now?
```

---

Separating these responsibilities makes the project easier to understand, test and maintain.

---

# 1️⃣3️⃣ Mental Model

Think of the PathFinder as:

```text
GPS navigation
```

It suggests routes.

---

Think of the Simulator as:

```text
Air traffic control
```

It decides:

```text
Who can move
When they can move
Whether the movement is valid
```

