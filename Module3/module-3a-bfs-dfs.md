# Module 3a — Graph Traversal: BFS and DFS

> **The question this module is always asking:**
> Given a graph, how do we systematically visit every reachable node — and does the *order* in which we visit them matter?

---

## From Maps to Mazes

In Module 1, we grouped data (DSU), looked things up (hash maps), and scanned slices (sliding window). All of those operated on flat structures.

Now the structure changes. Data isn't a sequence anymore — it's a **network**. Nodes have arbitrary connections. You can't just scan left to right; you have to navigate.

A **graph** $G = (V, E)$ is:
- $V$: a set of vertices (nodes)
- $E \subseteq V \times V$: a set of edges (connections between nodes)

Traversal means: starting from a source node $s$, visit every node reachable from $s$ exactly once.

Two algorithms. Same goal. Different tools. Radically different behavior.

---

## The Single Difference That Changes Everything

Both BFS and DFS use the same skeleton:

```
initialize a frontier data structure with the source node
mark source as visited

while frontier is not empty:
    take a node from the frontier
    process it
    for each unvisited neighbor:
        mark as visited
        add to frontier
```

The only difference is **what data structure the frontier is**:

| Algorithm | Frontier | Order |
|---|---|---|
| **BFS** | Queue (FIFO) | First in, first out |
| **DFS** | Stack (LIFO) | Last in, first out |

One line of code difference. Completely different traversal order. This is worth sitting with.

---

## BFS: Ripples in a Pond

A Queue is first-in, first-out. When you add neighbors of a node, they go to the *back* of the line. The nodes already waiting are processed first.

**The effect:** BFS processes nodes in order of their distance from the source. All nodes at distance 1 are visited before any node at distance 2. All nodes at distance 2 before any at distance 3. The frontier expands outward like a ripple.

**Student:** Why does FIFO guarantee distance ordering?

**Teacher:** Because all nodes at distance $d$ are added to the queue before any node at distance $d+1$ is processed. The nodes at distance $d$ were added when we processed nodes at distance $d-1$. By the time we finish processing all distance-$d$ nodes, the queue contains exactly the distance-$(d+1)$ nodes, in some order. FIFO preserves this layering.

**Critical property:** The first time BFS reaches a node, it has found the shortest path to that node (in an unweighted graph). No shorter path exists — if one did, BFS would have visited this node in an earlier layer.

---

## DFS: Down the Rabbit Hole

A Stack is last-in, first-out. When you add neighbors of a node, they go to the *top* of the stack and are processed immediately — before any previously-waiting nodes.

**The effect:** DFS commits to a path and follows it as deep as possible before backtracking. It exhausts one branch entirely before trying another.

**Student:** When would you prefer DFS over BFS?

**Teacher:** Several cases:

- **Existence queries:** "Does a path exist between $A$ and $B$?" You don't need the shortest path, just any path. DFS finds it quickly without the overhead of maintaining distance layers.
- **Cycle detection:** DFS naturally reveals cycles because it follows a single path and can detect when it revisits a node on the current path.
- **Topological sort:** Depends on DFS finishing times (post-order). BFS can't provide this.
- **Tree problems:** Most binary tree traversals (pre-order, in-order, post-order) are DFS patterns.
- **Memory:** On a graph with high branching factor, BFS's queue can grow to hold an entire layer of nodes. DFS's stack only holds the current path, which is bounded by the depth.

---

## Comparing Traversal Behavior

Consider this graph:

```
        1
       / \
      2   3
     / \   \
    4   5   6
```

**BFS order (queue):** 1, 2, 3, 4, 5, 6  
Process level by level: root → level 1 → level 2.

**DFS order (stack):** 1, 2, 4, 5, 3, 6  
Dive into the leftmost branch completely before touching node 3.

Same graph. Same goal. Completely different visit order. The order is determined entirely by the data structure, not the graph.

---

## Shortest Path: Why BFS Wins

**Problem:** Find the shortest path from node $s$ to node $t$ in an unweighted graph.

BFS terminates the moment it first reaches $t$. At that moment, the number of layers traversed equals the shortest path length. This is guaranteed because BFS visits nodes in non-decreasing order of distance.

