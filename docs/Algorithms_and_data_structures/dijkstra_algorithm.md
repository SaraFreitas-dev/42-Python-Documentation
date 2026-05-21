# Dijkstra Algorithm
> Understanding weighted graph pathfinding and cheapest-route calculation (used on the fly-in project).

---

# Table of Contents

1. What is Dijkstra?
2. Weighted Graphs
3. Cheapest Path vs Shortest Path
4. Priority Queue (`heapq`)
5. Cost Tracking
6. Parent Tracking
7. Visited Nodes
8. Dijkstra Logic
9. Path Reconstruction
10. Visual Example
11. Fly-in Connection
12. BFS vs Dijkstra
13. Complexity
14. Mental Model

---

# 1️⃣ What is Dijkstra?

Dijkstra is a graph algorithm used to:

- find cheapest paths
- calculate weighted routes
- optimize movement cost
- solve weighted graph problems

---

# Dijkstra focuses on:

```text
lowest total cost
```

NOT:

```text
fewest movements
```

---

# 2️⃣ Weighted Graphs

A weighted graph means:

```text
connections or zones have different costs
```

---

# Example

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

---

# 3️⃣ Cheapest Path vs Shortest Path

# BFS thinks:

```text
fewest steps
```

---

# Dijkstra thinks:

```text
lowest total cost
```

---

# Example

## Path A

```text
start -> restricted -> goal
```

Cost:

```text
3 turns
```

---

## Path B

```text
start -> normal -> normal -> goal
```

Cost:

```text
3 turns
```

---

## Path C

```text
start -> priority -> goal
```

Cost:

```text
2 turns
```

Dijkstra chooses:

```text
Path C
```

because:

```text
lowest total cost
```

---

# 4️⃣ Priority Queue (`heapq`)

Dijkstra usually uses:

```python
import heapq
```

because it constantly needs:

```text
cheapest current node
```

---

# Example

```python
queue = []

heapq.heappush(queue, (1, "A"))
heapq.heappush(queue, (5, "B"))
heapq.heappush(queue, (2, "C"))
```

---

# heapq automatically organizes:

```text
lowest cost first
```

---

# Then:

```python
heapq.heappop(queue)
```

returns:

```python
(1, "A")
```

---

# 5️⃣ Cost Tracking

Dijkstra stores:

```text
best known cost for every node
```

---

# Example

```python
costs = {
    "start": 0,
    "A": 1,
    "B": 4,
    "goal": 2
}
```

---

# Core Idea

If a cheaper route is discovered:

```text
update cost
```

---

# 6️⃣ Parent Tracking

Dijkstra also stores:

```text
where each node came from
```

This allows:

# path reconstruction

---

# Example

```python
parents = {
    "A": "start",
    "goal": "A"
}
```

---

# 7️⃣ Visited Nodes

Dijkstra tracks visited nodes to avoid:

- unnecessary processing
- loops
- recalculating explored routes

---

# Example

```python
visited = {
    "start",
    "A"
}
```

---

# 8️⃣ Dijkstra Logic

Dijkstra works by constantly exploring:

```text
The currently cheapest known route
```

The algorithm starts at the initial node and gradually expands through the graph.

Every time it discovers a cheaper route to a node:

- the cost is updated
- the route information is updated
- the node becomes a better candidate for exploration

---

# Core Concepts

## Cheapest-first exploration

Unlike BFS, Dijkstra does not explore based on:

```text
arrival order
```

Instead, it explores based on:

```text
lowest total cost
```

---

## Continuous optimization

The algorithm constantly compares:

```text
old path cost
vs
new possible path cost
```

If the new route is cheaper:

```text
replace old cost
```

---

## Progressive expansion

Dijkstra gradually expands through the graph:

```text
start
 ↓
cheap neighbors
 ↓
slightly more expensive neighbors
 ↓
more expensive neighbors
```

The graph is explored in order of increasing total cost.

---

## Weighted movement

This is why Dijkstra is extremely useful for:

- traffic systems
- GPS navigation
- weighted movement games
- drone routing
- path optimization

---

# 9️⃣ Path Reconstruction

Once the goal is reached:

```text
walk backwards using parents
```

---

# Example

```text
goal
 ↑
A
 ↑
start
```

Reverse result:

```text
start -> A -> goal
```

---

# 🔟 Visual Example

Graph:

```text
        start
       /     \
     (1)     (5)
     A         B
      \
      (1)
        \
        goal
```

---

# Exploration

Dijkstra sees:

```text
A = cost 1
B = cost 5
```

It explores:

```text
A first
```

because:

```text
1 < 5
```

---

# Final Path

```text
start -> A -> goal
```

Total cost:

```text
2
```

---

# 1️⃣1️⃣ Fly-in Connection

Fly-in is essentially:

# weighted graph pathfinding

because:

- movement costs exist
- restricted zones cost more
- blocked zones are invalid
- drones need optimized routes

---

# Dijkstra fits Fly-in because it:

- minimizes total movement cost
- avoids blocked zones
- supports weighted routing
- generates optimal paths

---

# 1️⃣2️⃣ BFS vs Dijkstra

# BFS

Explores:

```text
fewest steps
```

Uses:

```python
deque
```

---

# Dijkstra

Explores:

```text
lowest cost path
```

Uses:

```python
heapq
```

---

# BFS

Good for:

- unweighted graphs
- same movement costs

---

# Dijkstra

Good for:

- weighted graphs
- route optimization
- movement costs

---

# 1️⃣3️⃣ Complexity

Typical Dijkstra complexity:

```text
O((V + E) log V)
```

Where:

```text
V = vertices
E = edges
```

---

# Why more expensive than BFS?

Because:

```text
heapq sorting adds extra work
```

BUT:

```text
weighted pathfinding requires it
```

---

# 1️⃣4️⃣ Mental Model

Think of Dijkstra like:

```text
GPS route optimization
```

The algorithm constantly asks:

```text
"What is currently the cheapest route?"
```

NOT:

```text
"What was explored first?"
```

---

# Final Mental Image

# BFS

```text
Closest room first
```

---

# Dijkstra

```text
Cheapest road first
```


