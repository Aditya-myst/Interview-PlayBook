# 15 — Shortest Path

## The Pattern That Finds the Shortest Route

---

### What It Is

Shortest Path algorithms find the minimum distance (or cost) between nodes in a graph. Different algorithms suit different graph types.

**The key insight:** Choose the algorithm based on the graph's properties (weighted/unweighted, positive/negative weights, sparse/dense).

---

### When to Use It

**The problem involves:**
- Finding the shortest distance between nodes
- Finding the minimum cost path
- Weighted graphs
- Grid-based pathfinding

**Trigger Words:**

| Trigger Word/Phrase | Algorithm |
|---------------------|-----------|
| "shortest path" + unweighted | BFS |
| "shortest path" + positive weights | Dijkstra |
| "shortest path" + negative weights | Bellman-Ford |
| "shortest path" + all pairs | Floyd-Warshall |
| "minimum cost" + positive weights | Dijkstra |
| "cheapest flight" | Dijkstra or Bellman-Ford |
| "network delay" | Dijkstra |

---

### The Algorithm Selection

```
Shortest Path Problem?
    ↓
Unweighted graph?
    ↓
Yes → BFS (O(V+E))

Weighted graph?
    ↓
Positive weights only?
    ↓
Yes → Dijkstra (O((V+E) log V))

Negative weights?
    ↓
Yes → Bellman-Ford (O(V×E))

All pairs needed?
    ↓
Yes → Floyd-Warshall (O(V³))
```

---

### Algorithm 1: BFS (Unweighted Graphs)

When all edges have the same weight (or no weight), BFS finds the shortest path.

```python
from collections import deque

def bfs_shortest_path(graph, start, end):
    queue = deque([(start, 0)])  # (node, distance)
    visited = {start}
    
    while queue:
        node, dist = queue.popleft()
        if node == end:
            return dist
        
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
    
    return -1  # No path
```

---

### Algorithm 2: Dijkstra (Positive Weights)

Dijkstra finds the shortest path from a source to all other nodes in a graph with non-negative edge weights.

**Key idea:** Always process the unvisited node with the smallest known distance.

```python
import heapq

def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    heap = [(0, start)]  # (distance, node)
    
    while heap:
        d, node = heapq.heappop(heap)
        
        if d > dist[node]:  # Already found a shorter path
            continue
        
        for neighbor, weight in graph[node]:
            new_dist = d + weight
            if new_dist < dist[neighbor]:
                dist[neighbor] = new_dist
                heapq.heappush(heap, (new_dist, neighbor))
    
    return dist
```

**Visual:**

```
Graph:
    A --1-- B
    |       |
    4       2
    |       |
    C --1-- D

Start: A

Initial: dist = {A:0, B:∞, C:∞, D:∞}
Heap: [(0,A)]

Pop (0,A): Process A
  B: 0+1=1 < ∞ → update, push (1,B)
  C: 0+4=4 < ∞ → update, push (4,C)
  Heap: [(1,B), (4,C)]

Pop (1,B): Process B
  A: 1+1=2 > 0, skip
  D: 1+2=3 < ∞ → update, push (3,D)
  Heap: [(3,D), (4,C)]

Pop (3,D): Process D
  B: 3+2=5 > 1, skip
  C: 3+1=4 = 4, no improvement
  Heap: [(4,C)]

Pop (4,C): Process C
  A: 4+4=8 > 0, skip
  D: 4+1=5 > 3, skip
  Heap: []

Result: {A:0, B:1, C:4, D:3}
```

---

### Algorithm 3: Bellman-Ford (Negative Weights)

Bellman-Ford handles negative edge weights and detects negative cycles.

**Key idea:** Relax all edges V-1 times. If you can still relax after V-1 iterations, there's a negative cycle.

```python
def bellman_ford(n, edges, start):
    dist = [float('inf')] * n
    dist[start] = 0
    
    # Relax V-1 times
    for _ in range(n - 1):
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    
    # Check for negative cycle
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            return None  # Negative cycle detected
    
    return dist
```