DFS offers no such guarantee. It might find $t$ via a long winding path before ever trying the short direct route. To guarantee shortest path, DFS would have to explore *every* possible path and compare lengths — exponential work.

**Formal statement:** In an unweighted graph, BFS from source $s$ computes $d(s, v)$ (the minimum number of edges from $s$ to $v$) for all reachable $v$.

---

## When Edges Have Weights: Dijkstra's Algorithm

BFS assumes every edge costs exactly 1. If edges have different weights (a highway vs. a dirt road), BFS breaks — it no longer processes nodes in order of true distance.

The fix: replace the Queue with a **Priority Queue** (min-heap). Instead of "process the node added earliest," we "process the node with the lowest accumulated cost so far."

This is **Dijkstra's Algorithm**:

```
dist = {s: 0, all others: ∞}
priority queue = [(0, s)]

while queue not empty:
    (cost, node) = pop minimum from queue
    for each neighbor v of node with edge weight w:
        if dist[node] + w < dist[v]:
            dist[v] = dist[node] + w
            push (dist[v], v) to queue
```

**Why it works:** The priority queue ensures we always process the globally cheapest node next. When a node is popped, its `dist` value is finalized — no cheaper path can exist, because any alternative path through an unprocessed node would cost at least as much (since all edge weights are non-negative).

**The triangle inequality:** Dijkstra's correctness relies on edge weights being non-negative. If weights can be negative, the triangle inequality $d(x,z) \leq d(x,y) + d(y,z)$ can be violated — a longer path might actually be cheaper. Dijkstra's greedy assumption breaks. You need **Bellman-Ford**, which handles negative weights by relaxing edges $|V|-1$ times instead of using a greedy priority queue.

---

## Implementation Notes

### BFS with a Queue

```
from collections import deque

def bfs(graph, source):
    visited = {source}
    queue = deque([source])

    while queue:
        node = queue.popleft()          # FIFO: take from front
        process(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)  # add to back
```

### DFS with a Stack (Iterative)

```
def dfs(graph, source):
    visited = set()
    stack = [source]

    while stack:
        node = stack.pop()              # LIFO: take from top
        if node in visited: continue
        visited.add(node)
        process(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)  # add to top
```

### DFS Recursive (Natural Form)

```
def dfs(graph, node, visited):
    visited.add(node)
    process(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
```

The call stack *is* the stack. Recursive DFS is the cleanest form for tree problems where the depth is bounded.

---

## Representing the Graph

Before running any traversal, the graph must be in memory. Two representations:

**Adjacency List** (preferred for most problems):

$$\text{graph}[u] = [\text{list of neighbors of } u]$$

Space: $O(V + E)$. Efficient for sparse graphs. Most algorithmic problems use this.

**Adjacency Matrix**:

$$M[u][v] = 1 \text{ if edge } (u,v) \text{ exists, else } 0$$

Space: $O(V^2)$. $O(1)$ edge lookup. Best for dense graphs or when you need to check specific edges frequently.

---

## On Paper First

Before writing traversal code:

1. **Identify what you need:** Shortest path? Any path? All reachable nodes? A specific ordering?
2. **Choose the algorithm:**
   - Unweighted shortest path → BFS
   - Existence / cycle detection / topological sort → DFS
   - Weighted shortest path (non-negative) → Dijkstra
   - Weighted shortest path (negative weights) → Bellman-Ford
3. **Choose the representation:** Adjacency list for most problems.
4. **Handle the visited set:** Always mark nodes visited *before* adding to the frontier (not when popping), or you'll add the same node multiple times.

---

## Summary

| | BFS | DFS |
|---|---|---|
| Frontier | Queue (FIFO) | Stack (LIFO) |
| Traversal order | Level by level | Branch by branch |
| Shortest path (unweighted) | **Yes — guaranteed** | No |
| Memory | $O(W)$ where $W$ = max width | $O(D)$ where $D$ = max depth |
| Natural for | Shortest path, level-order | Cycles, topological sort, tree traversals |

> **The structural insight:** BFS and DFS are not fundamentally different algorithms. They are the same algorithm with one substitution: swap the Queue for a Stack. The entire behavioral difference — ripple vs. dive — emerges from that single data structure choice.

---

*← Module 2 | Module 3b: Topological Sort →*
