# 14 — Topological Sort

## The Pattern That Orders Dependencies

---

### What It Is

Topological Sort produces a linear ordering of vertices in a Directed Acyclic Graph (DAG) such that for every edge (u, v), vertex u comes before vertex v.

**The key insight:** If there's a dependency (A must come before B), topological sort finds a valid ordering that respects all dependencies. If no valid ordering exists, there's a cycle.

---

### When to Use It

**The problem involves:**
- Task ordering with prerequisites
- Course scheduling
- Build order / dependency resolution
- Finding if a cycle exists in a directed graph

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "prerequisites" | Course Schedule |
| "dependencies" | Build Order |
| "order" + "before" | Task Scheduling |
| "can you finish" | Course Schedule |
| "alien dictionary" | Alien Dictionary |
| "parallel courses" | Parallel Courses |
| "sequence" | Sequence Reconstruction |

**The Decision:**

```
Does the problem involve ordering with dependencies?
    ↓
Yes
    ↓
Is the graph directed?
    ↓
Yes
    ↓
TOPOLOGICAL SORT
```

---

### Mental Model: Prerequisites

```
Courses: A, B, C, D
Prerequisites: A→B, A→C, B→D, C→D

    A
   / \
  B   C
   \ /
    D

Valid orderings: [A, B, C, D] or [A, C, B, D]
Invalid: [B, A, ...] (B requires A first)
```

---

### The Two Algorithms

#### Algorithm 1: Kahn's Algorithm (BFS-based)

Count incoming edges (indegrees). Process nodes with 0 indegree first.

```python
from collections import deque, defaultdict

def topological_sort(n, edges):
    # Build graph and indegree
    graph = defaultdict(list)
    indegree = [0] * n
    for u, v in edges:
        graph[u].append(v)
        indegree[v] += 1
    
    # Start with nodes having 0 indegree
    queue = deque([i for i in range(n) if indegree[i] == 0])
    order = []
    
    while queue:
        node = queue.popleft()
        order.append(node)
        
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)
    
    # If order doesn't contain all nodes, there's a cycle
    if len(order) != n:
        return []  # Cycle exists
    
    return order
```

#### Algorithm 2: DFS-based

Use DFS and add nodes to order in reverse (post-order).

```python
def topological_sort_dfs(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
    
    visited = [0] * n  # 0=unvisited, 1=visiting, 2=done
    order = []
    has_cycle = False
    
    def dfs(node):
        nonlocal has_cycle
        if visited[node] == 1:  # Cycle detected
            has_cycle = True
            return
        if visited[node] == 2:  # Already processed
            return
        
        visited[node] = 1  # Mark as visiting
        for neighbor in graph[node]:
            dfs(neighbor)
        visited[node] = 2  # Mark as done
        order.append(node)
    
    for i in range(n):
        if visited[i] == 0:
            dfs(i)
    
    if has_cycle:
        return []
    
    return order[::-1]  # Reverse for correct order
```

---

### Dry Run: Course Schedule

**Problem:** Given numCourses and prerequisites, determine if you can finish all courses.

**Input:** `numCourses = 4`, `prerequisites = [[1,0],[2,0],[3,1],[3,2]]`

**Recognition:** "can you finish" + "prerequisites" → Topological Sort (cycle detection)

```python
def canFinish(numCourses, prerequisites):
    graph = defaultdict(list)
    indegree = [0] * numCourses
    
    for course, prereq in prerequisites:
        graph[prereq].append(course)
        indegree[course] += 1
    
    queue = deque([i for i in range(numCourses) if indegree[i] == 0])
    count = 0
    
    while queue:
        node = queue.popleft()
        count += 1
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)
    
    return count == numCourses
```

**Step-by-step:**

```
numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]

Graph:
  0 → 1, 2
  1 → 3
  2 → 3

Indegree: [0, 1, 1, 2]

Queue: [0] (only 0 has indegree 0)
count = 0

Process 0: count = 1
  Neighbors: 1, 2
  indegree[1] = 0 → enqueue
  indegree[2] = 0 → enqueue
  Queue: [1, 2]

Process 1: count = 2
  Neighbor: 3
  indegree[3] = 1
  Queue: [2]

Process 2: count = 3
  Neighbor: 3
  indegree[3] = 0 → enqueue
  Queue: [3]

Process 3: count = 4
  No neighbors
  Queue: []

count (4) == numCourses (4) → return True

Valid order: [0, 1, 2, 3] or [0, 2, 1, 3]
```

---

### Dry Run: Alien Dictionary

**Problem:** Given a sorted list of words in an alien language, find the order of characters.

**Input:** `words = ["wrt", "wrf", "er", "ett", "rftt"]`

**Recognition:** "order" + "dictionary" → Topological Sort

**Insight:** Compare adjacent words to find character ordering constraints.

```python
def alienOrder(words):
    # Build graph
    graph = defaultdict(set)
    indegree = {c: 0 for word in words for c in word}
    
    for i in range(len(words) - 1):
        w1, w2 = words[i], words[i+1]
        min_len = min(len(w1), len(w2))
        
        # Edge case: w1 is longer and prefix of w2 (invalid)
        if len(w1) > len(w2) and w1[:min_len] == w2[:min_len]:
            return ""
        
        for j in range(min_len):
            if w1[j] != w2[j]:
                if w2[j] not in graph[w1[j]]:
                    graph[w1[j]].add(w2[j])
                    indegree[w2[j]] += 1
                break
    
    # Topological sort (Kahn's)
    queue = deque([c for c in indegree if indegree[c] == 0])
    order = []
    
    while queue:
        c = queue.popleft()
        order.append(c)
        for neighbor in graph[c]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)
    
    if len(order) != len(indegree):
        return ""  # Cycle
    
    return "".join(order)
```

