# 02 — Two Pointers  

## The Pattern That Eliminates Nested Loops

---

### What It Is

Two Pointers is a technique where you use two indices (pointers) to traverse a data structure—usually an array or linked list—in a coordinated way.

Instead of checking every pair of elements with nested loops (O(n²)), two pointers let you solve the problem in a single pass (O(n)) by moving the pointers intelligently based on the problem's constraints.

**The key insight:** If the array is sorted (or has some monotonic property), you can make decisions about which pointer to move based on the current state, eliminating the need to check all pairs.

---

### When to Use It

**The problem involves:**
- Finding pairs or triplets in an array
- Processing elements from both ends toward the middle
- Comparing elements at different positions
- The array is **sorted** (or can be sorted)

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "pair" | Two Sum, Three Sum |
| "sorted array" | Merge sorted arrays |
| "target sum" | Find pair with target sum |
| "palindrome" | Valid palindrome |
| "reverse" | Reverse words in a string |
| "partition" | Dutch National Flag |
| "closest" | 3Sum Closest |
| "remove duplicates" | Remove duplicates from sorted array |
| "container" | Container with most water |
| "merge" | Merge two sorted arrays |

**The Decision:**

```
Is the problem about pairs or combinations?
    ↓
Yes
    ↓
Is the array sorted (or can you sort it)?
    ↓
Yes
    ↓
TWO POINTERS
```

---

### The Four Sub-Types

Two Pointers isn't one pattern—it's four. Each has a different pointer movement strategy.

```
Two Pointers
├── Type 1: Opposite Ends (left=0, right=n-1, converge)
├── Type 2: Same Direction (both start at 0, move right)
├── Type 3: Fast & Slow (different speeds for cycle detection)
└── Type 4: Partition (Dutch National Flag)
```

---

### Type 1: Opposite Ends

**When:** Array is sorted, looking for pairs with a target sum.

**Movement:** Start at both ends, move inward based on comparison.

```python
left, right = 0, len(nums) - 1
while left < right:
    current_sum = nums[left] + nums[right]
    if current_sum == target:
        found_answer()
    elif current_sum < target:
        left += 1   # need bigger sum
    else:
        right -= 1  # need smaller sum
```

**Why it works:** If the sum is too small, moving `right` leftward makes it even smaller. Moving `left` rightward increases it. The sorted property guarantees this direction is correct.

**Visual:**

```
Sorted array: [1, 3, 5, 7, 9, 11]    Target: 12

Step 1: [1,  3,  5,  7,  9, 11]
         L                    R     sum=12 ✓ Found!

Step 2: If target were 14:
         1   3, [5,  7,  9, 11]
                 L            R     sum=16 > 14, move R
         1,  3, [5,  7,  9] 11
                 L        R        sum=14 ✓ Found!
```

---

### Type 2: Same Direction (Slow-Fast for Merging/Copying)

**When:** Processing two arrays together, or modifying array in-place.

**Movement:** Both pointers move left to right at different speeds or in different arrays.

```python
# Example: Remove duplicates from sorted array
def removeDuplicates(nums):
    if not nums:
        return 0
    
    write = 0  # slow pointer (write position)
    for read in range(1, len(nums)):  # fast pointer
        if nums[read] != nums[write]:
            write += 1
            nums[write] = nums[read]
    
    return write + 1
```

**Visual:**

```
nums = [1, 1, 2, 2, 3]

read=0: [1, 1, 2, 2, 3]    write=0
         W  R

read=1: [1, 1, 2, 2, 3]    nums[1]==nums[0], skip
         W     R

read=2: [1, 2, 2, 2, 3]    nums[2]!=nums[0], write++, copy
            W     R

read=3: [1, 2, 2, 2, 3]    nums[3]==nums[1], skip
            W        R

read=4: [1, 2, 3, 2, 3]    nums[4]!=nums[1], write++, copy
               W        R

Result: [1, 2, 3, ...]     length = write + 1 = 3
```

---

### Type 3: Fast & Slow (Cycle Detection)

**When:** Detecting cycles in linked lists or sequences.

**Movement:** Slow pointer moves 1 step, fast pointer moves 2 steps.

