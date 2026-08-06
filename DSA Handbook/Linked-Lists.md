
# 07 — Linked Lists

## The Pattern That Masters Pointer Manipulation

---

### What It Is

Linked Lists are data structures where each node contains a value and a pointer to the next node. The patterns in this chapter teach you how to manipulate pointers efficiently to reverse, detect cycles, merge, and partition linked lists.

**The key insight:** Most linked list problems can be solved with a few pointer manipulation templates. The difficulty isn't the algorithm—it's getting the pointer assignments in the right order.

---

### When to Use It

**The problem involves:**
- A singly or doubly linked list
- Reversing part or all of a list
- Detecting cycles
- Finding the middle element
- Merging two sorted lists
- Removing nodes based on conditions

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "reverse" | Reverse Linked List |
| "cycle" | Linked List Cycle |
| "middle" | Middle of the Linked List |
| "merge" | Merge Two Sorted Lists |
| "remove" | Remove Nth Node From End |
| "palindrome" | Palindrome Linked List |
| "intersection" | Intersection of Two Linked Lists |
| "rotate" | Rotate List |
| "sort" | Sort List |
| "swap" | Swap Nodes in Pairs |

---

### The Three Core Patterns

#### Pattern 1: Reversal

Reverse the direction of pointers.

```python
def reverse_list(head):
    prev = None
    curr = head
    while curr:
        next_node = curr.next  # Save next
        curr.next = prev       # Reverse pointer
        prev = curr            # Move prev forward
        curr = next_node       # Move curr forward
    return prev  # New head
```

```c++
// Reverse the direction of pointers
ListNode* reverse_list(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr != nullptr) {
        ListNode* nextNode = curr->next; // Save next
        curr->next = prev;               // Reverse pointer
        prev = curr;                     // Move prev forward
        curr = nextNode;                 // Move curr forward
    }
    return prev; // New head
}
```

**Visual:**

```
Original: 1 → 2 → 3 → 4 → None

Step 1: None ← 1   2 → 3 → 4 → None
        prev  curr

Step 2: None ← 1 ← 2   3 → 4 → None
              prev curr

Step 3: None ← 1 ← 2 ← 3   4 → None
                   prev curr

Step 4: None ← 1 ← 2 ← 3 ← 4
                        prev curr

Return prev (4) as new head.
```

#### Pattern 2: Fast & Slow Pointers

Use two pointers moving at different speeds.

```python
def find_middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow  # Middle node
```

```c++
ListNode* find_middle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow; // Middle node
}
```

**Visual:**

```
1 → 2 → 3 → 4 → 5

slow=1, fast=1
slow=2, fast=3
slow=3, fast=5  → fast.next is None, return slow (3)
```

#### Pattern 3: Dummy Head

Use a dummy node to simplify edge cases (especially when the head might change).

```python
def remove_elements(head, val):
    dummy = ListNode(0)
    dummy.next = head
    prev, curr = dummy, head
    
    while curr:
        if curr.val == val:
            prev.next = curr.next  # Skip current node
        else:
            prev = curr
        curr = curr.next
    
    return dummy.next  # Original head might have been removed
```

```c++
ListNode* remove_elements(ListNode* head, int val) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* prev = dummy;
    ListNode* curr = head;
    
    while (curr != nullptr) {
        if (curr->val == val) {
            prev->next = curr->next; // Skip current node
        } else {
            prev = curr;
        }
        curr = curr->next;
    }
    
    ListNode* newHead = dummy->next;
    delete dummy;
    return newHead;
}
```

**Visual for Dummy Head (removing all nodes with value 1):**

```
Before:
  head → 1 → 2 → 3 → 1 → 4 → None

After adding dummy:
  dummy → 1 → 2 → 3 → 1 → 4 → None
  prev    curr

When curr.val == 1:
  dummy → 2 → 3 → 1 → 4 → None  (skip the first 1)
  prev    curr

... after processing:
  dummy → 2 → 3 → 4 → None

Return dummy.next → 2 → 3 → 4 → None
```

---

### The 80% Templates

#### Reverse a Linked List

```python
def reverse(head):
    prev = None
    curr = head
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    return prev
```

