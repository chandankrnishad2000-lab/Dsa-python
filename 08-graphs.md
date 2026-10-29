# 08 — Graphs

Four core algorithms cover most graph interviews: BFS, DFS, topological sort,
Union-Find, plus Dijkstra for weighted shortest paths.

## 0. Representation

```python
from collections import defaultdict, deque

adj = defaultdict(list)
for u, v in edges:
    adj[u].append(v)
    adj[v].append(u)        # omit for directed graphs
```

For grid problems, neighbors are `(r±1, c), (r, c±1)` inside bounds.

```python
DIRS = [(1, 0), (-1, 0), (0, 1), (0, -1)]
```

## 1. BFS — shortest path in unweighted graphs

```python
# T: BFS with distance
def bfs(adj, start):
    dist = {start: 0}
    q = deque([start])
    while q:
        u = q.popleft()
        for v in adj[u]:
            if v not in dist:
                dist[v] = dist[u] + 1
                q.append(v)
    return dist
```

Multi-source BFS (rotting oranges, walls-and-gates): enqueue **all** sources at
distance 0 first, then expand.

```python
def rotting_oranges(grid):
    R, C = len(grid), len(grid[0])
    q = deque()
    fresh = 0
    for r in range(R):
        for c in range(C):
            if grid[r][c] == 2:
                q.append((r, c, 0))
            elif grid[r][c] == 1:
                fresh += 1
    time = 0
    while q:
        r, c, t = q.popleft()
        time = max(time, t)
        for dr, dc in DIRS:
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and grid[nr][nc] == 1:
                grid[nr][nc] = 2
                fresh -= 1
                q.append((nr, nc, t + 1))
    return time if fresh == 0 else -1
```

## 2. DFS — components, reachability, cycle detection

```python
# T: iterative DFS
def dfs(adj, start, visited):
    stack = [start]
    visited.add(start)
    while stack:
        u = stack.pop()
        for v in adj[u]:
            if v not in visited:
                visited.add(v)
                stack.append(v)
```

### Number of islands (grid DFS/BFS)
```python
def num_islands(grid):
    if not grid:
        return 0
    R, C = len(grid), len(grid[0])
    def sink(r, c):
        if not (0 <= r < R and 0 <= c < C) or grid[r][c] != "1":
            return
        grid[r][c] = "0"
        for dr, dc in DIRS:
            sink(r + dr, c + dc)
    count = 0
    for r in range(R):
        for c in range(C):
            if grid[r][c] == "1":
                count += 1
                sink(r, c)
    return count
```

### Cycle detection in a directed graph (colors)
`0 = unvisited, 1 = in progress, 2 = done`. A back-edge to a `1` means a cycle.

```python
def has_cycle_directed(adj, n):
    color = [0] * n
    def dfs(u):
        color[u] = 1
        for v in adj[u]:
            if color[v] == 1:
                return True
            if color[v] == 0 and dfs(v):
                return True
        color[u] = 2
        return False
    return any(color[u] == 0 and dfs(u) for u in range(n))
```

## 3. Topological sort

Only for **directed acyclic graphs**. Two ways; know both.

### Kahn's algorithm (BFS, returns order + detects cycle)
```python
# T: topological sort via indegree
def topo_sort(n, edges):
    adj = defaultdict(list)
    indeg = [0] * n
    for u, v in edges:            # u -> v
        adj[u].append(v)
        indeg[v] += 1
    q = deque(i for i in range(n) if indeg[i] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in adj[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    return order if len(order) == n else []      # [] means cycle
```

### DFS postorder reversal
Push nodes after exploring all descendants, then reverse.

Use for: course scheduling, build order, alien dictionary, task dependencies,
"longest path in a DAG" (topo + DP).

## 4. Union-Find (Disjoint Set Union)

Best for dynamic connectivity, Kruskal, and "are these connected?" queries.

```python
# T: DSU with path compression + union by size
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        self.components = n

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]   # path halving
            x = self.parent[x]
        return x

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        if self.size[ra] < self.size[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        self.components -= 1
        return True

    def connected(self, a, b):
        return self.find(a) == self.find(b)
```
Time per op: O(α(n)) ≈ O(1).

### Kruskal's MST
Sort edges by weight, union if it does not create a cycle.
```python
def kruskal(n, edges):           # edges: (w, u, v)
    dsu = DSU(n)
    total = 0
    for w, u, v in sorted(edges):
        if dsu.union(u, v):
            total += w
    return total if dsu.components == 1 else -1
```

## 5. Dijkstra — single-source shortest path, non-negative weights

```python
# T: Dijkstra with heapq
def dijkstra(n, adj, src):
    INF = float("inf")
    dist = [INF] * n
    dist[src] = 0
    pq = [(0, src)]                    # (distance, node)
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:
            continue                   # stale entry
        for v, w in adj[u]:
            nd = d + w
            if nd < dist[v]:
                dist[v] = nd
                heapq.heappush(pq, (nd, v))
    return dist
```
Time O((V + E) log V). For dense graphs use O(V²) array version.
Use **Bellman-Ford** when negative edges exist; **Floyd-Warshall** for all pairs (O(V³)).

### 0-1 BFS (weights are 0 or 1)
Use a deque: push 0-weight edges to the front, 1-weight to the back. O(V + E).

### Dijkstra with state (wall-following / elevator problems)
The node is `(cell, state)` — e.g., `(r, c, remaining_k)`. Run Dijkstra over the
expanded state graph.

## 6. Grid graph tricks

- Mark visited by mutating the grid (e.g., `grid[r][c] = '0'`).
- BFS from the border for "surrounded regions", "pacific atlantic".
- For "shortest path through obstacles" with k removals, state = `(r, c, used)`.
- For "word ladder", BFS over strings where neighbors are one-letter mutations.

## Decision guide

| Question | Algorithm |
|----------|-----------|
| shortest path, unweighted | BFS |
| shortest path, weights 0/1 | 0-1 BFS |
| shortest path, non-negative weights | Dijkstra |
| negative weights, single source | Bellman-Ford |
| all-pairs shortest path | Floyd-Warshall |
| connected components | DFS / BFS / DSU |
| dynamic connectivity, MST | DSU / Kruskal |
| ordering with dependencies | topological sort |
| cycle in directed graph | colors DFS / Kahn |
| cycle in undirected graph | DFS parent check / DSU |

## Practice set

| Problem | Algorithm | Difficulty |
|---------|-----------|------------|
| Number of Islands | DFS/BFS | Medium |
| Max Area of Island | DFS | Medium |
| Clone Graph | DFS + map | Medium |
| Rotting Oranges | multi-source BFS | Medium |
| 01 Matrix | multi-source BFS | Medium |
| Walls and Gates | multi-source BFS | Medium |
| Pacific Atlantic Water Flow | BFS from borders | Medium |
| Course Schedule I / II | topological sort | Medium |
| Alien Dictionary | topological sort | Hard |
| Number of Connected Components | DFS/DSU | Medium |
| Redundant Connection | DSU | Medium |
| Accounts Merge | DSU | Medium |
| Graph Valid Tree | DSU / DFS | Medium |
| Network Delay Time | Dijkstra | Medium |
| Path with Minimum Effort | Dijkstra / binary search | Medium |
| Cheapest Flights Within K Stops | Bellman-Ford / Dijkstra | Medium |
| Word Ladder | BFS | Hard |
| Shortest Path in Grid with Obstacles Elimination | BFS + state | Hard |
| Swim in Rising Water | Dijkstra / binary search | Hard |