```python
def hasCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

**Why it works:** If there's a cycle, the fast pointer will eventually "lap" the slow pointer, like two runners on a circular track.

**Finding the cycle start (Floyd's Algorithm):**

```python
def detectCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            # Found meeting point. Reset one pointer to head.
            slow = head
            while slow != fast:
                slow = slow.next
                fast = fast.next
            return slow  # Cycle start
    return None
```

---

### Type 4: Partition (Dutch National Flag)

**When:** Partitioning array into three regions.

**Movement:** Three pointers—low, mid, high.

```python
def sortColors(nums):  # Dutch National Flag
    low, mid, high = 0, 0, len(nums) - 1
    
    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:  # nums[mid] == 2
            nums[mid], nums[high] = nums[high], nums[mid]
            high -= 1
```

**Visual:**

```
nums = [2, 0, 2, 1, 1, 0]

[2, 0, 2, 1, 1, 0]
 L
 M                 H

M sees 2, swap with H: [0, 0, 2, 1, 1, 2]
L                     M            H

M sees 0, swap with L: [0, 0, 2, 1, 1, 2]
   L  M                    H

M sees 0, swap with L: [0, 0, 2, 1, 1, 2]
      L  M                 H

M sees 2, swap with H: [0, 0, 1, 1, 2, 2]
      L  M           H

M sees 1, just move M: [0, 0, 1, 1, 2, 2]
      L     M      H

M sees 1, just move M: [0, 0, 1, 1, 2, 2]
      L        M   H

M > H, done!

Result: [0, 0, 1, 1, 2, 2]
```

---

### The 80% Template: Opposite Ends

```python
def two_pointer_sorted(nums, target):
    left, right = 0, len(nums) - 1
    
    while left < right:
        current = nums[left] + nums[right]  # or other operation
        
        if current == target:
            return [left, right]  # or collect answer
        elif current < target:
            left += 1   # need larger
        else:
            right -= 1  # need smaller
    
    return []  # no answer found
```

### The 80% Template: Same Direction

```python
def two_pointer_same_direction(nums):
    slow = 0
    
    for fast in range(len(nums)):
        if condition(nums[fast]):
            nums[slow] = nums[fast]
            slow += 1
    
    return slow  # new length or position
```

---

### Dry Run: Two Sum (Sorted Array)

**Problem:** Given a sorted array and a target, find two numbers that add up to target.

**Input:** `nums = [2, 7, 11, 15]`, `target = 9`

**Recognition:** "pair" + "target sum" + "sorted" → Two Pointers (Opposite Ends)

```python
def twoSum(nums, target):
    left, right = 0, len(nums) - 1
    
    while left < right:
        current = nums[left] + nums[right]
        if current == target:
            return [left, right]
        elif current < target:
            left += 1
        else:
            right -= 1
    
    return []
```

**Step-by-step:**

```
nums = [2, 7, 11, 15]    target = 9

left=0, right=3:  2 + 15 = 17 > 9  → move right leftward
left=0, right=2:  2 + 11 = 13 > 9  → move right leftward
left=0, right=1:  2 + 7  =  9 = 9  → FOUND!

Return [0, 1]
```

---

### Dry Run: Container With Most Water

**Problem:** Given n non-negative integers representing heights, find two lines that together with the x-axis form a container that holds the most water.

**Input:** `height = [1, 8, 6, 2, 5, 4, 8, 3, 7]`

**Recognition:** "container" + "most water" → Two Pointers (Opposite Ends)

**Why:** We want to maximize area = width × min(height[left], height[right]). Starting from maximum width and narrowing.

```python
def maxArea(height):
    left, right = 0, len(height) - 1
    max_water = 0
    
    while left < right:
        width = right - left
        h = min(height[left], height[right])
        max_water = max(max_water, width * h)
        
        # Move the pointer pointing to the shorter line
        # (moving the taller one can never increase area)
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_water
```

**Step-by-step:**

```
height = [1, 8, 6, 2, 5, 4, 8, 3, 7]

left=0, right=8: area = 8 * min(1,7) = 8*1 = 8    max=8
  height[0]=1 < height[8]=7, move left

left=1, right=8: area = 7 * min(8,7) = 7*7 = 49   max=49
  height[1]=8 > height[8]=7, move right

left=1, right=7: area = 6 * min(8,3) = 6*3 = 18   max=49
  height[1]=8 > height[7]=3, move right