**Step-by-step:**

```
words = ["wrt", "wrf", "er", "ett", "rftt"]

Compare "wrt" and "wrf":
  First difference at index 2: 't' vs 'f'
  Constraint: t → f

Compare "wrf" and "er":
  First difference at index 0: 'w' vs 'e'
  Constraint: w → e

Compare "er" and "ett":
  First difference at index 1: 'r' vs 't'
  Constraint: r → t

Compare "ett" and "rftt":
  First difference at index 0: 'e' vs 'r'
  Constraint: e → r

Graph: t→f, w→e, r→t, e→r
Indegree: {w:0, r:0, t:1, f:1, e:1}

Wait, let me recalculate:
  t→f: indegree[f]++
  w→e: indegree[e]++
  r→t: indegree[t]++
  e→r: indegree[r]++

Indegree: {w:0, t:1, f:1, e:1, r:1}

Queue: [w]
Process w: order=[w], enqueue e (indegree[e]=0)
Process e: order=[w,e], enqueue r (indegree[r]=0)
Process r: order=[w,e,r], enqueue t (indegree[t]=0)
Process t: order=[w,e,r,t], enqueue f (indegree[f]=0)
Process f: order=[w,e,r,t,f]

Return "wertf"
```

---

### The Two Approaches: Kahn's vs DFS

| Aspect | Kahn's (BFS) | DFS |
|--------|--------------|-----|
| Direction | Process nodes with no dependencies first | Process deepest nodes first |
| Cycle Detection | If processed < total nodes | If back edge found during DFS |
| Intuition | "What can I do now?" | "What must be done before this?" |
| Output Order | Natural ordering | Reverse of finish times |

**Use Kahn's when:**
- You want to process tasks as they become available
- You need to detect cycles naturally
- You want level-by-level processing

**Use DFS when:**
- You're already using DFS for other reasons
- You need to find all possible orderings
- The graph is more naturally explored depth-first

---

### Code Templates (4 Languages)

#### Python

```python
# Kahn's Algorithm
def topo_sort(n, edges):
    graph = defaultdict(list)
    indegree = [0] * n
    for u, v in edges:
        graph[u].append(v)
        indegree[v] += 1
    
    queue = deque([i for i in range(n) if indegree[i] == 0])
    order = []
    
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)
    
    return order if len(order) == n else []  # Empty if cycle

# DFS-based
def topo_sort_dfs(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
    
    visited = [0] * n
    order = []
    
    def dfs(node):
        if visited[node]: return visited[node] == 2
        visited[node] = 1
        for neighbor in graph[node]:
            if not dfs(neighbor): return False
        visited[node] = 2
        order.append(node)
        return True
    
    for i in range(n):
        if not visited[i]:
            if not dfs(i): return []
    
    return order[::-1]
```

#### Java

```java
// Kahn's Algorithm
List<Integer> topoSort(int n, int[][] edges) {
    List<List<Integer>> graph = new ArrayList<>();
    int[] indegree = new int[n];
    for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
    for (int[] e : edges) {
        graph.get(e[0]).add(e[1]);
        indegree[e[1]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) queue.offer(i);
    }
    
    List<Integer> order = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order.add(node);
        for (int next : graph.get(node)) {
            if (--indegree[next] == 0) queue.offer(next);
        }
    }
    
    return order.size() == n ? order : new ArrayList<>();
}
```

#### C++

```cpp
vector<int> topoSort(int n, vector<vector<int>>& edges) {
    vector<vector<int>> graph(n);
    vector<int> indegree(n, 0);
    for (auto& e : edges) {
        graph[e[0]].push_back(e[1]);
        indegree[e[1]]++;
    }
    
    queue<int> q;
    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) q.push(i);
    }
    
    vector<int> order;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);
        for (int next : graph[node]) {
            if (--indegree[next] == 0) q.push(next);
        }
    }
    
    return order.size() == n ? order : vector<int>();
}
```

#### JavaScript

```javascript
function topoSort(n, edges) {
    const graph = Array.from({length: n}, () => []);
    const indegree = new Array(n).fill(0);
    for (const [u, v] of edges) {
        graph[u].push(v);
        indegree[v]++;
    }
    
    const queue = [];
    for (let i = 0; i < n; i++) {
        if (indegree[i] === 0) queue.push(i);
    }
    
    const order = [];
    while (queue.length) {
        const node = queue.shift();
        order.push(node);
        for (const next of graph[node]) {
            if (--indegree[next] === 0) queue.push(next);
        }
    }
    
    return order.length === n ? order : [];
}
```

---

### Practice Problems

| # | Problem | Key |
|---|---------|-----|
| 1 | [207. Course Schedule](https://leetcode.com/problems/course-schedule/) | Cycle detection |
| 2 | [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Return order |
| 3 | [269. Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) | Build graph from constraints |
| 4 | [310. Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/) | Topological sort (leaf removal) |
| 5 | [2115. Find All Possible Recipes](https://leetcode.com/problems/find-all-possible-recipes-from-given-supplies/) | Topological sort |
| 6 | [1136. Parallel Courses](https://leetcode.com/problems/parallel-courses/) | Longest path in DAG |
| 7 | [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Topological order |
| 8 | [329. Longest Increasing Path in Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) | DAG + DFS |

---

### Interview Tips

1. **State the approach.** "This is a dependency problem, so I'll use topological sort."

2. **Explain Kahn's algorithm.** "I'll process nodes with no dependencies first, then reduce indegrees of their neighbors."

3. **Mention cycle detection.** "If not all nodes are processed, there's a cycle."

4. **Edge cases:**
   - No edges (any order works)
   - Cycle (no valid ordering)
   - Single node
   - Disconnected components

---

*Next: [15 — Shortest Path](15-Shortest-Path.md)*
