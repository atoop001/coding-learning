# Chapter 11: Graphs & Traversal (BFS, DFS, and Representations)

## Overview

A graph is the most general data structure in this track: things (**vertices/nodes**) and connections between them (**edges**), with no rules about shape. Social networks, road maps, the web, package dependencies, airline routes, game states — all graphs. Trees (Chapter 9) are just graphs with no cycles and one root; linked lists are graphs in single file. Two algorithms unlock nearly everything: **breadth-first search** (BFS, powered by a queue) and **depth-first search** (DFS, powered by a stack or recursion). Between them they answer "can I get there?", "what's the shortest route?", "what's connected to what?", and "in what order must these tasks run?" — which is also most of the graph questions interviews ask.

## Definitions & Explanations

### Vocabulary

```
undirected:            directed (digraph):        weighted:
  A --- B                A --> B                    A --5-- B
  |     |                ^     |                    |       |
  C --- D                |     v                    2       1
                         C <-- D                    C --7-- D
```

- **Undirected** edge: friendship (mutual). **Directed** edge: Twitter follow (one-way).
- **Weighted** edge: carries a cost (miles, milliseconds, dollars).
- **Degree** of a vertex: number of edges touching it (in-degree/out-degree when directed).
- **Path**: sequence of vertices connected by edges. **Cycle**: path back to its start.
- **Connected component**: a maximal set of vertices all reachable from each other.
- **DAG** (directed acyclic graph): directed, no cycles — the shape of dependencies (build systems, prerequisites, spreadsheets).

Conventionally |V| = number of vertices, |E| = number of edges; complexities are stated in both.

### Representations

**Adjacency list** — map each vertex to its neighbors. The default choice:

```python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "D"],
    "D": ["B", "C"],
}
```

**Adjacency matrix** — n×n grid where `m[i][j] = 1` (or a weight) iff edge i→j exists:

```
      A  B  C  D
   A [0, 1, 1, 0]
   B [1, 0, 0, 1]
   C [1, 0, 0, 1]
   D [0, 1, 1, 0]
```

| | Adjacency list | Adjacency matrix |
|---|---|---|
| Space | O(V + E) | O(V²) |
| "Is there an edge u–v?" | O(degree(u)) | **O(1)** |
| Iterate u's neighbors | O(degree(u)) | O(V) |
| Best for | sparse graphs (most real ones) | dense graphs, tiny V, edge-lookup-heavy math |

Also common in interviews: an **implicit graph** — a 2D grid where cells are vertices and adjacent cells are edges, or word-ladder states. No structure is built; neighbors are *computed*.

### BFS — explore in rings

BFS visits everything 1 edge away, then 2, then 3... using a **queue** (FIFO, Chapter 4). Because it explores in distance order, **BFS finds shortest paths when every edge counts as 1**. Cost O(V + E).

```
BFS from A:        ring 0:  A
  A --- B --- E    ring 1:  B, C      (A's neighbors)
  |     |          ring 2:  D, E     (their unvisited neighbors)
  C --- D
```

### DFS — explore by tunneling

DFS follows one path as deep as possible, backtracks, tries the next branch — recursion (Chapter 6) or an explicit stack. Same O(V + E) cost, opposite personality. DFS is the natural tool for: does *any* path exist, cycle detection, connected components, topological sort, backtracking on implicit graphs (mazes).

### The non-negotiable: the visited set

Graphs, unlike trees, can have cycles. Both traversals MUST track visited vertices (a hash set — Chapter 5), or a cycle loops them forever and even cycle-free graphs get revisited exponentially.

### Shortest paths with weights: Dijkstra (preview-level)

When edges have weights, BFS's ring logic breaks (a 2-edge path can be cheaper than a 1-edge path). **Dijkstra's algorithm** repairs it: replace the queue with a **priority queue** (Chapter 10) keyed on total distance so far, and always expand the cheapest known frontier vertex. O((V + E) log V) with a binary heap; requires non-negative weights. It's BFS with a heap — if you know Chapters 4, 5, 10, you already own all its parts.

