# 08 — Trees

## The Pattern That Recurses Naturally

---

### What It Is

Trees are hierarchical data structures where each node has at most one parent and zero or more children. Binary trees (each node has at most two children) are the most common in interviews.

**The key insight:** Almost every tree problem can be solved with recursion. The trick is defining what information each node needs from its children (the return value) and what to do at each node.

---

### When to Use It

**The problem involves:**
- A binary tree, BST, N-ary tree, or trie
- Traversal (inorder, preorder, postorder)
- Finding height, depth, diameter
- Path sums
- Validation (is it a BST?)
- Serialization/deserialization
- Lowest common ancestor

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "binary tree" | Maximum Depth of Binary Tree |
| "BST" | Validate BST |
| "inorder" | Binary Tree Inorder Traversal |
| "preorder" | Binary Tree Preorder Traversal |
| "postorder" | Binary Tree Postorder Traversal |
| "level order" | Binary Tree Level Order Traversal |
| "path sum" | Path Sum |
| "depth" | Maximum Depth |
| "height" | Balanced Binary Tree |
| "diameter" | Diameter of Binary Tree |
| "ancestor" | Lowest Common Ancestor |
| "serialize" | Serialize and Deserialize BST |
| "trie" | Implement Trie |

---

### The Three Sub-Patterns

```
Trees
├── Sub-pattern 1: Traversal (visit every node)
├── Sub-pattern 2: Divide & Conquer (compute from children)
└── Sub-pattern 3: BST Properties (use sorted order)
```

---

### Sub-Pattern 1: Traversal

Visit every node in a specific order.

```python
# Inorder (Left, Root, Right) — gives sorted order for BST
def inorder(root):
    if not root:
        return []
    return inorder(root.left) + [root.val] + inorder(root.right)

# Preorder (Root, Left, Right) — used for serialization
def preorder(root):
    if not root:
        return []
    return [root.val] + preorder(root.left) + preorder(root.right)

# Postorder (Left, Right, Root) — used for deletion
def postorder(root):
    if not root:
        return []
    return postorder(root.left) + postorder(root.right) + [root.val]
```

**Visual:**

```
        1
       / \
      2   3
     / \
    4   5

Inorder:   4, 2, 5, 1, 3
Preorder:  1, 2, 4, 5, 3
Postorder: 4, 5, 2, 3, 1
```

#### Iterative Inorder (using stack)

```python
def inorder_iterative(root):
    result = []
    stack = []
    curr = root
    
    while curr or stack:
        while curr:
            stack.append(curr)
            curr = curr.left
        curr = stack.pop()
        result.append(curr.val)
        curr = curr.right
    
    return result
```

---

### Sub-Pattern 2: Divide & Conquer

Compute a result by combining results from left and right subtrees.

```python
def max_depth(root):
    if not root:
        return 0
    left_depth = max_depth(root.left)
    right_depth = max_depth(root.right)
    return 1 + max(left_depth, right_depth)
```

**The template:**

```python
def solve(root):
    if not root:
        return base_case
    
    left = solve(root.left)
    right = solve(root.right)
    
    return combine(root.val, left, right)
```

**Examples:**

| Problem | Base Case | Combine |
|---------|-----------|---------|
| Max Depth | 0 | 1 + max(L, R) |
| Min Depth | 0 | 1 + min(L, R) (if both children exist) |
| Diameter | 0 | max(L+R, max(L, R)) |
| Balanced | (True, 0) | (abs(L-R)≤1 and both balanced, 1+max(L,R)) |
| Path Sum | False | check at each node |

---

### Sub-Pattern 3: BST Properties

Use the sorted property of BSTs: left < root < right.

```python
def validate_bst(root, lo=float('-inf'), hi=float('inf')):
    if not root:
        return True
    if root.val <= lo or root.val >= hi:
        return False
    return (validate_bst(root.left, lo, root.val) and
            validate_bst(root.right, root.val, hi))
```

**Key insight:** Each recursive call narrows the valid range.

---

### The 80% Template: Recursive Tree Problem

```python
def tree_problem(root):
    if not root:
        return base_case
    
    # Process left subtree
    left_result = tree_problem(root.left)
    
    # Process right subtree
    right_result = tree_problem(root.right)
    
    # Combine results at current node
    return combine(root, left_result, right_result)
```

**When to use this template:**
- Finding max/min depth
- Checking if tree is balanced
- Computing diameter
- Checking if two trees are the same
- Symmetric tree check

---

### Dry Run: Maximum Depth of Binary Tree

**Problem:** Given a binary tree, find its maximum depth.