```c++
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr) {
        ListNode* nxt = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nxt;
    }
    return prev;
}
```

#### Reverse Between Positions (m to n)

```python
def reverseBetween(head, m, n):
    dummy = ListNode(0)
    dummy.next = head
    prev = dummy
    
    # Move to position m
    for _ in range(m - 1):
        prev = prev.next
    
    # Reverse from m to n
    curr = prev.next
    for _ in range(n - m):
        nxt = curr.next
        curr.next = nxt.next
        nxt.next = prev.next
        prev.next = nxt
    
    return dummy.next
```

```c++
ListNode* reverseBetween(ListNode* head, int m, int n) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* prev = dummy;
    
    // Move to position m
    for (int i = 0; i < m - 1; i++) {
        prev = prev->next;
    }
    
    // Reverse from m to n
    ListNode* curr = prev->next;
    for (int i = 0; i < n - m; i++) {
        ListNode* nxt = curr->next;
        curr->next = nxt->next;
        nxt->next = prev->next;
        prev->next = nxt;
    }
    
    ListNode* newHead = dummy->next;
    delete dummy;
    return newHead;
}
```

**Visual for reverseBetween:**

```
Original: 1 → 2 → 3 → 4 → 5    m=2, n=4

After moving prev to position 1:
  1 → 2 → 3 → 4 → 5
  prev  curr

Step 1: Move 3 after prev:
  1 → 3 → 2 → 4 → 5
  prev

Step 2: Move 4 after prev:
  1 → 4 → 3 → 2 → 5
  prev

Result: 1 → 4 → 3 → 2 → 5
```

#### Detect Cycle and Find Start

```python
def detectCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            # Meeting point found
            slow = head
            while slow != fast:
                slow = slow.next
                fast = fast.next
            return slow  # Cycle start
    return None
```

```c++
ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            // Meeting point found
            slow = head;
            while (slow != fast) {
                slow = slow->next;
                fast = fast->next;
            }
            return slow; // Cycle start
        }
    }
    return nullptr;
}
```

**Why this works:**
- Let `a` = distance from head to cycle start
- Let `b` = distance from cycle start to meeting point
- Let `c` = cycle length
- When they meet: slow traveled `a + b`, fast traveled `a + b + n*c`
- Since fast travels 2x speed: `2(a + b) = a + b + n*c` → `a + b = n*c` → `a = n*c - b`
- So moving `a` steps from both head and meeting point reaches cycle start

**Visual for Cycle Detection:**

```
Given: 1 → 2 → 3 → 4 → 5 → 6 → 7
                    ↑         ↓
                    9 ← 8 ← ← ←

Step 1: slow=1, fast=1
Step 2: slow=2, fast=3
Step 3: slow=3, fast=5
Step 4: slow=4, fast=7
Step 5: slow=5, fast=9
Step 6: slow=6, fast=8
Step 7: slow=7, fast=7 → meet at 7

Then move slow to head (1), fast stays at 7.
Move both one step at a time:
  slow=1, fast=7
  slow=2, fast=8
  slow=3, fast=9
  slow=4, fast=4 → meet at 4, which is cycle start.
```

#### Merge Two Sorted Lists

```python
def mergeTwoLists(l1, l2):
    dummy = ListNode(0)
    curr = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    
    curr.next = l1 or l2  # Attach remaining
    return dummy.next
```

```c++
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode* dummy = new ListNode(0);
    ListNode* curr = dummy;
    
    while (l1 && l2) {
        if (l1->val <= l2->val) {
            curr->next = l1;
            l1 = l1->next;
        } else {
            curr->next = l2;
            l2 = l2->next;
        }
        curr = curr->next;
    }
    
    curr->next = l1 ? l1 : l2; // Attach remaining
    ListNode* newHead = dummy->next;
    delete dummy;
    return newHead;
}
```

**Visual for Merge:**