**Visual:**

```
Edges: [(A,B,1), (B,C,-2), (C,D,3), (A,C,5)]

Iteration 1:
  A→B: dist[B] = min(∞, 0+1) = 1
  B→C: dist[C] = min(∞, 1+(-2)) = -1
  C→D: dist[D] = min(∞, -1+3) = 2
  A→C: dist[C] = min(-1, 0+5) = -1

Iteration 2:
  A→B: no change
  B→C: dist[C] = min(-1, 1+(-2)) = -1
  C→D: dist[D] = min(2, -1+3) = 2
  A→C: no change

No negative cycle. Result: {A:0, B:1, C:-1, D:2}
```

---

### Dry Run: Network Delay Time (Dijkstra)

**Problem:** Given a network of n nodes and travel times, find how long it takes for all nodes to receive a signal from node k.

**Input:** `times = [[2,1,1],[2,3,1],[3,4,1]]`, `n = 4`, `k = 2`

**Recognition:** "network delay" + "travel times" → Dijkstra

```python
import heapq
from collections import defaultdict

def networkDelayTime(times, n, k):
    graph = defaultdict(list)
    for u, v, w in times:
        graph[u].append((v, w))
    
    dist = {}
    heap = [(0, k)]
    
    while heap:
        d, node = heapq.heappop(heap)
        if node in dist:
            continue
        dist[node] = d
        for neighbor, weight in graph[node]:
            if neighbor not in dist:
                heapq.heappush(heap, (d + weight, neighbor))
    
    return max(dist.values()) if len(dist) == n else -1
```

**Step-by-step:**

```
times = [[2,1,1],[2,3,1],[3,4,1]], n=4, k=2

Graph:
  2 → [(1,1), (3,1)]
  3 → [(4,1)]

Heap: [(0,2)]

Pop (0,2): dist[2]=0
  Push (1,1), (1,3)
  Heap: [(1,1), (1,3)]

Pop (1,1): dist[1]=1
  No neighbors
  Heap: [(1,3)]

Pop (1,3): dist[3]=1
  Push (2,4)
  Heap: [(2,4)]

Pop (2,4): dist[4]=2
  Heap: []

dist = {2:0, 1:1, 3:1, 4:2}
All 4 nodes reached. max = 2.

Return 2
```

---

### Algorithm 4: Floyd-Warshall (All Pairs)

Floyd-Warshall finds shortest paths between all pairs of nodes.

**Key idea:** For each intermediate node k, check if going through k gives a shorter path from i to j.

```python
def floyd_warshall(n, edges):
    # Initialize distance matrix
    dist = [[float('inf')] * n for _ in range(n)]
    for i in range(n):
        dist[i][i] = 0
    for u, v, w in edges:
        dist[u][v] = w
    
    # Main algorithm
    for k in range(n):
        for i in range(n):
            for j in range(n):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    
    return dist
```

**When to use:** When you need shortest paths between ALL pairs and n is small (≤ 400).

---

### Code Templates (4 Languages)

#### Python

```python
# BFS (Unweighted)
def bfs(graph, start):
    dist = {start: 0}
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in dist:
                dist[neighbor] = dist[node] + 1
                queue.append(neighbor)
    return dist

# Dijkstra (Positive Weights)
import heapq
def dijkstra(graph, start):
    dist = {start: 0}
    heap = [(0, start)]
    while heap:
        d, node = heapq.heappop(heap)
        if d > dist.get(node, float('inf')):
            continue
        for neighbor, weight in graph[node]:
            new_dist = d + weight
            if new_dist < dist.get(neighbor, float('inf')):
                dist[neighbor] = new_dist
                heapq.heappush(heap, (new_dist, neighbor))
    return dist

# Bellman-Ford
def bellman_ford(n, edges, start):
    dist = [float('inf')] * n
    dist[start] = 0
    for _ in range(n - 1):
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            return None  # Negative cycle
    return dist
```

#### Java