**Input:**
```
    3
   / \
  9  20
    /  \
   15   7
```

```python
def maxDepth(root):
    if not root:
        return 0
    left = maxDepth(root.left)
    right = maxDepth(root.right)
    return 1 + max(left, right)
```

**Step-by-step:**

```
maxDepth(3)
├── maxDepth(9)
│   ├── maxDepth(None) → 0
│   └── maxDepth(None) → 0
│   → 1 + max(0, 0) = 1
├── maxDepth(20)
│   ├── maxDepth(15)
│   │   ├── maxDepth(None) → 0
│   │   └── maxDepth(None) → 0
│   │   → 1 + max(0, 0) = 1
│   └── maxDepth(7)
│       ├── maxDepth(None) → 0
│       └── maxDepth(None) → 0
│       → 1 + max(0, 0) = 1
│   → 1 + max(1, 1) = 2
→ 1 + max(1, 2) = 3

Return 3
```

---

### Dry Run: Validate Binary Search Tree

**Problem:** Determine if a binary tree is a valid BST.

**Input:**
```
    5
   / \
  1   7
     / \
    6   8
```

```python
def isValidBST(root, lo=float('-inf'), hi=float('inf')):
    if not root:
        return True
    if root.val <= lo or root.val >= hi:
        return False
    return (isValidBST(root.left, lo, root.val) and
            isValidBST(root.right, root.val, hi))
```

**Step-by-step:**

```
isValidBST(5, -inf, inf)
├── isValidBST(1, -inf, 5)
│   ├── isValidBST(None, -inf, 1) → True
│   └── isValidBST(None, 1, 5) → True
│   → True (1 is between -inf and 5)
├── isValidBST(7, 5, inf)
│   ├── isValidBST(6, 5, 7)
│   │   ├── isValidBST(None, 5, 6) → True
│   │   └── isValidBST(None, 6, 7) → True
│   │   → True (6 is between 5 and 7)
│   └── isValidBST(8, 7, inf)
│       ├── isValidBST(None, 7, 8) → True
│       └── isValidBST(None, 8, inf) → True
│       → True (8 is between 7 and inf)
│   → True
→ True

Return True ✓
```

**Counter-example:**
```
    5
   / \
  1   7
     / \
    4   8    ← 4 is NOT between 5 and 7, so this is invalid
```

---

### Dry Run: Lowest Common Ancestor

**Problem:** Find the lowest common ancestor of two nodes in a BST.

**Input:** BST with nodes 2 and 8 in the tree below.
```
        6
       / \
      2   8
     / \ / \
    0  4 7  9
```

```python
def lowestCommonAncestor(root, p, q):
    while root:
        if p.val < root.val and q.val < root.val:
            root = root.left
        elif p.val > root.val and q.val > root.val:
            root = root.right
        else:
            return root
```

**Step-by-step:**

```
root=6, p=2, q=8
  2 < 6 and 8 > 6 → they split at 6
  Return 6

If p=2, q=4:
root=6: both < 6, go left
root=2: p==2, found it
  Return 2
```

---

### BFS Level Order Template

```python
from collections import deque

def level_order(root):
    if not root:
        return []
    
    result = []
    queue = deque([root])
    
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)
    
    return result
```

**Visual:**

```
        3
       / \
      9  20
        /  \
       15   7

Queue: [3]
Level 0: [3], add children → Queue: [9, 20]

Queue: [9, 20]
Level 1: [9, 20], add children → Queue: [15, 7]

Queue: [15, 7]
Level 2: [15, 7], add children → Queue: []

Result: [[3], [9, 20], [15, 7]]
```

---

### Code Templates (4 Languages)

#### Python

```python
# Recursive Template
def solve(root):
    if not root:
        return base_case
    left = solve(root.left)
    right = solve(root.right)
    return combine(root.val, left, right)

# Iterative Inorder
def inorder(root):
    result, stack, curr = [], [], root
    while curr or stack:
        while curr:
            stack.append(curr)
            curr = curr.left
        curr = stack.pop()
        result.append(curr.val)
        curr = curr.right
    return result

# BFS Level Order
from collections import deque
def level_order(root):
    if not root: return []
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left: queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result
```

#### Java

```java
// Recursive Template
int solve(TreeNode root) {
    if (root == null) return baseCase;
    int left = solve(root.left);
    int right = solve(root.right);
    return combine(root.val, left, right);
}

// Iterative Inorder
List<Integer> inorder(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        result.add(curr.val);
        curr = curr.right;
    }
    return result;
}
```

#### C++