```
l1: 1 → 3 → 5
l2: 2 → 4 → 6

dummy → null, curr=dummy
Compare 1 and 2: take 1
  dummy → 1 → null, curr=1

Compare 3 and 2: take 2
  dummy → 1 → 2 → null, curr=2

Compare 3 and 4: take 3
  dummy → 1 → 2 → 3 → null, curr=3

Compare 5 and 4: take 4
  dummy → 1 → 2 → 3 → 4 → null, curr=4

Compare 5 and 6: take 5
  dummy → 1 → 2 → 3 → 4 → 5 → null, curr=5

Attach remaining: l2=6
  dummy → 1 → 2 → 3 → 4 → 5 → 6 → null

Result: 1 → 2 → 3 → 4 → 5 → 6
```

---

### Dry Run: Reverse Linked List

**Problem:** Reverse a singly linked list.

**Input:** `1 → 2 → 3 → 4 → 5 → None`

```python
def reverseList(head):
    prev = None
    curr = head
    
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    
    return prev
```

```c++
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    
    while (curr) {
        ListNode* nxt = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nxt;
    }
    
    return prev;
}
```

**Step-by-step:**

```
Initial: prev=None, curr=1

Iteration 1:
  nxt = 2
  1.next = None
  prev = 1, curr = 2
  State: None ← 1   2 → 3 → 4 → 5

Iteration 2:
  nxt = 3
  2.next = 1
  prev = 2, curr = 3
  State: None ← 1 ← 2   3 → 4 → 5

Iteration 3:
  nxt = 4
  3.next = 2
  prev = 3, curr = 4
  State: None ← 1 ← 2 ← 3   4 → 5

Iteration 4:
  nxt = 5
  4.next = 3
  prev = 4, curr = 5
  State: None ← 1 ← 2 ← 3 ← 4   5

Iteration 5:
  nxt = None
  5.next = 4
  prev = 5, curr = None
  State: None ← 1 ← 2 ← 3 ← 4 ← 5

Return prev = 5 (new head)
Result: 5 → 4 → 3 → 2 → 1 → None
```

---

### Dry Run: Remove Nth Node From End

**Problem:** Remove the Nth node from the end of the list.

**Input:** `1 → 2 → 3 → 4 → 5`, `n = 2`

**Recognition:** "remove" + "from end" → Two pointers with gap

```python
def removeNthFromEnd(head, n):
    dummy = ListNode(0)
    dummy.next = head
    fast = slow = dummy
    
    # Move fast n+1 steps ahead
    for _ in range(n + 1):
        fast = fast.next
    
    # Move both until fast reaches end
    while fast:
        fast = fast.next
        slow = slow.next
    
    # Remove the nth from end
    slow.next = slow.next.next
    
    return dummy.next
```

```c++
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* fast = dummy;
    ListNode* slow = dummy;
    
    // Move fast n+1 steps ahead
    for (int i = 0; i <= n; i++) {
        fast = fast->next;
    }
    
    // Move both until fast reaches end
    while (fast) {
        fast = fast->next;
        slow = slow->next;
    }
    
    // Remove the nth from end
    slow->next = slow->next->next;
    
    ListNode* newHead = dummy->next;
    delete dummy;
    return newHead;
}
```

**Step-by-step:**

```
head = 1 → 2 → 3 → 4 → 5    n = 2

dummy → 1 → 2 → 3 → 4 → 5

Move fast 3 steps (n+1):
  fast at node 3 (1→2→3)

Move both:
  slow at dummy, fast at 3
  slow at 1, fast at 4
  slow at 2, fast at 5
  fast.next is None, stop

slow is at node 2. Remove node 3 (slow.next):
  2.next = 4

Result: 1 → 2 → 4 → 5
```

---

### Additional Useful Templates

#### Get Length of Linked List

```python
def get_length(head):
    length = 0
    curr = head
    while curr:
        length += 1
        curr = curr.next
    return length
```

```c++
int get_length(ListNode* head) {
    int length = 0;
    ListNode* curr = head;
    while (curr) {
        length++;
        curr = curr->next;
    }
    return length;
}
```

#### Find Kth Node from End (using two pointers)

```python
def find_kth_from_end(head, k):
    fast = slow = head
    for _ in range(k):
        fast = fast.next
    while fast:
        fast = fast.next
        slow = slow.next
    return slow  # kth from end (1-indexed)
```

