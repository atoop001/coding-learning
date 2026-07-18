# Project 6: Route Finder

## Description

Build a route-planning engine for a transit-style network: load a map of stations and connections from a file, then answer real questions — "can I get from A to B?", "fewest stops?", "fastest route with travel times?", "what's reachable within 20 minutes?", and "which single station's closure would split the network?". This is Chapter 11 turned into a working tool: BFS for fewest hops, Dijkstra (heap-powered) for weighted shortest paths, DFS for connectivity analysis — plus an ASCII grid mode where the same algorithms solve mazes, proving your code generalizes to implicit graphs.

## Difficulty

**Advanced.** Estimated effort: 10–14 hours.

## Chapters used

- Chapter 4 (queues drive BFS)
- Chapter 5 (adjacency dicts, visited sets, distance maps)
- Chapter 6 (recursion/backtracking for DFS)
- Chapter 10 (priority queue drives Dijkstra)
- Chapter 11 (graphs, BFS/DFS, Dijkstra, components)

## Requirements checklist

### Graph foundation
- [ ] A `Graph` class on an adjacency-list dict: `add_node`, `add_edge(u, v, weight=1, directed=False)`, `neighbors`, `nodes`, `has_edge` — plus `__repr__` summarizing node/edge counts
- [ ] Loader for a plain-text map format you define and document (e.g., lines of `StationA,StationB,minutes`), tolerant of blank lines/comments, with clear errors on malformed lines
- [ ] A sample network file you author: ≥ 20 stations, ≥ 30 connections, at least two disconnected components, and at least one pair of stations whose fewest-stops route differs from their fastest route (you'll need this for testing — engineer it deliberately)

### Queries
- [ ] `reachable(a, b)`: connectivity via DFS (iterative — must survive a 10,000-node chain; include a generated stress test)
- [ ] `fewest_stops(a, b)`: BFS returning the **actual route** (predecessor reconstruction), not just the count
- [ ] `fastest_route(a, b)`: Dijkstra with a heap, lazy deletion, returning route + total time; raises/reports clearly on unreachable pairs
- [ ] `reachable_within(a, minutes)`: all stations within a time budget (Dijkstra frontier cut)
- [ ] `components()`: list the network's islands
- [ ] `critical_stations()`: stations whose individual removal increases the component count (brute-force re-traversal per station is acceptable — state its complexity honestly)

### Maze mode (implicit graph)
- [ ] Parse an ASCII maze (`#` wall, `.` floor, `S` start, `E` exit) into *no* explicit graph — neighbors computed on the fly — and reuse your BFS to print the maze with the shortest path drawn onto it
- [ ] At least 3 maze test files: solvable, unsolvable, and one where greedy "head toward the exit" would fail but BFS succeeds

### Interface & proof
- [ ] CLI: `route <map> fastest A B`, `route <map> stops A B`, `route <map> within A 20`, `route maze <file>` (or an equivalent menu loop)
- [ ] Tests: known shortest paths on your sample network (hand-verify them once and hard-code expectations), unreachable pairs, the stops-vs-time divergent pair, single-node graph, self-route (A to A)
- [ ] `DESIGN.md`: complexity of every query in V and E, why BFS can't handle weights (with your divergent pair as the concrete example), and why Dijkstra needs non-negative weights

## Hints

- Path reconstruction: store `parent[child] = current` when you first discover a node; walk backward from the goal and reverse. One helper shared by BFS and Dijkstra.
- Dijkstra's heap entries: `(distance_so_far, node)` tuples; on pop, skip if you've already finalized a shorter distance for that node (the staleness check from Chapter 11). No decrease-key needed.
- For `reachable_within`, Dijkstra already visits nodes in increasing distance order — you can stop early the moment a popped distance exceeds the budget. Say so in a comment; early termination is the insight.
- Engineering the divergent pair: give a direct edge a big weight (express line closed for repairs: 1 stop, 30 min) and a multi-hop alternative small weights (4 stops, 12 min).
- Maze neighbors: from `(r, c)` yield the ≤ 4 in-bounds, non-wall cells. Keep coordinates as tuples — they're hashable, so your visited set and parent map work unchanged.
- `critical_stations`: count components, then for each station, count components of the graph *without it* (skip it in traversals rather than mutating the graph). Compare counts among nodes of the same original component.

## Stretch goals

- A* for mazes: Dijkstra plus a Manhattan-distance heuristic; count nodes expanded by BFS vs A* on a large maze and report the savings.
- Transfer penalties: edges tagged with a line name; add cost when a route changes lines. What has to change in your state definition? (Hint: "station" is no longer enough — this is a bigger lesson than it looks.)
- k-alternative routes: after the best route, find the next-best that differs by at least one edge.
- Random network generator (n stations, edge probability p) and a measurement of how average component count and route lengths change as p grows.
- Visualize a solved maze animation in the terminal: reprint the grid each BFS ring with `time.sleep`, showing the wavefront expand.