## Code Examples

```python
# graphs.py — BFS, DFS, shortest paths, components, topological sort.
from collections import deque
import heapq

def bfs_shortest_path(graph, start, goal):
    """Fewest-edges path in an UNWEIGHTED graph. O(V + E).
    Track each vertex's predecessor so the path can be rebuilt."""
    if start == goal:
        return [start]
    visited = {start}
    parent = {}
    q = deque([start])
    while q:
        u = q.popleft()
        for v in graph.get(u, []):
            if v in visited:
                continue
            visited.add(v)                  # mark when ENQUEUED (see pitfalls)
            parent[v] = u
            if v == goal:                   # first time we see goal = shortest
                path = [goal]
                while path[-1] != start:    # walk predecessors backward
                    path.append(parent[path[-1]])
                return path[::-1]
            q.append(v)
    return None                             # unreachable


def dfs_recursive(graph, start, visited=None):
    """Visit everything reachable from start, depth-first. O(V + E)."""
    if visited is None:
        visited = set()
    visited.add(start)
    for v in graph.get(start, []):
        if v not in visited:
            dfs_recursive(graph, v, visited)
    return visited


def dfs_iterative(graph, start):
    """Same reachability with an explicit stack — survives deep graphs."""
    visited = set()
    stack = [start]
    while stack:
        u = stack.pop()
        if u in visited:                    # may be pushed twice; skip dupes
            continue
        visited.add(u)
        for v in graph.get(u, []):
            if v not in visited:
                stack.append(v)
    return visited


def connected_components(graph):
    """Group vertices into islands: repeated DFS from unseen vertices."""
    seen, components = set(), []
    for vertex in graph:
        if vertex not in seen:
            comp = dfs_iterative(graph, vertex)
            seen |= comp
            components.append(sorted(comp))
    return components


def topological_sort(graph):
    """Order a DAG so every edge points forward (Kahn's algorithm):
    repeatedly emit a vertex with in-degree 0. Detects cycles for free."""
    indeg = {u: 0 for u in graph}
    for u in graph:
        for v in graph[u]:
            indeg[v] = indeg.get(v, 0) + 1
    q = deque(u for u, d in indeg.items() if d == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in graph.get(u, []):
            indeg[v] -= 1
            if indeg[v] == 0:               # last prerequisite satisfied
                q.append(v)
    if len(order) != len(indeg):
        raise ValueError("graph has a cycle — no valid ordering")
    return order


def dijkstra(graph, start):
    """Cheapest-total-weight distances from start. graph[u] = [(v, w), ...].
    BFS with the queue swapped for a min-heap of (distance, vertex)."""
    dist = {start: 0}
    heap = [(0, start)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist.get(u, float("inf")):   # stale entry (lazy deletion): skip
            continue
        for v, w in graph.get(u, []):
            nd = d + w
            if nd < dist.get(v, float("inf")):
                dist[v] = nd
                heapq.heappush(heap, (nd, v))   # may duplicate v — that's fine
    return dist


if __name__ == "__main__":
    g = {"A": ["B", "C"], "B": ["A", "D"], "C": ["A", "D"],
         "D": ["B", "C", "E"], "E": ["D"], "X": ["Y"], "Y": ["X"]}
    print(bfs_shortest_path(g, "A", "E"))       # ['A', 'B', 'D', 'E'] (or via C)
    print(connected_components(g))              # [[...A..E...], ['X','Y']]

    dag = {"wake": ["dress", "coffee"], "dress": ["leave"],
           "coffee": ["leave"], "leave": []}
    print(topological_sort(dag))                # wake before dress/coffee before leave

    wg = {"A": [("B", 5), ("C", 2)], "B": [("D", 1)],
          "C": [("B", 1), ("D", 7)], "D": []}
    print(dijkstra(wg, "A"))                    # {'A':0,'C':2,'B':3,'D':4}
```

The implicit-grid graph, interview staple:

