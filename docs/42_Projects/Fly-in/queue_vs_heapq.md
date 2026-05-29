# Queue vs Heap Queue (`heapq`)
> Understanding why BFS uses queues and Dijkstra uses priority queues.

---

# What is a Queue?

A queue works like:

```text
First In
First Out
```

This is called:

# FIFO

---

# Example

```text
[A, B, C]
```

Removing:

```text
A
```

because:

```text
A entered first
```

---

# deque

Python provides:

```python
from collections import deque
```

`deque` means:

```text
double-ended queue
```

Useful operations:

```python
append()
popleft()
```

---

# Why BFS uses deque

BFS explores:

```text
level by level
```

Example:

```text
start
 ├── A
 ├── B
 └── C
```

BFS processes:

```text
A first
then B
then C
```

because:

```text
they were inserted first
```

---

# BFS Behavior

```text
ORDER OF ARRIVAL
```

This is PERFECT for:

```python
deque
```

---

# What is heapq?

`heapq` is Python's built-in:

# Priority Queue

---

# Difference from normal queue

A normal queue removes:

```text
oldest element
```

A priority queue removes:

```text
lowest cost first
```

---

# Example

You insert:

```python
(5, "B")
(1, "A")
(3, "C")
```

---

# heapq automatically organizes:

```text
lowest cost first
```

Then:

```python
heapq.heappop(queue)
```

returns:

```python
(1, "A")
```

EVEN IF:

```text
A was inserted later
```

---

# Why Dijkstra uses heapq

Dijkstra constantly asks:

```text
"What is the CHEAPEST current route?"
```

NOT:

```text
"What was inserted first?"
```

That is why:

```python
deque
```

is not ideal for Dijkstra.

---

# Dijkstra needs:

```text
LOWEST COST FIRST
```

Which is exactly what:

```python
heapq
```

does.

---

# Dijkstra Example

```python
import heapq

queue = []

heapq.heappush(queue, (2, "roof1"))
heapq.heappush(queue, (1, "corridorA"))
heapq.heappush(queue, (5, "goal"))

print(heapq.heappop(queue))
```

Output:

```python
(1, "corridorA")
```

---

# Visual Comparison

# deque / BFS

```text
ORDER OF ARRIVAL
```

```text
A → B → C
```

---

# heapq / Dijkstra

```text
LOWEST COST FIRST
```

```text
1 → 2 → 5
```

---

# Fly-in Connection

# BFS

Good for:
- unweighted graphs
- same movement costs

---

# Dijkstra

Good for:
- weighted graphs
- movement costs
- route optimization

Fly-in is:

```text
Weighted Graph Pathfinding
```

So:

# heapq + Dijkstra
fits the project much better.

---

# Mental Model

# BFS / deque

Think:

```text
"Who arrived first?"
```

---

# Dijkstra / heapq

Think:

```text
"Who is currently the cheapest?"
```

---

# Final Mental Image

# deque

```text
Supermarket line
```

First customer:
- enters first
- leaves first

---

# heapq

```text
Hospital emergency room
```

Most urgent patient:
- treated first
- even if arrived later