```java
// Dijkstra
int[] dijkstra(List<List<int[]>> graph, int start) {
    int n = graph.size();
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[start] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    pq.offer(new int[]{start, 0});
    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int node = curr[0], d = curr[1];
        if (d > dist[node]) continue;
        for (int[] edge : graph.get(node)) {
            int next = edge[0], w = edge[1];
            if (d + w < dist[next]) {
                dist[next] = d + w;
                pq.offer(new int[]{next, dist[next]});
            }
        }
    }
    return dist;
}
```

#### C++

```cpp
// Dijkstra
vector<int> dijkstra(vector<vector<pair<int,int>>>& graph, int start) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX);
    dist[start] = 0;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    pq.push({0, start});
    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (d > dist[node]) continue;
        for (auto& [next, w] : graph[node]) {
            if (d + w < dist[next]) {
                dist[next] = d + w;
                pq.push({dist[next], next});
            }
        }
    }
    return dist;
}
```

#### JavaScript

```javascript
// Dijkstra
function dijkstra(graph, start) {
    const dist = new Map();
    dist.set(start, 0);
    const heap = new MinPriorityQueue(x => x[1]);
    heap.enqueue([start, 0]);
    while (!heap.isEmpty()) {
        const [node, d] = heap.dequeue();
        if (d > (dist.get(node) ?? Infinity)) continue;
        for (const [next, w] of graph[node]) {
            const newDist = d + w;
            if (newDist < (dist.get(next) ?? Infinity)) {
                dist.set(next, newDist);
                heap.enqueue([next, newDist]);
            }
        }
    }
    return dist;
}
```

---

### Common Mistakes

#### Mistake 1: Using Dijkstra with Negative Weights

```python
# WRONG: Dijkstra doesn't work with negative weights
# Example: A→B (weight 1), A→C (weight 2), C→B (weight -2)
# Dijkstra: processes A, then B (dist=1), then C (dist=2)
# But A→C→B = 2 + (-2) = 0 < 1, so the answer is wrong!

# RIGHT: Use Bellman-Ford for negative weights
```

#### Mistake 2: Not Checking for Visited/Processed Nodes

```python
# WRONG: might process same node multiple times
while heap:
    d, node = heapq.heappop(heap)
    # Process node and update neighbors

# RIGHT: skip if already processed
while heap:
    d, node = heapq.heappop(heap)
    if d > dist[node]:
        continue  # Already found shorter path
```

#### Mistake 3: Off-by-One in Bellman-Ford

```python
# WRONG: only V-2 iterations (should be V-1)
for _ in range(n - 2):  # Wrong!

# RIGHT: V-1 iterations
for _ in range(n - 1):  # Correct
```

---

### Practice Problems

| # | Problem | Algorithm |
|---|---------|-----------|
| 1 | [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Dijkstra |
| 2 | [787. Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Bellman-Ford or BFS |
| 3 | [787. Cheapest Flights](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Dijkstra with constraint |
| 4 | [1514. Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) | Dijkstra (maximize) |
| 5 | [778. Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Dijkstra |
| 6 | [1631. Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Dijkstra |
| 7 | [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | BFS |
| 8 | [127. Word Ladder](https://leetcode.com/problems/word-ladder/) | BFS |
| 9 | [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Dijkstra |
| 10 | [1334. Find the City With the Smallest Number of Neighbors](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) | Floyd-Warshall |

---

### Interview Tips

1. **Ask about edge weights.** "Are all weights positive? Can there be negative weights?"

2. **State your algorithm choice.** "Since all weights are positive, I'll use Dijkstra's algorithm."

3. **Explain the approach.** "I maintain a min-heap of (distance, node) pairs. I always process the closest unvisited node."

4. **Mention time complexity.** "Dijkstra is O((V + E) log V) with a binary heap."

5. **Edge cases:**
   - Disconnected graph (some nodes unreachable)
   - Negative cycles (Bellman-Ford detects these)
   - Single node
   - Self-loops

---

*Next: [16 — Bit Manipulation](16-Bit-Manipulation.md)*