```python
def count_islands(grid):
    """'Number of islands': components of 1-cells in a grid.
    The grid IS the graph; neighbors are computed, not stored."""
    if not grid:
        return 0
    rows, cols = len(grid), len(grid[0])
    seen = set()

    def flood(r, c):                        # DFS over the implicit graph
        stack = [(r, c)]
        while stack:
            r, c = stack.pop()
            if (r, c) in seen or not (0 <= r < rows and 0 <= c < cols):
                continue
            if grid[r][c] != 1:
                continue
            seen.add((r, c))
            stack.extend([(r+1, c), (r-1, c), (r, c+1), (r, c-1)])

    islands = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1 and (r, c) not in seen:
                islands += 1
                flood(r, c)
    return islands

print(count_islands([[1,1,0,0], [1,0,0,1], [0,0,1,1]]))   # 2
```

JavaScript: `Map`/`Set` play the roles of dict/set; arrays with `push`/`pop` give DFS. For BFS remember `shift()` is O(n) (Chapter 4) — use an index pointer over the array.

## Common Pitfalls

**1. No visited set (or checking too late).** The cardinal sin — infinite loops on any cycle. Subtler version in BFS: marking visited when *dequeued* instead of when *enqueued* lets the same vertex sit in the queue many times; correct answers, blown-up memory and time. Mark on enqueue (as `bfs_shortest_path` does).

**2. Using DFS for shortest paths.** DFS finds *a* path, cheerfully returning a 12-hop route when a 2-hop one exists. Unweighted shortest = BFS. Weighted shortest = Dijkstra. Reserve DFS for existence/structure questions.

**3. Forgetting both directions of an undirected edge.** Building `graph["A"].append("B")` without the mirror `graph["B"].append("A")` silently makes the graph directed; BFS from B can't reach A. Write an `add_edge(u, v)` helper that does both, and use `defaultdict(list)` so isolated mentions don't KeyError.

**4. Assuming the graph is connected.** Traversal from one start vertex covers only its component. Anything that must consider *all* vertices (counting components, cycle detection over the whole graph) needs the `for vertex in graph:` outer loop pattern from `connected_components`.

**5. Dijkstra with negative weights / without the staleness check.** Negative edges break Dijkstra's core assumption (use Bellman-Ford instead — know the name). And omitting the `if d > dist[u]: continue` line makes lazy deletion reprocess stale heap entries — still correct here, but wasteful, and a marker of not understanding the algorithm.

**6. Mutating the neighbor list while traversing it.** Removing edges inside `for v in graph[u]:` skips neighbors (the Chapter 2 iteration bug in a new costume). Iterate over a copy, or collect removals and apply after.

## Practice Exercises

1. Write `Graph` as a class wrapping `defaultdict(list)` with `add_edge(u, v, directed=False)`, `neighbors(u)`, and `vertices()`. Rebuild the chapter's functions as methods and re-run the demos.
2. Write `is_bipartite(graph)`: can vertices be 2-colored so no edge connects same colors? BFS coloring ring by ring; return a conflict edge as a witness when the answer is no. Handle disconnected graphs.
3. `word_ladder(start, end, word_list)`: shortest chain of one-letter changes (`cold → cord → card → ward`). Vertices are words; compute neighbors implicitly. Return the path, not just its length. What's the neighbor-generation cost per word, and total?
4. Detect a cycle in a **directed** graph using DFS with three states per vertex (unvisited / in-progress / done) — explain why the plain visited set that suffices for undirected graphs gives false positives here. Then use it to make `topological_sort` report *which* vertices form the cycle.
5. `maze_solver(grid, start, exit)`: shortest path in a grid with walls, returning the actual route as coordinates. Then add "portals" (pairs of cells that teleport, costing 1 move) — what changes in the neighbor function, and what doesn't change at all?

---

**Next:** Chapter 12 steps back from structures to *patterns* — two pointers, sliding windows, and frequency counting: the reusable moves that crack whole categories of problems.
