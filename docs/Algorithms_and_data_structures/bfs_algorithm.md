# Breadth-First Search (BFS)
> Understanding graph traversal and shortest path search (used in the a-maze-ing project)

---

# Table of Contents

1. What is BFS?
2. Graph Traversal
3. Queue Concept
4. Why BFS uses deque
5. Layer-by-Layer Exploration
6. Visual Example
7. BFS Logic
8. Visited Nodes
9. Parent Tracking
10. Path Reconstruction
11. Complexity
12. BFS vs Dijkstra
13. Fly-in Connection
14. Mental Model

---

# 1️⃣ What is BFS?

BFS means:

# Breadth-First Search

It is an algorithm used to:

- traverse graphs
- explore neighbors
- find shortest paths
- avoid infinite loops

---

# BFS explores:

```text
layer by layer
```

NOT:

```text
deep first
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

BFS explores:

```text
start
↓
A B C
↓
D goal
```

It explores all neighbors before going deeper.

---

# 3️⃣ Queue Concept

BFS uses:

# FIFO Queue

FIFO means:

```text
First In
First Out
```

---

# Example

```text
ADD A
ADD B
ADD C
```

Queue:

```text
[A, B, C]
```

Remove:

```text
A
```

because:

```text
A entered first
```

---

# 4️⃣ Why BFS uses deque

Python provides:

```python
from collections import deque
```

`deque` means:

```text
double-ended queue
```

---

# Useful Operations

```python
append()
popleft()
```

---

# Example

```python
from collections import deque

queue = deque()

queue.append("A")
queue.append("B")
queue.append("C")

print(queue.popleft())
```

Output:

```text
A
```

---

# 5️⃣ Layer-by-Layer Exploration

BFS explores nodes in waves.

---

# Visual Example

```text
LEVEL 0:
start

LEVEL 1:
A B C

LEVEL 2:
D goal
```

BFS always explores:

```text
closest nodes first
```

---

# 6️⃣ Visual Simulation

Imagine water spreading:

```text
start
  ↓
A B C
  ↓
D goal
```

BFS spreads equally in all directions.

---

# 7️⃣ BFS Logic

# Core Idea

```text
Explore neighbors first.
```

---

# Pseudo Code

```text
CREATE queue
CREATE visited set
CREATE parent dictionary

ADD start zone to queue
MARK start as visited

WHILE queue not empty:

    current = queue.pop_left()

    IF current == goal:
        reconstruct path
        RETURN path

    FOR each connected zone:

        IF not visited:

            mark visited
            save parent
            add to queue
```

---

# 8️⃣ Visited Nodes

BFS MUST track visited nodes.

Why?

Because graphs may contain loops.

---

# Example

```text
A -> B
↑    ↓
D <- C
```

Without visited tracking:

```text
infinite loop
```

---

# Visited Set

Example:

```python
visited = {
    "start",
    "A",
    "B"
}
```

---

# 9️⃣ Parent Tracking

BFS usually stores:

```text
where each node came from
```

---

# Example

```python
{
    "A": "start",
    "goal": "C"
}
```

This allows path reconstruction later.

---

# 🔟 Path Reconstruction

Once BFS reaches the goal:

```text
go backwards using parents
```

---

# Example

```text
goal
 ↑
 C
 ↑
 start
```

Reverse result:

```text
start -> C -> goal
```

---

# 1️⃣1️⃣ Complexity

BFS complexity is usually:

```text
O(V + E)
```

Where:

```text
V = vertices (zones)
E = edges (connections)
```

---

# 1️⃣2️⃣ BFS vs Dijkstra

# BFS

Good for:

- unweighted graphs
- same movement costs
- shortest path in number of steps

---

# Dijkstra

Good for:

- weighted graphs
- movement costs
- cheapest total route

---

# Example

BFS thinks:

```text
fewest movements
```

Dijkstra thinks:

```text
lowest total cost
```

---

# 1️⃣3️⃣ Fly-in Connection

Fly-in uses:

```text
normal      = 1
priority    = 1
restricted  = 2
blocked     = impossible
```

This means:

```text
not all paths cost the same
```

So:

```text
BFS is useful to learn graph traversal
```

BUT:

```text
Dijkstra fits Fly-in better
```

---

# 1️⃣4️⃣ Mental Model

Think of BFS like:

```text
water spreading equally
```

or:

```text
exploring closest rooms first
```

---

# Final Mental Image

BFS always asks:

```text
"What is the closest unexplored node?"
```

NOT:

```text
"What is the cheapest node?"
```

That is the BIG difference between:

```text
BFS
vs
Dijkstra
```

