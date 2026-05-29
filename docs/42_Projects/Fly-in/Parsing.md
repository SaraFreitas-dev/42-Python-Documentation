# 📄 Parsing — Input File Introduction

# 🛩️ Fly-in Input System

The project receives a `.txt` file describing:

- the drone count
- all available zones
- zone metadata
- all graph connections
- movement constraints

The parser is responsible for:

- reading the file
- validating syntax
- validating rules
- converting raw text into internal objects

---

# 📦 Example Input File

```txt
# Drone count
nb_drones: 5

# Start and end zones
start_hub: hub 0 0 [color=green]
end_hub: goal 10 10 [color=yellow]

# Regular zones
hub: roof1 3 4 [zone=restricted color=red]
hub: roof2 6 2 [zone=normal color=blue]
hub: corridorA 4 3 [zone=priority color=green max_drones=2]
hub: tunnelB 7 4 [zone=normal color=red]
hub: obstacleX 5 5 [zone=blocked color=gray]

# Connections
connection: hub-roof1
connection: hub-corridorA
connection: roof1-roof2
connection: roof2-goal
connection: corridorA-tunnelB [max_link_capacity=2]
connection: tunnelB-goal
```

---

# 🧾 File Structure

The parser reads the file line by line.

Each line may represent:

- drone count
- zone declaration
- connection declaration
- comments
- empty lines

---

# 💬 Comments

Lines starting with `#` are comments.

- comments are ignored by the parser
- comments may appear anywhere
- empty lines should also be ignored

Example:

```txt
# This is a comment
```

---

# 🔢 Drone Count

The first mandatory entry defines:

```txt
nb_drones: <number>
```

Example:

```txt
nb_drones: 5
```

## Rules

- must exist
- must be a positive integer
- only one definition is allowed

---

# 📍 Zone Definitions

| Type | Example | Rules |
|---|---|---|
| 🟢 Start Zone | `start_hub: hub 0 0` | - Exactly one start zone must exist<br>- Coordinates are mandatory<br>- Coordinates are always integers |
| 🔴 End Zone | `end_hub: goal 10 10` | - Exactly one end zone must exist<br>- Coordinates are mandatory<br>- Coordinates are always integers |
| 🔵 Regular Zone | `hub: roof1 3 4` | - Zone names must be unique<br>- Coordinates are mandatory<br>- Coordinates are integers |

---

# 🚫 Invalid Zone Names

| Invalid | Valid |
|---|---|
| `my-zone` | `my_zone` |
| `my zone` | `roof1` |

## Rules

- zone names cannot contain spaces
- zone names cannot contain dashes (`-`)

---

# 📦 Optional Metadata

Zones may include optional metadata inside:

```txt
[ ... ]
```

Example:

```txt
hub: roof1 3 4 [zone=restricted color=red]
```

Metadata order does not matter.

---

# 🏷️ Zone Metadata

| Metadata | Example | Rules |
|---|---|---|
| `zone=<type>` | `zone=restricted` | Allowed values: `normal`, `restricted`, `priority`, `blocked` |
| `color=<value>` | `color=red` | Optional single-word string |
| `max_drones=<number>` | `max_drones=2` | Must be a positive integer |

---

# 🔗 Connection Definitions

Connections define graph edges.

## Syntax

```txt
connection: zoneA-zoneB
```

Example:

```txt
connection: roof1-roof2
```

---

# ↔️ Connection Rules

Connections are:

- bidirectional

Meaning:

```txt
A-B
```

allows:

- A → B
- B → A

---

# 🚫 Invalid Connections

Connections are invalid if:

- zones do not exist
- duplicate edges exist
- syntax is malformed

Example duplicates:

```txt
A-B
B-A
```

---

# 📦 Connection Metadata

Connections may also contain metadata:

```txt
[max_link_capacity=N]
```

Example:

```txt
connection: A-B [max_link_capacity=2]
```

## Rules

- optional
- positive integer only

---

# ⚠️ Parsing Validation

The parser should validate:

- duplicated zones
- duplicated connections
- invalid metadata
- invalid zone types
- invalid capacities
- malformed syntax
- missing start/end zones
- invalid coordinates

---

# 🚨 Error Handling

Any parsing error should:

- stop parsing immediately
- return a clear error message

Ideally including:

- line number
- reason of failure

Example:

```txt
Line 12:
Invalid zone type: lava
```

---

# 🧠 Parsing Goal

The parsing system should transform the raw `.txt` file into:

- graph structures
- zone objects
- connection objects
- metadata structures

that will later be used by:

- pathfinding
- simulation
- scheduling
- visualization systems