left=1, right=6: area = 5 * min(8,8) = 5*8 = 40   max=49
  height[1]=8 = height[6]=8, move either (move right)

left=1, right=5: area = 4 * min(8,4) = 4*4 = 16   max=49
  height[1]=8 > height[5]=4, move right

left=1, right=4: area = 3 * min(8,5) = 3*5 = 15   max=49
  height[1]=8 > height[4]=5, move right

left=1, right=3: area = 2 * min(8,2) = 2*2 = 4    max=49
  height[1]=8 > height[3]=2, move right

left=1, right=2: area = 1 * min(8,6) = 1*6 = 6    max=49
  left >= right, done!

Return 49
```

---

### Dry Run: Three Sum

**Problem:** Find all unique triplets that sum to zero.

**Input:** `nums = [-1, 0, 1, 2, -1, -4]`

**Recognition:** "triplets" + "target sum" → Two Pointers (after sorting)

```python
def threeSum(nums):
    nums.sort()
    result = []
    
    for i in range(len(nums) - 2):
        # Skip duplicates for the first element
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        
        # Two pointer on the rest
        left, right = i + 1, len(nums) - 1
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                # Skip duplicates
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                left += 1
                right -= 1
            elif total < 0:
                left += 1
            else:
                right -= 1
    
    return result
```

**Key insight:** Three Sum reduces to Two Sum after fixing one element. For each `nums[i]`, find pairs that sum to `-nums[i]`.

---

### Code Templates (4 Languages)

#### Python

```python
# Opposite Ends
def opposite_ends(nums, target):
    left, right = 0, len(nums) - 1
    while left < right:
        curr = nums[left] + nums[right]
        if curr == target:
            return [left, right]
        elif curr < target:
            left += 1
        else:
            right -= 1
    return []

# Same Direction (Slow-Fast)
def same_direction(nums):
    slow = 0
    for fast in range(len(nums)):
        if should_keep(nums[fast]):
            nums[slow] = nums[fast]
            slow += 1
    return slow

# Cycle Detection
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

#### Java

```java
// Opposite Ends
public int[] twoSum(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return new int[]{left, right};
        else if (sum < target) left++;
        else right--;
    }
    return new int[]{};
}

// Same Direction
public int removeDuplicates(int[] nums) {
    int slow = 0;
    for (int fast = 0; fast < nums.length; fast++) {
        if (fast == 0 || nums[fast] != nums[fast - 1]) {
            nums[slow++] = nums[fast];
        }
    }
    return slow;
}
```

#### C++

```cpp
// Opposite Ends
vector<int> twoSum(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return {left, right};
        else if (sum < target) left++;
        else right--;
    }
    return {};
}

// Cycle Detection
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

#### JavaScript

```javascript
// Opposite Ends
function twoSum(nums, target) {
    let left = 0, right = nums.length - 1;
    while (left < right) {
        const sum = nums[left] + nums[right];
        if (sum === target) return [left, right];
        else if (sum < target) left++;
        else right--;
    }
    return [];
}

// Dutch National Flag
function sortColors(nums) {
    let low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        if (nums[mid] === 0) {
            [nums[low], nums[mid]] = [nums[mid], nums[low]];
            low++; mid++;
        } else if (nums[mid] === 1) {
            mid++;
        } else {
            [nums[mid], nums[high]] = [nums[high], nums[mid]];
            high--;
        }
    }
}
```

---

### Common Mistakes

#### Mistake 1: Forgetting to Sort

```python
# WRONG: array isn't sorted, two pointers won't work
def twoSum(nums, target):
    left, right = 0, len(nums) - 1
    # This won't find the answer reliably!

# RIGHT: sort first (if indices aren't needed)
nums.sort()
```

**Note:** If the problem asks for original indices, don't sort—use a hash map instead.

#### Mistake 2: Infinite Loop

```python
# WRONG: forgot to move pointers
while left < right:
    if nums[left] + nums[right] == target:
        return [left, right]
    # Missing: left += 1 or right -= 1

# RIGHT: always move at least one pointer in each iteration
while left < right:
    if nums[left] + nums[right] == target:
        return [left, right]
    elif nums[left] + nums[right] < target:
        left += 1
    else:
        right -= 1
