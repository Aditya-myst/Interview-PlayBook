# 09 — DFS and BFS: Complete Guide

## The Fundamental Graph Traversal Patterns That Unlock Complex Problems

---

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [When to Use Each Algorithm](#when-to-use-each-algorithm)
3. [Mental Models and Visualization](#mental-models-and-visualization)
4. [DFS Templates](#dfs-templates)
5. [BFS Templates](#bfs-templates)
6. [Grid-Based Problems](#grid-based-problems)
7. [Common Mistakes and How to Avoid Them](#common-mistakes-and-how-to-avoid-them)
8. [Dry Runs and Step-by-Step Walkthroughs](#dry-runs-and-step-by-step-walkthroughs)
9. [Practice Problems](#practice-problems)

---

## Core Concepts

### What Is DFS (Depth-First Search)?

**Depth-First Search (DFS)** is a graph traversal algorithm that explores as far as possible along each branch before backtracking. It employs a **Last-In-First-Out (LIFO)** strategy through the use of a stack (or recursion).

**Key Characteristics:**
- Explores deeply into branches before exploring siblings
- Uses either recursion (implicit stack) or an explicit stack data structure
- Naturally suited for problems involving paths, connectivity, and exhaustive exploration
- Memory efficient for deep trees (uses O(h) space where h is depth)
- Can be inefficient for wide trees (in worst case, still O(n))

**Real-world analogy:** Like exploring a maze by always turning left and following the wall until you hit a dead end, then backtracking to try other paths.

### What Is BFS (Breadth-First Search)?

**Breadth-First Search (BFS)** is a graph traversal algorithm that explores all neighbors at the current depth level before moving to nodes at the next depth level. It employs a **First-In-First-Out (FIFO)** strategy through the use of a queue.

**Key Characteristics:**
- Explores level-by-level, visiting all neighbors before going deeper
- Uses a queue data structure for iteration
- Essential for finding shortest paths in unweighted graphs
- More memory-intensive than DFS in deep trees (uses O(w) space where w is width)
- Guarantees the shortest path in unweighted graphs

**Real-world analogy:** Like throwing a stone into water and watching the ripples expand outward in concentric circles, visiting all nearby points before reaching distant ones.

---

## When to Use Each Algorithm

### Decision Matrix

| Scenario | DFS | BFS | Both | Neither |
|----------|-----|-----|------|---------|
| All possible paths | ✓ | | | |
| Path exists (any path) | ✓ | ✓ | | |
| Shortest path (unweighted) | | ✓ | | |
| Longest path (unweighted) | ✓ | | | |
| Level-order traversal | | ✓ | | |
| Connected components count | ✓ | | | |
| Cycle detection | ✓ | ✓ | | |
| Topological sorting | ✓ | | | |
| Bipartite graph checking | ✓ | ✓ | | |
| Island problems | ✓ | ✓ | | |
| Flood fill | ✓ | ✓ | | |
| Shortest path in weighted graph | | | | ✓ (Dijkstra) |
| Multiple source shortest path | | ✓ | | |

### Trigger Words and Phrases

```
🔴 DFS Triggers:
  - "all paths" → Find every possible route
  - "connected components" → Count separate clusters
  - "cycle detection" → Find loops in graph
  - "topological sort" → Order with dependencies
  - "backtracking" → Try all possibilities
  - "deep exploration" → Go as far as possible
  - "can reach" → Connectivity check
  - "number of provinces" → Connected regions

🔵 BFS Triggers:
  - "shortest path" → Minimum distance/steps
  - "minimum steps" → Fewest moves needed
  - "level by level" → Process in layers
  - "closest node" → Nearest element
  - "word ladder" → Transform one to another
  - "rotting oranges" → Multi-source spread
  - "distance from source" → How far away
  - "alien dictionary" → Order from relationships
```

---

## Mental Models and Visualization

### DFS: The Depth Explorer

```
Visual Representation:
        1
       /|\
      2 3 4
     /|   
    5 6   

Exploration Order: 1 → 2 → 5 → (backtrack) → 6 → (backtrack) → 3 → (backtrack) → 4

Timeline:
  VISIT(1) → EXPLORE_BRANCH
    VISIT(2) → EXPLORE_BRANCH
      VISIT(5) → LEAF, BACKTRACK
      VISIT(6) → LEAF, BACKTRACK
    BACKTRACK
    VISIT(3) → LEAF, BACKTRACK
    VISIT(4) → LEAF, BACKTRACK
  DONE
```

**Key Insight:** DFS follows one path to its end, then backs up and tries another. It's greedy and committal.

### BFS: The Level Explorer

```
Visual Representation:
        1          (Level 0)
       /|\
      2 3 4        (Level 1)
     /|   |
    5 6   7        (Level 2)

Exploration Order: 1 → [2,3,4] → [5,6,7]

Timeline:
  QUEUE: [1]
  PROCESS 1: ADD CHILDREN → QUEUE: [2,3,4]
  PROCESS 2: ADD CHILDREN → QUEUE: [3,4,5,6]
  PROCESS 3: ADD CHILDREN → QUEUE: [4,5,6]
  PROCESS 4: ADD CHILDREN → QUEUE: [5,6,7]
  PROCESS 5,6,7: NO CHILDREN → QUEUE: []
  DONE
```

**Key Insight:** BFS processes all nodes at distance `d` before processing any node at distance `d+1`. It's systematic and comprehensive.

---

## DFS Templates

### Template 1: Recursive DFS (Graph)

The most intuitive and commonly used approach. Perfect for interview settings.

#### Python
```python
def dfs_recursive(node, visited, graph):
    """
    Recursive DFS traversal for graphs.
    
    Args:
        node: Current node being explored
        visited: Set to track visited nodes (prevent infinite loops)
        graph: Adjacency list or adjacency matrix representation
    
    Returns:
        None (modifies visited set in-place)
    
    Time Complexity: O(V + E) where V = vertices, E = edges
    Space Complexity: O(V) for recursion stack in worst case
    """
    # Base case: prevent revisiting
    if node in visited:
        return
    
    # Mark as visited IMMEDIATELY to prevent cycles
    visited.add(node)
    
    # Process current node (print, accumulate, etc.)
    print(f"Visiting node: {node}")
    
    # Recursively visit all unvisited neighbors
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs_recursive(neighbor, visited, graph)


# Example usage:
if __name__ == "__main__":
    # Graph represented as adjacency list
    graph = {
        1: [2, 3],
        2: [1, 4, 5],
        3: [1],
        4: [2],
        5: [2]
    }
    
    visited = set()
    dfs_recursive(1, visited, graph)
    print(f"Visited nodes: {visited}")
```

#### Java
```java
import java.util.*;

public class DFSRecursive {
    /**
     * Recursive DFS traversal for graphs.
     * 
     * @param node Current node being explored
     * @param visited Set to track visited nodes
     * @param graph Adjacency list representation
     * 
     * Time Complexity: O(V + E)
     * Space Complexity: O(V) for recursion stack
     */
    public static void dfsRecursive(int node, 
                                    Set<Integer> visited, 
                                    List<List<Integer>> graph) {
        // Prevent revisiting
        if (visited.contains(node)) {
            return;
        }
        
        // Mark as visited immediately
        visited.add(node);
        
        // Process current node
        System.out.println("Visiting node: " + node);
        
        // Recursively visit all unvisited neighbors
        for (int neighbor : graph.get(node)) {
            if (!visited.contains(neighbor)) {
                dfsRecursive(neighbor, visited, graph);
            }
        }
    }
    
    // Example usage
    public static void main(String[] args) {
        // Build graph: 1 -> [2,3], 2 -> [1,4,5], etc.
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < 6; i++) {
            graph.add(new ArrayList<>());
        }
        
        // Add edges (undirected)
        graph.get(1).addAll(Arrays.asList(2, 3));
        graph.get(2).addAll(Arrays.asList(1, 4, 5));
        graph.get(3).add(1);
        graph.get(4).add(2);
        graph.get(5).add(2);
        
        Set<Integer> visited = new HashSet<>();
        dfsRecursive(1, visited, graph);
        System.out.println("Visited nodes: " + visited);
    }
}
```

#### C++
```cpp
#include <iostream>
#include <unordered_set>
#include <vector>

/**
 * Recursive DFS traversal for graphs.
 * 
 * Time Complexity: O(V + E)
 * Space Complexity: O(V) for recursion stack
 */
void dfsRecursive(int node, 
                  std::unordered_set<int>& visited, 
                  const std::vector<std::vector<int>>& graph) {
    // Prevent revisiting
    if (visited.find(node) != visited.end()) {
        return;
    }
    
    // Mark as visited immediately
    visited.insert(node);
    
    // Process current node
    std::cout << "Visiting node: " << node << std::endl;
    
    // Recursively visit all unvisited neighbors
    for (int neighbor : graph[node]) {
        if (visited.find(neighbor) == visited.end()) {
            dfsRecursive(neighbor, visited, graph);
        }
    }
}

int main() {
    // Build graph: 1 -> [2,3], 2 -> [1,4,5], etc.
    std::vector<std::vector<int>> graph(6);
    
    // Add edges (undirected)
    graph[1] = {2, 3};
    graph[2] = {1, 4, 5};
    graph[3] = {1};
    graph[4] = {2};
    graph[5] = {2};
    
    std::unordered_set<int> visited;
    dfsRecursive(1, visited, graph);
    
    std::cout << "Visited nodes: ";
    for (int node : visited) {
        std::cout << node << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

### Template 2: Iterative DFS (Explicit Stack)

Use this when recursion depth might cause stack overflow or when you need more control.

#### Python
```python
def dfs_iterative(start_node, graph):
    """
    Iterative DFS using explicit stack.
    
    Advantages:
        - No risk of stack overflow from recursion
        - More control over iteration
        - Useful for very deep graphs
    
    Args:
        start_node: Node to start traversal
        graph: Adjacency list representation
    
    Returns:
        Set of visited nodes
    
    Time Complexity: O(V + E)
    Space Complexity: O(V) for stack storage
    """
    stack = [start_node]
    visited = set()
    
    while stack:
        node = stack.pop()  # LIFO - this is what makes it a stack
        
        # Skip if already visited (handles duplicates in stack)
        if node in visited:
            continue
        
        # Mark as visited
        visited.add(node)
        
        # Process node
        print(f"Visiting node: {node}")
        
        # Add unvisited neighbors to stack (in reverse for left-to-right order)
        for neighbor in reversed(graph.get(node, [])):
            if neighbor not in visited:
                stack.append(neighbor)
    
    return visited


# Example usage
if __name__ == "__main__":
    graph = {
        1: [2, 3],
        2: [1, 4, 5],
        3: [1],
        4: [2],
        5: [2]
    }
    
    visited_nodes = dfs_iterative(1, graph)
    print(f"All visited nodes: {visited_nodes}")
```

#### Java
```java
import java.util.*;

public class DFSIterative {
    /**
     * Iterative DFS using explicit stack.
     * 
     * @param startNode Node to start traversal
     * @param graph Adjacency list representation
     * @return Set of visited nodes
     * 
     * Time Complexity: O(V + E)
     * Space Complexity: O(V) for stack storage
     */
    public static Set<Integer> dfsIterative(int startNode, 
                                            List<List<Integer>> graph) {
        Stack<Integer> stack = new Stack<>();
        Set<Integer> visited = new HashSet<>();
        
        stack.push(startNode);
        
        while (!stack.isEmpty()) {
            int node = stack.pop();
            
            // Skip if already visited
            if (visited.contains(node)) {
                continue;
            }
            
            // Mark as visited
            visited.add(node);
            
            // Process node
            System.out.println("Visiting node: " + node);
            
            // Add unvisited neighbors to stack
            List<Integer> neighbors = graph.get(node);
            // Reverse iteration to maintain left-to-right order
            for (int i = neighbors.size() - 1; i >= 0; i--) {
                int neighbor = neighbors.get(i);
                if (!visited.contains(neighbor)) {
                    stack.push(neighbor);
                }
            }
        }
        
        return visited;
    }
    
    public static void main(String[] args) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < 6; i++) {
            graph.add(new ArrayList<>());
        }
        
        graph.get(1).addAll(Arrays.asList(2, 3));
        graph.get(2).addAll(Arrays.asList(1, 4, 5));
        graph.get(3).add(1);
        graph.get(4).add(2);
        graph.get(5).add(2);
        
        Set<Integer> visitedNodes = dfsIterative(1, graph);
        System.out.println("All visited nodes: " + visitedNodes);
    }
}
```

#### C++
```cpp
#include <iostream>
#include <stack>
#include <unordered_set>
#include <vector>

/**
 * Iterative DFS using explicit stack.
 * 
 * Time Complexity: O(V + E)
 * Space Complexity: O(V) for stack storage
 */
std::unordered_set<int> dfsIterative(int startNode, 
                                      const std::vector<std::vector<int>>& graph) {
    std::stack<int> stack;
    std::unordered_set<int> visited;
    
    stack.push(startNode);
    
    while (!stack.empty()) {
        int node = stack.top();
        stack.pop();
        
        // Skip if already visited
        if (visited.find(node) != visited.end()) {
            continue;
        }
        
        // Mark as visited
        visited.insert(node);
        
        // Process node
        std::cout << "Visiting node: " << node << std::endl;
        
        // Add unvisited neighbors to stack
        const auto& neighbors = graph[node];
        // Push in reverse order to maintain left-to-right exploration order
        for (int i = neighbors.size() - 1; i >= 0; --i) {
            int neighbor = neighbors[i];
            if (visited.find(neighbor) == visited.end()) {
                stack.push(neighbor);
            }
        }
    }
    
    return visited;
}

int main() {
    std::vector<std::vector<int>> graph(6);
    
    graph[1] = {2, 3};
    graph[2] = {1, 4, 5};
    graph[3] = {1};
    graph[4] = {2};
    graph[5] = {2};
    
    auto visitedNodes = dfsIterative(1, graph);
    
    std::cout << "All visited nodes: ";
    for (int node : visitedNodes) {
        std::cout << node << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## BFS Templates

### Template 1: Standard BFS (Graph)

Essential for shortest path problems in unweighted graphs.

#### Python
```python
from collections import deque

def bfs(start_node, graph):
    """
    Standard BFS traversal for graphs.
    
    Characteristics:
        - Explores level by level
        - FIFO queue ensures shortest path in unweighted graphs
        - Systematic and comprehensive
    
    Args:
        start_node: Node to start traversal
        graph: Adjacency list representation
    
    Returns:
        Dictionary with nodes and their distances from start
    
    Time Complexity: O(V + E)
    Space Complexity: O(V) for queue
    """
    queue = deque([start_node])
    visited = {start_node}
    distances = {start_node: 0}
    
    while queue:
        node = queue.popleft()  # FIFO - this is what makes it a queue
        current_distance = distances[node]
        
        # Process node
        print(f"Visiting node: {node} at distance {current_distance}")
        
        # Explore all unvisited neighbors
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                distances[neighbor] = current_distance + 1
                queue.append(neighbor)
    
    return distances


# Example usage
if __name__ == "__main__":
    graph = {
        1: [2, 3],
        2: [1, 4, 5],
        3: [1],
        4: [2],
        5: [2]
    }
    
    distances = bfs(1, graph)
    print(f"Distances from node 1: {distances}")
```

#### Java
```java
import java.util.*;

public class BFSGraph {
    /**
     * Standard BFS traversal for graphs.
     * 
     * @param startNode Node to start traversal
     * @param graph Adjacency list representation
     * @return Map of nodes and their distances from start
     * 
     * Time Complexity: O(V + E)
     * Space Complexity: O(V) for queue
     */
    public static Map<Integer, Integer> bfs(int startNode, 
                                            List<List<Integer>> graph) {
        Queue<Integer> queue = new LinkedList<>();
        Set<Integer> visited = new HashSet<>();
        Map<Integer, Integer> distances = new HashMap<>();
        
        queue.offer(startNode);
        visited.add(startNode);
        distances.put(startNode, 0);
        
        while (!queue.isEmpty()) {
            int node = queue.poll();
            int currentDistance = distances.get(node);
            
            // Process node
            System.out.println("Visiting node: " + node + " at distance " + currentDistance);
            
            // Explore all unvisited neighbors
            for (int neighbor : graph.get(node)) {
                if (!visited.contains(neighbor)) {
                    visited.add(neighbor);
                    distances.put(neighbor, currentDistance + 1);
                    queue.offer(neighbor);
                }
            }
        }
        
        return distances;
    }
    
    public static void main(String[] args) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < 6; i++) {
            graph.add(new ArrayList<>());
        }
        
        graph.get(1).addAll(Arrays.asList(2, 3));
        graph.get(2).addAll(Arrays.asList(1, 4, 5));
        graph.get(3).add(1);
        graph.get(4).add(2);
        graph.get(5).add(2);
        
        Map<Integer, Integer> distances = bfs(1, graph);
        System.out.println("Distances from node 1: " + distances);
    }
}
```

#### C++
```cpp
#include <iostream>
#include <queue>
#include <unordered_set>
#include <unordered_map>
#include <vector>

/**
 * Standard BFS traversal for graphs.
 * 
 * Time Complexity: O(V + E)
 * Space Complexity: O(V) for queue
 */
std::unordered_map<int, int> bfs(int startNode, 
                                  const std::vector<std::vector<int>>& graph) {
    std::queue<int> q;
    std::unordered_set<int> visited;
    std::unordered_map<int, int> distances;
    
    q.push(startNode);
    visited.insert(startNode);
    distances[startNode] = 0;
    
    while (!q.empty()) {
        int node = q.front();
        q.pop();
        
        int currentDistance = distances[node];
        
        // Process node
        std::cout << "Visiting node: " << node << " at distance " << currentDistance << std::endl;
        
        // Explore all unvisited neighbors
        for (int neighbor : graph[node]) {
            if (visited.find(neighbor) == visited.end()) {
                visited.insert(neighbor);
                distances[neighbor] = currentDistance + 1;
                q.push(neighbor);
            }
        }
    }
    
    return distances;
}

int main() {
    std::vector<std::vector<int>> graph(6);
    
    graph[1] = {2, 3};
    graph[2] = {1, 4, 5};
    graph[3] = {1};
    graph[4] = {2};
    graph[5] = {2};
    
    auto distances = bfs(1, graph);
    
    std::cout << "Distances from node 1: ";
    for (const auto& [node, dist] : distances) {
        std::cout << "(" << node << ":" << dist << ") ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## Grid-Based Problems

### DFS on Grids

When dealing with 2D grids, each cell is a node, and adjacent cells are neighbors (typically 4-directional).

#### Python
```python
def dfs_grid(grid, start_row, start_col, visited=None):
    """
    DFS traversal on 2D grid.
    
    Handles:
        - Boundary checking
        - Cell state validation
        - 4-directional movement
    
    Args:
        grid: 2D list where 0 = obstacle, 1 = valid
        start_row, start_col: Starting cell coordinates
        visited: Set to track visited cells (optional, creates if None)
    
    Returns:
        None (modifies visited set)
    
    Time Complexity: O(rows × cols)
    Space Complexity: O(rows × cols) for visited set
    """
    if visited is None:
        visited = set()
    
    # Boundary checks (crucial for grids!)
    if (start_row < 0 or start_row >= len(grid) or
        start_col < 0 or start_col >= len(grid[0]) or
        (start_row, start_col) in visited or
        grid[start_row][start_col] == 0):  # 0 = obstacle
        return
    
    # Mark as visited
    visited.add((start_row, start_col))
    
    # Process current cell
    print(f"Visiting cell: ({start_row}, {start_col})")
    
    # 4-directional movement: up, down, left, right
    directions = [(-1, 0), (1, 0), (0, -1), (0, 1)]
    
    for dr, dc in directions:
        new_row, new_col = start_row + dr, start_col + dc
        dfs_grid(grid, new_row, new_col, visited)
    
    return visited


# Example usage
if __name__ == "__main__":
    grid = [
        [1, 1, 0, 1],
        [1, 0, 1, 1],
        [1, 1, 1, 0],
        [0, 1, 1, 1]
    ]
    
    visited = dfs_grid(grid, 0, 0)
    print(f"Visited cells: {visited}")
```

#### Java
```java
import java.util.HashSet;
import java.util.Set;

public class DFSGrid {
    /**
     * DFS traversal on 2D grid with 4-directional movement.
     * 
     * @param grid 2D array where 0 = obstacle, 1 = valid
     * @param row Current row coordinate
     * @param col Current column coordinate
     * @param visited Set to track visited cells
     * 
     * Time Complexity: O(rows × cols)
     * Space Complexity: O(rows × cols)
     */
    public static void dfsGrid(int[][] grid, 
                               int row, 
                               int col, 
                               Set<String> visited) {
        // Boundary checks
        if (row < 0 || row >= grid.length || 
            col < 0 || col >= grid[0].length ||
            visited.contains(row + "," + col) ||
            grid[row][col] == 0) {  // 0 = obstacle
            return;
        }
        
        // Mark as visited
        visited.add(row + "," + col);
        
        // Process current cell
        System.out.println("Visiting cell: (" + row + ", " + col + ")");
        
        // 4-directional movement
        int[][] directions = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        
        for (int[] dir : directions) {
            int newRow = row + dir[0];
            int newCol = col + dir[1];
            dfsGrid(grid, newRow, newCol, visited);
        }
    }
    
    public static void main(String[] args) {
        int[][] grid = {
            {1, 1, 0, 1},
            {1, 0, 1, 1},
            {1, 1, 1, 0},
            {0, 1, 1, 1}
        };
        
        Set<String> visited = new HashSet<>();
        dfsGrid(grid, 0, 0, visited);
        System.out.println("Visited cells: " + visited.size());
    }
}
```

#### C++
```cpp
#include <iostream>
#include <set>
#include <vector>
#include <utility>

/**
 * DFS traversal on 2D grid.
 * 
 * Time Complexity: O(rows × cols)
 * Space Complexity: O(rows × cols)
 */
void dfsGrid(const std::vector<std::vector<int>>& grid,
             int row,
             int col,
             std::set<std::pair<int, int>>& visited) {
    int rows = grid.size();
    int cols = grid[0].size();
    
    // Boundary checks
    if (row < 0 || row >= rows ||
        col < 0 || col >= cols ||
        visited.find({row, col}) != visited.end() ||
        grid[row][col] == 0) {  // 0 = obstacle
        return;
    }
    
    // Mark as visited
    visited.insert({row, col});
    
    // Process current cell
    std::cout << "Visiting cell: (" << row << ", " << col << ")" << std::endl;
    
    // 4-directional movement
    int directions[][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    
    for (const auto& dir : directions) {
        int newRow = row + dir[0];
        int newCol = col + dir[1];
        dfsGrid(grid, newRow, newCol, visited);
    }
}

int main() {
    std::vector<std::vector<int>> grid = {
        {1, 1, 0, 1},
        {1, 0, 1, 1},
        {1, 1, 1, 0},
        {0, 1, 1, 1}
    };
    
    std::set<std::pair<int, int>> visited;
    dfsGrid(grid, 0, 0, visited);
    
    std::cout << "Visited cells: " << visited.size() << std::endl;
    
    return 0;
}
```

### BFS on Grids

Essential for finding shortest paths in grids and multi-source problems.

#### Python
```python
from collections import deque

def bfs_grid(grid, start_row, start_col):
    """
    BFS traversal on 2D grid finding distances from source.
    
    Returns:
        Dictionary mapping (row, col) to distance from start
    
    Time Complexity: O(rows × cols)
    Space Complexity: O(rows × cols)
    """
    rows, cols = len(grid), len(grid[0])
    queue = deque([(start_row, start_col, 0)])  # (row, col, distance)
    visited = {(start_row, start_col)}
    distances = {(start_row, start_col): 0}
    
    # 4-directional movement
    directions = [(-1, 0), (1, 0), (0, -1), (0, 1)]
    
    while queue:
        row, col, dist = queue.popleft()
        
        # Process current cell
        print(f"Processing cell: ({row}, {col}) at distance {dist}")
        
        # Explore all 4 neighbors
        for dr, dc in directions:
            new_row, new_col = row + dr, col + dc
            
            # Validate bounds and state
            if (0 <= new_row < rows and
                0 <= new_col < cols and
                (new_row, new_col) not in visited and
                grid[new_row][new_col] != 0):  # 0 = obstacle
                
                visited.add((new_row, new_col))
                new_distance = dist + 1
                distances[(new_row, new_col)] = new_distance
                queue.append((new_row, new_col, new_distance))
    
    return distances


# Example usage
if __name__ == "__main__":
    grid = [
        [1, 1, 0, 1],
        [1, 0, 1, 1],
        [1, 1, 1, 0],
        [0, 1, 1, 1]
    ]
    
    distances = bfs_grid(grid, 0, 0)
    for (row, col), dist in sorted(distances.items()):
        print(f"Cell ({row}, {col}): distance = {dist}")
```

#### Java
```java
import java.util.*;

public class BFSGrid {
    static class Cell {
        int row, col, distance;
        Cell(int r, int c, int d) { row = r; col = c; distance = d; }
    }
    
    /**
     * BFS on 2D grid to find distances from source.
     * 
     * Time Complexity: O(rows × cols)
     * Space Complexity: O(rows × cols)
     */
    public static Map<String, Integer> bfsGrid(int[][] grid, 
                                               int startRow, 
                                               int startCol) {
        int rows = grid.length;
        int cols = grid[0].length;
        
        Queue<Cell> queue = new LinkedList<>();
        Set<String> visited = new HashSet<>();
        Map<String, Integer> distances = new HashMap<>();
        
        String startKey = startRow + "," + startCol;
        queue.offer(new Cell(startRow, startCol, 0));
        visited.add(startKey);
        distances.put(startKey, 0);
        
        // 4-directional movement
        int[][] directions = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        
        while (!queue.isEmpty()) {
            Cell current = queue.poll();
            
            System.out.println("Processing cell: (" + current.row + ", " + 
                             current.col + ") at distance " + current.distance);
            
            // Explore 4 neighbors
            for (int[] dir : directions) {
                int newRow = current.row + dir[0];
                int newCol = current.col + dir[1];
                String key = newRow + "," + newCol;
                
                // Validate
                if (newRow >= 0 && newRow < rows &&
                    newCol >= 0 && newCol < cols &&
                    !visited.contains(key) &&
                    grid[newRow][newCol] != 0) {
                    
                    visited.add(key);
                    int newDist = current.distance + 1;
                    distances.put(key, newDist);
                    queue.offer(new Cell(newRow, newCol, newDist));
                }
            }
        }
        
        return distances;
    }
    
    public static void main(String[] args) {
        int[][] grid = {
            {1, 1, 0, 1},
            {1, 0, 1, 1},
            {1, 1, 1, 0},
            {0, 1, 1, 1}
        };
        
        Map<String, Integer> distances = bfsGrid(grid, 0, 0);
        distances.forEach((key, dist) -> 
            System.out.println("Cell " + key + ": distance = " + dist)
        );
    }
}
```

#### C++
```cpp
#include <iostream>
#include <queue>
#include <set>
#include <map>
#include <vector>
#include <utility>

/**
 * BFS on 2D grid to find distances.
 * 
 * Time Complexity: O(rows × cols)
 * Space Complexity: O(rows × cols)
 */
std::map<std::pair<int, int>, int> bfsGrid(const std::vector<std::vector<int>>& grid,
                                            int startRow,
                                            int startCol) {
    int rows = grid.size();
    int cols = grid[0].size();
    
    std::queue<std::tuple<int, int, int>> q;  // row, col, distance
    std::set<std::pair<int, int>> visited;
    std::map<std::pair<int, int>, int> distances;
    
    q.push({startRow, startCol, 0});
    visited.insert({startRow, startCol});
    distances[{startRow, startCol}] = 0;
    
    // 4-directional movement
    int directions[][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    
    while (!q.empty()) {
        auto [row, col, dist] = q.front();
        q.pop();
        
        std::cout << "Processing cell: (" << row << ", " << col 
                  << ") at distance " << dist << std::endl;
        
        // Explore 4 neighbors
        for (const auto& dir : directions) {
            int newRow = row + dir[0];
            int newCol = col + dir[1];
            
            // Validate
            if (newRow >= 0 && newRow < rows &&
                newCol >= 0 && newCol < cols &&
                visited.find({newRow, newCol}) == visited.end() &&
                grid[newRow][newCol] != 0) {
                
                visited.insert({newRow, newCol});
                int newDist = dist + 1;
                distances[{newRow, newCol}] = newDist;
                q.push({newRow, newCol, newDist});
            }
        }
    }
    
    return distances;
}

int main() {
    std::vector<std::vector<int>> grid = {
        {1, 1, 0, 1},
        {1, 0, 1, 1},
        {1, 1, 1, 0},
        {0, 1, 1, 1}
    };
    
    auto distances = bfsGrid(grid, 0, 0);
    
    for (const auto& [coord, dist] : distances) {
        std::cout << "Cell (" << coord.first << ", " << coord.second 
                  << "): distance = " << dist << std::endl;
    }
    
    return 0;
}
```

---

## Common Mistakes and How to Avoid Them

### Mistake 1: Forgetting to Mark Visited Immediately

**The Problem:**
```python
# ❌ WRONG: Mark visited AFTER exploring
def dfs_wrong(node, visited):
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs_wrong(neighbor, visited)
    visited.add(node)  # TOO LATE! Revisits possible

# ✅ CORRECT: Mark visited BEFORE exploring
def dfs_correct(node, visited):
    visited.add(node)  # Mark immediately
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs_correct(neighbor, visited)
```

**Why it matters:** In cyclic graphs, a node can be reached via multiple paths. If you mark it after exploring neighbors, the same node gets reprocessed, causing:
- Infinite loops
- Exponential time complexity
- Stack overflow

---

### Mistake 2: Forgetting Boundary Checks in Grids

**The Problem:**
```python
# ❌ WRONG: Access before checking bounds
def dfs_wrong(grid, i, j):
    if grid[i][j] != 0:  # What if i < 0?
        process(grid[i][j])

# ✅ CORRECT: Check bounds FIRST
def dfs_correct(grid, i, j):
    if i < 0 or i >= len(grid) or j < 0 or j >= len(grid[0]):
        return  # Out of bounds
    if grid[i][j] == 0:
        return  # Obstacle or visited
    # Now safe to process
    process(grid[i][j])
```

**Impact:** Index out of bounds errors leading to runtime crashes.

---

### Mistake 3: Using DFS for Shortest Path

**The Problem:**
```python
# ❌ WRONG: DFS doesn't guarantee shortest path
# DFS explores deeply, might find longer path first

# ✅ CORRECT: Use BFS for shortest path
def shortest_path_bfs(graph, start, end):
    queue = deque([start])
    distances = {start: 0}
    while queue:
        node = queue.popleft()
        if node == end:
            return distances[end]  # First found = shortest
        for neighbor in graph[node]:
            if neighbor not in distances:
                distances[neighbor] = distances[node] + 1
                queue.append(neighbor)
    return -1  # No path
```

**Key insight:** BFS processes nodes by distance level, so first encounter = shortest path.

---

### Mistake 4: Incorrect Queue vs Stack Usage

**The Problem:**
```python
# ❌ WRONG: Using list as queue (inefficient)
queue = []
queue.append(item)  # O(1)
item = queue.pop(0)  # O(n) - slow!

# ✅ CORRECT: Use deque for queue
from collections import deque
queue = deque()
queue.append(item)  # O(1)
item = queue.popleft()  # O(1) - fast!

# ❌ WRONG: Using list as stack (ok but inconsistent)
stack = []
stack.append(item)  # O(1)
item = stack.pop()  # O(1) - ok

# ✅ CORRECT: Explicit stack usage
stack = []
stack.append(item)
item = stack.pop()  # Clear intent
```

---

## Dry Runs and Step-by-Step Walkthroughs

### Example 1: Number of Islands

**Problem Statement:**
Count the number of islands in a 2D grid where '1' is land and '0' is water.

**Input:**
```
grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
```

**Expected Output:** `3`

#### Python Solution
```python
def numIslands(grid):
    """
    Count number of disconnected components in grid.
    
    Approach:
        1. Iterate through each cell
        2. When finding unvisited land, start DFS
        3. Mark entire island as visited
        4. Increment island count
    
    Time: O(m × n) - visit each cell once
    Space: O(m × n) - visited set
    """
    if not grid:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    visited = set()
    count = 0
    
    def dfs(r, c):
        # Boundary and state check
        if (r < 0 or r >= rows or c < 0 or c >= cols or
            (r, c) in visited or grid[r][c] == '0'):
            return
        
        visited.add((r, c))
        
        # Explore 4 neighbors
        dfs(r - 1, c)  # up
        dfs(r + 1, c)  # down
        dfs(r, c - 1)  # left
        dfs(r, c + 1)  # right
    
    # Scan entire grid
    for i in range(rows):
        for j in range(cols):
            if grid[i][j] == '1' and (i, j) not in visited:
                dfs(i, j)  # Explore entire island
                count += 1
    
    return count


# Test
grid = [
    ["1","1","0","0","0"],
    ["1","1","0","0","0"],
    ["0","0","1","0","0"],
    ["0","0","0","1","1"]
]
print(f"Number of islands: {numIslands(grid)}")  # Output: 3
```

#### Java Solution
```java
public class IslandCounter {
    public static int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) {
            return 0;
        }
        
        int rows = grid.length;
        int cols = grid[0].length;
        boolean[][] visited = new boolean[rows][cols];
        int count = 0;
        
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (grid[i][j] == '1' && !visited[i][j]) {
                    dfs(grid, i, j, visited);
                    count++;
                }
            }
        }
        
        return count;
    }
    
    private static void dfs(char[][] grid, 
                           int row, 
                           int col, 
                           boolean[][] visited) {
        int rows = grid.length;
        int cols = grid[0].length;
        
        // Boundary check
        if (row < 0 || row >= rows || col < 0 || col >= cols ||
            visited[row][col] || grid[row][col] == '0') {
            return;
        }
        
        visited[row][col] = true;
        
        // 4-directional exploration
        dfs(grid, row - 1, col, visited);  // up
        dfs(grid, row + 1, col, visited);  // down
        dfs(grid, row, col - 1, visited);  // left
        dfs(grid, row, col + 1, visited);  // right
    }
    
    public static void main(String[] args) {
        char[][] grid = {
            {'1','1','0','0','0'},
            {'1','1','0','0','0'},
            {'0','0','1','0','0'},
            {'0','0','0','1','1'}
        };
        System.out.println("Number of islands: " + numIslands(grid));  // 3
    }
}
```

#### C++ Solution
```cpp
#include <iostream>
#include <vector>

using namespace std;

class IslandCounter {
private:
    void dfs(vector<vector<char>>& grid, 
             int row, 
             int col, 
             vector<vector<bool>>& visited) {
        int rows = grid.size();
        int cols = grid[0].size();
        
        // Boundary check
        if (row < 0 || row >= rows || col < 0 || col >= cols ||
            visited[row][col] || grid[row][col] == '0') {
            return;
        }
        
        visited[row][col] = true;
        
        // 4-directional exploration
        dfs(grid, row - 1, col, visited);  // up
        dfs(grid, row + 1, col, visited);  // down
        dfs(grid, row, col - 1, visited);  // left
        dfs(grid, row, col + 1, visited);  // right
    }

public:
    int numIslands(vector<vector<char>>& grid) {
        if (grid.empty() || grid[0].empty()) {
            return 0;
        }
        
        int rows = grid.size();
        int cols = grid[0].size();
        vector<vector<bool>> visited(rows, vector<bool>(cols, false));
        int count = 0;
        
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (grid[i][j] == '1' && !visited[i][j]) {
                    dfs(grid, i, j, visited);
                    count++;
                }
            }
        }
        
        return count;
    }
};

int main() {
    IslandCounter ic;
    vector<vector<char>> grid = {
        {'1','1','0','0','0'},
        {'1','1','0','0','0'},
        {'0','0','1','0','0'},
        {'0','0','0','1','1'}
    };
    
    cout << "Number of islands: " << ic.numIslands(grid) << endl;  // 3
    return 0;
}
```

**Step-by-Step Walkthrough:**
```
Grid:
  1 1 0 0 0
  1 1 0 0 0
  0 0 1 0 0
  0 0 0 1 1

Scan:
  (0,0): '1' not visited → DFS
    Mark (0,0), (0,1), (1,0), (1,1) as visited
    Island count = 1
  
  (0,1): '1' but visited → Skip
  ...continue scanning...
  
  (2,2): '1' not visited → DFS
    Mark (2,2) as visited
    Island count = 2
  
  (3,3): '1' not visited → DFS
    Mark (3,3), (3,4) as visited
    Island count = 3

Result: 3 islands ✓
```

---

## Practice Problems

### Easy Level
1. **Max Area of Island** - Find largest connected component
2. **Flood Fill** - Similar to paint bucket tool
3. **Same Tree** - DFS tree comparison
4. **Balanced Binary Tree** - DFS validation

### Medium Level
1. **Rotting Oranges** - Multi-source BFS
2. **Pacific Atlantic Water Flow** - DFS from edges
3. **Number of Connected Components** - Union-Find alternative
4. **Course Schedule** - Cycle detection in DAG

### Hard Level
1. **Longest Increasing Path** - DFS + Memoization
2. **Word Ladder** - BFS shortest path
3. **Alien Dictionary** - Topological sort

---

**Next: [10 — Backtracking](10-Backtracking.md)**