```c++
ListNode* find_kth_from_end(ListNode* head, int k) {
    ListNode* fast = head;
    ListNode* slow = head;
    for (int i = 0; i < k; i++) {
        fast = fast->next;
    }
    while (fast) {
        fast = fast->next;
        slow = slow->next;
    }
    return slow;
}
```

#### Partition List (Odd-Even)

```python
def odd_even_list(head):
    if not head:
        return head
    odd = head
    even = head.next
    even_head = even
    while even and even.next:
        odd.next = even.next
        odd = odd.next
        even.next = odd.next
        even = even.next
    odd.next = even_head
    return head
```

```c++
ListNode* odd_even_list(ListNode* head) {
    if (!head) return head;
    ListNode* odd = head;
    ListNode* even = head->next;
    ListNode* even_head = even;
    while (even && even->next) {
        odd->next = even->next;
        odd = odd->next;
        even->next = odd->next;
        even = even->next;
    }
    odd->next = even_head;
    return head;
}
```

#### Rotate List (Right by k)

```python
def rotate_right(head, k):
    if not head or not head.next:
        return head
    # Get length
    n = 1
    tail = head
    while tail.next:
        tail = tail.next
        n += 1
    k %= n
    if k == 0:
        return head
    # Find the new tail: n - k - 1
    new_tail = head
    for _ in range(n - k - 1):
        new_tail = new_tail.next
    new_head = new_tail.next
    new_tail.next = None
    tail.next = head
    return new_head
```

```c++
ListNode* rotate_right(ListNode* head, int k) {
    if (!head || !head->next) return head;
    // Get length
    int n = 1;
    ListNode* tail = head;
    while (tail->next) {
        tail = tail->next;
        n++;
    }
    k %= n;
    if (k == 0) return head;
    // Find the new tail: n - k - 1
    ListNode* new_tail = head;
    for (int i = 0; i < n - k - 1; i++) {
        new_tail = new_tail->next;
    }
    ListNode* new_head = new_tail->next;
    new_tail->next = nullptr;
    tail->next = head;
    return new_head;
}
```

#### Check Palindrome

```python
def is_palindrome(head):
    if not head or not head.next:
        return True
    # Find middle (left)
    slow = fast = head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    # Reverse second half
    second = reverse(slow.next)
    # Compare
    p1, p2 = head, second
    while p2:
        if p1.val != p2.val:
            return False
        p1 = p1.next
        p2 = p2.next
    return True
```

```c++
bool is_palindrome(ListNode* head) {
    if (!head || !head->next) return true;
    // Find middle (left)
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    // Reverse second half
    ListNode* second = reverse(slow->next);
    // Compare
    ListNode* p1 = head;
    ListNode* p2 = second;
    while (p2) {
        if (p1->val != p2->val) return false;
        p1 = p1->next;
        p2 = p2->next;
    }
    return true;
}
```

---

### Common Patterns Summary

| Pattern | Technique | Use Case |
|---------|-----------|----------|
| Reverse | Three pointers (prev, curr, next) | Reverse all or part |
| Find Middle | Fast & slow pointers | Middle node |
| Detect Cycle | Fast & slow pointers | Cycle detection |
| Remove Node | Dummy head + prev/curr | Conditional removal |
| Merge | Dummy head + compare | Merge sorted lists |
| Partition | Two dummy heads or two pointers | Partition by value or odd/even |
| Rotate | Find kth from end, break and reconnect | Rotate list |
| Palindrome | Find middle, reverse second half, compare | Check palindrome |

---

### Code Templates (4 Languages)

#### Python

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Reverse
def reverse(head):
    prev, curr = None, head
    while curr:
        nxt = curr.next
        curr.next = prev
        prev, curr = curr, nxt
    return prev

# Find Middle
def middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow

# Detect Cycle
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False

# Merge Two Sorted
def merge(l1, l2):
    dummy = ListNode(0)
    curr = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    curr.next = l1 or l2
    return dummy.next
```

#### Java

```java
// Reverse
ListNode reverse(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}

// Find Middle
ListNode middle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

#### C++

```cpp
// Reverse
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr) {
        ListNode* next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}

// Find Middle
ListNode* middle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```