```

#### Mistake 3: Skipping Duplicates in Three Sum

```python
# WRONG: might produce duplicate triplets
for i in range(len(nums)):
    # ... find pairs ...

# RIGHT: skip duplicate values for the fixed element
for i in range(len(nums)):
    if i > 0 and nums[i] == nums[i - 1]:
        continue
```

#### Mistake 4: Wrong Pointer for Cycle Start

```python
# WRONG: returning meeting point instead of cycle start
if slow == fast:
    return slow  # This is the meeting point, not cycle start

# RIGHT: reset one pointer to head and find meeting point
if slow == fast:
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next
    return slow  # Now this is the cycle start
```

---

### Complexity Analysis

| Sub-Type | Time | Space |
|----------|------|-------|
| Opposite Ends | O(n) | O(1) |
| Same Direction | O(n) | O(1) |
| Fast & Slow | O(n) | O(1) |
| Partition | O(n) | O(1) |

**Why O(n):** Each pointer traverses the array at most once.

---

### Practice Problems

#### Easy

| # | Problem | Sub-Type |
|---|---------|----------|
| 1 | [167. Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Opposite Ends |
| 2 | [344. Reverse String](https://leetcode.com/problems/reverse-string/) | Opposite Ends |
| 3 | [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Opposite Ends |
| 4 | [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Same Direction |
| 5 | [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/) | Same Direction |
| 6 | [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Fast & Slow |

#### Medium

| # | Problem | Sub-Type |
|---|---------|----------|
| 7 | [15. 3Sum](https://leetcode.com/problems/3sum/) | Opposite Ends + Sort |
| 8 | [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Opposite Ends |
| 9 | [16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/) | Opposite Ends + Sort |
| 10 | [75. Sort Colors](https://leetcode.com/problems/sort-colors/) | Partition (Dutch Flag) |
| 11 | [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Fast & Slow |
| 12 | [881. Boats to Save People](https://leetcode.com/problems/boats-to-save-people/) | Opposite Ends |
| 13 | [977. Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/) | Opposite Ends |

#### Hard

| # | Problem | Sub-Type |
|---|---------|----------|
| 14 | [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Opposite Ends |
| 15 | [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Same Direction (hybrid with Sliding Window) |
| 16 | [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Fast & Slow (cycle detection on array) |

---

### Recognition Exercises

**Exercise 1:** "Given a sorted array, find all pairs that sum to a target value."

<details>
<summary>Answer</summary>

Sub-Type: Opposite Ends
- Sorted array → two pointers from both ends
- If sum < target, move left rightward
- If sum > target, move right leftward

</details>

**Exercise 2:** "Remove all instances of a value from an array in-place."

<details>
<summary>Answer</summary>

Sub-Type: Same Direction
- Slow pointer tracks write position
- Fast pointer scans for elements to keep
- Copy non-target elements to slow position

</details>

**Exercise 3:** "Determine if a linked list has a cycle."

<details>
<summary>Answer</summary>

Sub-Type: Fast & Slow
- Slow moves 1 step, fast moves 2 steps
- If they meet, there's a cycle
- To find start: reset one to head, move both at speed 1

</details>

---

### Related Patterns

| Pattern | When to Use Instead |
|---------|---------------------|
| Sliding Window | When you need contiguous subarrays with a condition |
| Binary Search | When you're searching for a specific value in sorted data |
| HashMap | When you need O(1) lookup and don't need sorted order |
| Prefix Sum | When you need cumulative sums |

---

### Interview Tips

1. **Ask about sorting.** "Can I sort the array?" If yes and indices don't matter, two pointers becomes viable.

2. **Explain why you move each pointer.** "I move the left pointer because increasing it increases the sum, which is what I need."

3. **Handle duplicates explicitly.** In problems like 3Sum, explain how you skip duplicates to avoid duplicate triplets.

4. **Mention the time savings.** "This is O(n) instead of O(n²) because each pointer makes at most one pass."

5. **Test edge cases:**
   - Empty array
   - Single element
   - All elements the same
   - No valid pair exists
   - Multiple valid pairs

6. **If stuck, ask:** "Is the array sorted? If not, can I sort it? If I can, can I use two pointers from both ends?"

---

*Next: [03 — Binary Search](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/DSA%20Handbook/Binary-search.md)*