```cpp
// Recursive Template
int solve(TreeNode* root) {
    if (!root) return baseCase;
    int left = solve(root->left);
    int right = solve(root->right);
    return combine(root->val, left, right);
}

// Iterative Inorder
vector<int> inorder(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> st;
    TreeNode* curr = root;
    while (curr || !st.empty()) {
        while (curr) {
            st.push(curr);
            curr = curr->left;
        }
        curr = st.top(); st.pop();
        result.push_back(curr->val);
        curr = curr->right;
    }
    return result;
}
```

#### JavaScript

```javascript
// Recursive Template
function solve(root) {
    if (!root) return baseCase;
    const left = solve(root.left);
    const right = solve(root.right);
    return combine(root.val, left, right);
}

// Iterative Inorder
function inorder(root) {
    const result = [];
    const stack = [];
    let curr = root;
    while (curr || stack.length) {
        while (curr) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        result.push(curr.val);
        curr = curr.right;
    }
    return result;
}
```

---

### Common Mistakes

#### Mistake 1: Not Handling Null Nodes

```python
# WRONG: will crash on None
def max_depth(root):
    return 1 + max(max_depth(root.left), max_depth(root.right))

# RIGHT: check for None first
def max_depth(root):
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

#### Mistake 2: Wrong BST Validation

```python
# WRONG: only checks immediate children
def is_bst(root):
    if not root:
        return True
    if root.left and root.left.val >= root.val:
        return False
    if root.right and root.right.val <= root.val:
        return False
    return is_bst(root.left) and is_bst(root.right)

# Problem: This tree would pass but isn't a BST:
#     5
#    / \
#   1   7
#      / \
#     4   8   ← 4 < 5 but it's in the right subtree!

# RIGHT: pass down valid range
def is_bst(root, lo=float('-inf'), hi=float('inf')):
    if not root:
        return True
    if root.val <= lo or root.val >= hi:
        return False
    return is_bst(root.left, lo, root.val) and is_bst(root.right, root.val, hi)
```

#### Mistake 3: Forgetting to Return at Base Case

```python
# WRONG: doesn't return for None
def solve(root):
    if not root:
        pass  # Should return!
    # ...

# RIGHT:
def solve(root):
    if not root:
        return base_case
```

---

### Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| DFS (recursive) | O(n) | O(h) where h = height |
| BFS (level order) | O(n) | O(w) where w = max width |
| BST search | O(h) | O(h) |
| Balanced BST search | O(log n) | O(log n) |

**Note:** For a balanced tree, h = log n. For a skewed tree, h = n.

---

### Practice Problems

#### Easy

| # | Problem | Sub-Pattern |
|---|---------|-------------|
| 1 | [104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Divide & Conquer |
| 2 | [100. Same Tree](https://leetcode.com/problems/same-tree/) | Divide & Conquer |
| 3 | [101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/) | Divide & Conquer |
| 4 | [226. Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) | Divide & Conquer |
| 5 | [572. Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) | Divide & Conquer |
| 6 | [94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/) | Traversal |

#### Medium

| # | Problem | Sub-Pattern |
|---|---------|-------------|
| 7 | [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | BFS |
| 8 | [98. Validate BST](https://leetcode.com/problems/validate-binary-search-tree/) | BST |
| 9 | [236. Lowest Common Ancestor](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) | Divide & Conquer |
| 10 | [230. Kth Smallest in BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | BST + Inorder |
| 11 | [114. Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/) | Postorder |
| 12 | [437. Path Sum III](https://leetcode.com/problems/path-sum-iii/) | Prefix Sum + DFS |
| 13 | [110. Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) | Divide & Conquer |
| 14 | [543. Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | Divide & Conquer |
| 15 | [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) | BFS |

#### Hard

| # | Problem | Sub-Pattern |
|---|---------|-------------|
| 16 | [124. Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Divide & Conquer |
| 17 | [297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Traversal |
| 18 | [105. Construct from Preorder and Inorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | Divide & Conquer |
| 19 | [106. Construct from Inorder and Postorder](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/) | Divide & Conquer |

---

### Interview Tips

1. **Start with recursion.** Almost every tree problem has a clean recursive solution.

2. **Define the return value clearly.** "This function returns [the depth / whether it's balanced / the sum]."

3. **Handle the base case first.** "If the node is None, return [0 / True / False]."

4. **For BST problems, use the sorted property.** "Since it's a BST, I know everything in the left subtree is smaller."

5. **Edge cases:**
   - Empty tree
   - Single node
   - Skewed tree (all left or all right)
   - Duplicate values (in BST problems)

---

*Next: [09 — DFS and BFS](09-DFS-and-BFS.md)*
