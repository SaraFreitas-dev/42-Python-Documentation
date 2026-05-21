# Depth-First Search (DFS)
> Understanding recursive graph traversal and maze/path exploration (used in the a-maze-ing project).

---

# Table of Contents

1. What is DFS?
2. Graph Traversal
3. DFS Mentality
4. Stack Behavior
5. Recursive Exploration
6. Visual Example
7. DFS Logic
8. Visited Nodes
9. Recursive Backtracking
10. Iterative DFS
11. Complexity
12. DFS vs BFS
13. DFS in Maze Generation
14. Fly-in Connection
15. Mental Model

---

# 1️⃣ What is DFS?

DFS means:

# Depth-First Search

It is an algorithm used to:

- traverse graphs
- explore paths deeply
- generate mazes
- search recursive structures

---

# DFS explores:

```text
go as deep as possible first
```

ONLY after:

```text
backtrack
```

---

# 2️⃣ Graph Traversal

Imagine this graph:

```text
        start
       /  |  \
      A   B   C
     /         \
    D           goal
```

DFS might explore:

```text
start
↓
A
↓
D
↓
backtrack
↓
B
↓
C
↓
goal
```

---

# 3️⃣ DFS Mentality

DFS constantly thinks:

```text
"Keep going deeper."
```

It prioritizes:

```text
current branch exploration
```

before checking siblings.

---

# 4️⃣ Stack Behavior

DFS behaves like:

# Stack

A stack works like:

```text
Last In
First Out
```

This is called:

# LIFO

---

# Example

```text
ADD A
ADD B
ADD C
```

Stack:

```text
[A, B, C]
```

Remove:

```text
C
```

because:

```text
C was added last
```

---

# 5️⃣ Recursive Exploration

DFS is commonly implemented using:

```text
recursion
```

---

# Example

```python
visit(node)
    ↓
visit(neighbor)
    ↓
visit(neighbor)
```

---

# Recursion Flow

```text
start
 ↓
A
 ↓
D
 ↓
end branch
 ↓
backtrack
```

---

# 6️⃣ Visual Example

Graph:

```text
        start
       /     \
      A       B
     / \
    C   D
```

DFS exploration:

```text
start
↓
A
↓
C
↓
backtrack
↓
D
↓
backtrack
↓
B
```

---

# 7️⃣ DFS Logic

# Core Idea

```text
Explore deeply before exploring widely.
```

---

# Recursive Pseudo Code

```text
FUNCTION dfs(node):

    mark node as visited

    FOR each neighbor:

        IF neighbor not visited:

            dfs(neighbor)
```

---

# 8️⃣ Visited Nodes

DFS MUST track visited nodes.

Otherwise:

```text
infinite recursion
```

may happen.

---

# Example Loop

```text
A -> B
↑    ↓
D <- C
```

Without visited tracking:

```text
A -> B -> C -> D -> A -> ...
```

forever 😄

---

# Visited Example

```python
visited = {
    "start",
    "A",
    "B"
}
```

---

# 9️⃣ Recursive Backtracking

When DFS reaches:

```text
end of branch
```

it:

```text
returns backwards
```

This is called:

# backtracking

---

# Visual Example

```text
start
 ↓
A
 ↓
C
 ↓
no neighbors
 ↓
backtrack to A
 ↓
explore D
```

---

# 🔟 Iterative DFS

DFS can also be implemented without recursion.

Using:

```text
stack
```

---

# Iterative Pseudo Code

```text
CREATE stack
CREATE visited set

PUSH start node

WHILE stack not empty:

    current = stack.pop()

    IF not visited:

        mark visited

        PUSH neighbors
```

---

# 1️⃣1️⃣ Complexity

DFS complexity is usually:

```text
O(V + E)
```

Where:

```text
V = vertices
E = edges
```

---

# 1️⃣2️⃣ DFS vs BFS

# DFS

Explores:

```text
deep first
```

---

# BFS

Explores:

```text
layer by layer
```

---

# DFS Mentality

```text
"Keep going deeper"
```

---

# BFS Mentality

```text
"Explore closest nodes first"
```

---

# 1️⃣3️⃣ DFS in Maze Generation

DFS is EXTREMELY popular for:

# maze generation

Because:

```text
DFS naturally creates long corridors
```

---

# Example

Your A-Maze-ing project uses:

```text
Randomized DFS
```

to:

- carve paths
- explore cells
- backtrack
- generate perfect mazes

---

# DFS Maze Flow

```text
current cell
    ↓
random neighbor
    ↓
carve wall
    ↓
continue deeper
    ↓
no neighbors?
    ↓
backtrack
```

---

# 1️⃣4️⃣ Fly-in Connection

DFS is useful in Fly-in for:

- graph traversal learning
- recursive exploration
- understanding pathfinding
- understanding backtracking

BUT:

```text
DFS does NOT guarantee cheapest path
```

or:

```text
shortest weighted route
```

That is why:

```text
Dijkstra fits Fly-in better
```

---

# 1️⃣5️⃣ Mental Model

Think of DFS like:

```text
exploring a cave
```

You:

- keep moving deeper
- follow one tunnel
- only return when stuck

---

# Final Mental Image

DFS always asks:

```text
"How far can I go from here?"
```

NOT:

```text
"What is the closest node?"
```

That is the BIG difference between:

```text
DFS
vs
BFS
```