#### JavaScript

```javascript
// Reverse
function reverse(head) {
    let prev = null, curr = head;
    while (curr) {
        const next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}

// Find Middle
function middle(head) {
    let slow = head, fast = head;
    while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

---

### Common Mistakes

#### Mistake 1: Losing Reference to Next Node

```python
# WRONG: can't traverse after reversing
curr.next = prev
curr = curr.next  # This moves to prev, not the original next!

# RIGHT: save next before modifying
nxt = curr.next
curr.next = prev
prev = curr
curr = nxt
```

```c++
// WRONG: can't traverse after reversing
curr->next = prev;
curr = curr->next;  // This moves to prev, not the original next!

// RIGHT: save next before modifying
ListNode* nxt = curr->next;
curr->next = prev;
prev = curr;
curr = nxt;
```

#### Mistake 2: Not Using Dummy Head

```python
# WRONG: head might be removed, can't return it
def remove(head, val):
    while head and head.val == val:
        head = head.next
    # ... rest of logic

# RIGHT: use dummy node
def remove(head, val):
    dummy = ListNode(0)
    dummy.next = head
    # ... logic using dummy
    return dummy.next
```

```c++
// WRONG: head might be removed, can't return it
ListNode* remove(ListNode* head, int val) {
    while (head && head->val == val) {
        head = head->next;
    }
    // ... rest of logic
}

// RIGHT: use dummy node
ListNode* remove(ListNode* head, int val) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    // ... logic using dummy
    ListNode* newHead = dummy->next;
    delete dummy;
    return newHead;
}
```

#### Mistake 3: Off-by-One in Fast/Slow

```python
# For even-length lists, there are two middles
# Decide which one you want:

# Left middle (for palindrome split):
slow = fast = head
while fast.next and fast.next.next:
    slow = slow.next
    fast = fast.next.next

# Right middle:
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

```c++
// For even-length lists, there are two middles
// Decide which one you want:

// Left middle (for palindrome split):
ListNode* slow = head;
ListNode* fast = head;
while (fast->next && fast->next->next) {
    slow = slow->next;
    fast = fast->next->next;
}

// Right middle:
ListNode* slow = head;
ListNode* fast = head;
while (fast && fast->next) {
    slow = slow->next;
    fast = fast->next->next;
}
```

---

### Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Reverse | O(n) | O(1) |
| Find Middle | O(n) | O(1) |
| Detect Cycle | O(n) | O(1) |
| Merge Two Sorted | O(n + m) | O(1) |
| Remove Nth From End | O(n) | O(1) |

---

### Practice Problems

#### Easy

| # | Problem | Pattern |
|---|---------|---------|
| 1 | [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Reversal |
| 2 | [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) | Fast & Slow |
| 3 | [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Cycle Detection |
| 4 | [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | Merge |
| 5 | [83. Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) | Conditional Remove |

#### Medium

| # | Problem | Pattern |
|---|---------|---------|
| 6 | [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/) | Partial Reversal |
| 7 | [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Find Cycle Start |
| 8 | [19. Remove Nth Node From End](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | Two Pointers Gap |
| 9 | [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) | Reverse + Compare |
| 10 | [328. Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/) | Partition |
| 11 | [143. Reorder List](https://leetcode.com/problems/reorder-list/) | Find Middle + Reverse + Merge |
| 12 | [61. Rotate List](https://leetcode.com/problems/rotate-list/) | Find Kth + Break |

#### Hard

| # | Problem | Pattern |
|---|---------|---------|
| 13 | [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Heap + Merge |
| 14 | [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | Group Reversal |
| 15 | [138. Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) | Hash Map + Clone |

---

### Interview Tips

1. **Draw the pointers.** Always draw before and after states when manipulating pointers.

2. **Use a dummy head.** It simplifies edge cases when the head might change.

3. **Save the next pointer.** Before modifying any node's `next`, save it.

4. **Test with small examples.** Use lists of length 1, 2, and 3 to verify.

5. **Edge cases:**
   - Empty list
   - Single node
   - Two nodes
   - Cycle in the list
   - Removing head or tail

---
