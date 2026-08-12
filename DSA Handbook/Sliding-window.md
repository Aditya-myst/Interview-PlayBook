# 01 — Sliding Window

## The Pattern That Turns O(n²) Into O(n)

---

### What It Is

Sliding Window is a technique for efficiently processing **contiguous sequences** (subarrays or substrings) in arrays and strings.

Instead of examining every possible subarray (which takes O(n²) time), you maintain a "window" that slides across the data, adding elements on one side and removing them from the other.

**The key insight:** When you move the window one step to the right, you don't need to recompute everything from scratch. You just:
- Add the new element entering the window
- Remove the old element leaving the window

This turns a brute-force O(n²) into an elegant O(n).

---

### When to Use It

**The problem involves:**
- A contiguous subarray or substring
- Finding the longest, shortest, maximum, or minimum of something
- A constraint on the window contents (e.g., "at most K distinct elements")

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "contiguous subarray" | Find the contiguous subarray with sum ≤ K |
| "substring" | Longest substring without repeating characters |
| "longest ... that ..." | Longest subarray with at most 2 distinct numbers |
| "shortest ... that ..." | Minimum window substring containing all characters |
| "maximum ... in a window" | Maximum average of any subarray of size K |
| "at most K" | Subarrays with at most K odd numbers |
| "exactly K" | Subarrays with exactly K distinct integers |
| "without repeating" | Longest substring without repeating characters |
| "contains all" | Smallest window containing all characters of another string |

**The Decision:**

```
Is the data contiguous?
    ↓
Yes
    ↓
Are you looking for longest/shortest/maximum/minimum?
    ↓
Yes
    ↓
SLIDING WINDOW
```

---

### Mental Model

Think of a window pane sliding across a row of numbers:

```
Array: [2, 1, 5, 1, 3, 2]    K = 8 (max sum)

Step 1: [2, 1, 5] 1, 3, 2     sum = 8 ✓  window size = 3
Step 2:  2,[1, 5, 1] 3, 2     sum = 7 ✓  window size = 3
Step 3:  2, 1,[5, 1, 3] 2     sum = 9 ✗  too big! shrink
Step 4:  2, 1, 5,[1, 3] 2     sum = 4 ✓  window size = 2
Step 5:  2, 1, 5, 1,[3, 2]    sum = 5 ✓  window size = 2
```

The window **expands** when the current state is valid.
The window **shrinks** when the current state becomes invalid.

This expand-shrink rhythm is the heartbeat of every sliding window problem.

---

### The Two Types of Sliding Window

#### Type 1: Fixed Size Window

The window size is given (e.g., "find max sum of subarray of size K").

```
Window size = K (given)

for right in range(len(nums)):
    add nums[right] to window
    if right >= K:
        remove nums[right - K] from window
    if window size == K:
        update answer
```

**When to recognize:** The problem explicitly states a window size.

#### Type 2: Variable Size Window

The window size changes based on a validity condition.

```
left = 0
for right in range(len(nums)):
    add nums[right] to window
    while window is invalid:
        remove nums[left] from window
        left += 1
    update answer
```

**When to recognize:** The problem asks for longest/shortest subarray satisfying some condition.

---

### The 80% Template

This template solves the vast majority of sliding window problems. The only things that change are:
1. What makes the window "invalid"
2. How you update the answer
3. Whether you're maximizing or minimizing

```python
def sliding_window(nums, k):
    left = 0
    window_state = {}  # or a counter, sum, set, etc.
    answer = 0  # or float('inf') if minimizing
    
    for right in range(len(nums)):
        # 1. Expand: add nums[right] to the window
        add_to_window(nums[right], window_state)
        
        # 2. Shrink: while window is invalid, remove from left
        while window_is_invalid(window_state, k):
            remove_from_window(nums[left], window_state)
            left += 1
        
        # 3. Update answer
        # For longest:  answer = max(answer, right - left + 1)
        # For shortest: answer = min(answer, right - left + 1)
        answer = max(answer, right - left + 1)
    
    return answer
```

**Why this works:**
- `right` always moves forward (O(n) total steps)
- `left` only moves forward (O(n) total steps)
- Total work: O(n) + O(n) = O(n)

---

### Dry Run: Longest Substring Without Repeating Characters

**Problem:** Given a string `s`, find the length of the longest substring without repeating characters.

**Input:** `s = "abcabcbb"`

**Recognition:** "longest substring" + "without repeating" → Sliding Window

**Template Application:**
- Window state: a set (or counter) of characters currently in the window
- Invalid condition: a character appears more than once
- Answer: maximize window length

```python
def lengthOfLongestSubstring(s):
    left = 0
    char_count = {}
    answer = 0
    
    for right in range(len(s)):
        # Expand
        char_count[s[right]] = char_count.get(s[right], 0) + 1
        
        # Shrink while invalid (duplicate exists)
        while char_count[s[right]] > 1:
            char_count[s[left]] -= 1
            left += 1
        
        # Update answer (longest)
        answer = max(answer, right - left + 1)
    
    return answer
```

**Step-by-step:**

```
s = "abcabcbb"

right=0: 'a'  → {a:1}           window="a"       len=1  answer=1
right=1: 'b'  → {a:1,b:1}       window="ab"      len=2  answer=2
right=2: 'c'  → {a:1,b:1,c:1}   window="abc"     len=3  answer=3
right=3: 'a'  → {a:2,b:1,c:1}   INVALID! 'a' appears twice
         shrink: remove 'a' at left=0 → {a:1,b:1,c:1}  left=1
         window="bca"  len=3  answer=3
right=4: 'b'  → {a:1,b:2,c:1}   INVALID! 'b' appears twice
         shrink: remove 'b' at left=1 → {a:1,b:1,c:1}  left=2
         window="cab"  len=3  answer=3
right=5: 'c'  → {a:1,b:1,c:2}   INVALID! 'c' appears twice
         shrink: remove 'c' at left=2 → {a:1,b:1,c:1}  left=3
         window="abc"  len=3  answer=3
right=6: 'b'  → {a:1,b:2,c:1}   INVALID! 'b' appears twice
         shrink: remove 'a' at left=3 → {a:0,b:2,c:1}  left=4
         still invalid! remove 'b' at left=4 → {a:0,b:1,c:1}  left=5
         window="cb"   len=2  answer=3
right=7: 'b'  → {a:0,b:2,c:1}   INVALID! 'b' appears twice
         shrink: remove 'c' at left=5 → {a:0,b:2,c:0}  left=6
         still invalid! remove 'b' at left=6 → {a:0,b:1,c:0}  left=7
         window="b"   len=1  answer=3

Return 3. The longest substring without repeating characters is "abc".
```

---

### Dry Run: Minimum Window Substring

**Problem:** Given strings `s` and `t`, find the minimum window in `s` that contains all characters of `t`.

**Input:** `s = "ADOBECODEBANC"`, `t = "ABC"`

**Recognition:** "minimum window" + "contains all" → Sliding Window

**Template Application:**
- Window state: counter of characters in the window
- Valid condition: window contains all characters of `t` (with required counts)
- Answer: minimize window length

```python
from collections import Counter

def minWindow(s, t):
    need = Counter(t)
    missing = len(t)  # total characters still needed
    left = 0
    best_start, best_len = 0, float('inf')
    
    for right in range(len(s)):
        # Expand
        if need[s[right]] > 0:
            missing -= 1
        need[s[right]] -= 1
        
        # When valid (all characters found), try to shrink
        while missing == 0:
            # Update answer (shortest)
            if right - left + 1 < best_len:
                best_len = right - left + 1
                best_start = left
            
            # Shrink
            need[s[left]] += 1
            if need[s[left]] > 0:
                missing += 1
            left += 1
    
    return s[best_start:best_start + best_len] if best_len != float('inf') else ""
```

**Key difference from the previous problem:**
- "Longest substring without repeating" → maximize window, shrink while invalid
- "Minimum window substring" → minimize window, shrink while valid

Same template. Different optimization direction. Different validity condition.

---

### The Expand-Shrink Pattern Visualized

```
MAXIMIZE (longest valid):
    
    expand → → → → → shrink ← ← expand → → → → → shrink ←
    [  valid window  ][invalid][   valid window   ][invalid]
    
    Update answer WHENEVER window is valid.
    Shrink to TRY to make it valid again.


MINIMIZE (shortest valid):

    expand → → → → shrink ← ← ← expand → → → → shrink ← ←
    [  too big...  ][ valid ][     too big...    ][ valid  ]
    
    Update answer ONLY when window is valid.
    Shrink to FIND a smaller valid window.
```

---

### Code Templates (4 Languages)

#### Python

```python
# Fixed Size Window
def fixed_window(nums, k):
    window_sum = 0
    for i in range(len(nums)):
        window_sum += nums[i]
        if i >= k:
            window_sum -= nums[i - k]
        if i >= k - 1:
            # process window
            pass
    return 0

# Variable Size Window
def variable_window(s):
    left = 0
    state = {}
    answer = 0
    
    for right in range(len(s)):
        # expand
        state[s[right]] = state.get(s[right], 0) + 1
        
        # shrink while invalid
        while is_invalid(state):
            state[s[left]] -= 1
            if state[s[left]] == 0:
                del state[s[left]]
            left += 1
        
        # update answer
        answer = max(answer, right - left + 1)
    
    return answer
```

#### Java

```java
// Fixed Size Window
public int fixedWindow(int[] nums, int k) {
    int windowSum = 0;
    int answer = 0;
    for (int right = 0; right < nums.length; right++) {
        windowSum += nums[right];
        if (right >= k) {
            windowSum -= nums[right - k];
        }
        if (right >= k - 1) {
            // process window
            answer = Math.max(answer, windowSum);
        }
    }
    return answer;
}

// Variable Size Window
public int variableWindow(String s) {
    Map<Character, Integer> state = new HashMap<>();
    int left = 0;
    int answer = 0;
    
    for (int right = 0; right < s.length(); right++) {
        // expand
        char c = s.charAt(right);
        state.put(c, state.getOrDefault(c, 0) + 1);
        
        // shrink while invalid
        while (isInvalid(state)) {
            char leftChar = s.charAt(left);
            state.put(leftChar, state.get(leftChar) - 1);
            if (state.get(leftChar) == 0) {
                state.remove(leftChar);
            }
            left++;
        }
        
        // update answer
        answer = Math.max(answer, right - left + 1);
    }
    return answer;
}
```

#### C++

```cpp
// Fixed Size Window
int fixedWindow(vector<int>& nums, int k) {
    int windowSum = 0;
    int answer = 0;
    for (int right = 0; right < nums.size(); right++) {
        windowSum += nums[right];
        if (right >= k) {
            windowSum -= nums[right - k];
        }
        if (right >= k - 1) {
            answer = max(answer, windowSum);
        }
    }
    return answer;
}

// Variable Size Window
int variableWindow(string s) {
    unordered_map<char, int> state;
    int left = 0;
    int answer = 0;
    
    for (int right = 0; right < s.size(); right++) {
        // expand
        state[s[right]]++;
        
        // shrink while invalid
        while (isInvalid(state)) {
            state[s[left]]--;
            if (state[s[left]] == 0) {
                state.erase(s[left]);
            }
            left++;
        }
        
        // update answer
        answer = max(answer, right - left + 1);
    }
    return answer;
}
```

#### JavaScript

```javascript
// Fixed Size Window
function fixedWindow(nums, k) {
    let windowSum = 0;
    let answer = 0;
    for (let right = 0; right < nums.length; right++) {
        windowSum += nums[right];
        if (right >= k) {
            windowSum -= nums[right - k];
        }
        if (right >= k - 1) {
            answer = Math.max(answer, windowSum);
        }
    }
    return answer;
}

// Variable Size Window
function variableWindow(s) {
    const state = new Map();
    let left = 0;
    let answer = 0;
    
    for (let right = 0; right < s.length; right++) {
        // expand
        state.set(s[right], (state.get(s[right]) || 0) + 1);
        
        // shrink while invalid
        while (isInvalid(state)) {
            state.set(s[left], state.get(s[left]) - 1);
            if (state.get(s[left]) === 0) {
                state.delete(s[left]);
            }
            left++;
        }
        
        // update answer
        answer = Math.max(answer, right - left + 1);
    }
    return answer;
}
```

---

### Variations

#### Variation 1: At Most K → Exactly K

Sometimes the problem asks for "exactly K" but the template works best with "at most K."

**Trick:** `exactly(K) = atMost(K) - atMost(K-1)`

```python
def exactlyK(nums, k):
    return atMost(nums, k) - atMost(nums, k - 1)

def atMost(nums, k):
    left = 0
    count = 0
    freq = {}
    for right in range(len(nums)):
        freq[nums[right]] = freq.get(nums[right], 0) + 1
        while len(freq) > k:
            freq[nums[left]] -= 1
            if freq[nums[left]] == 0:
                del freq[nums[left]]
            left += 1
        count += right - left + 1
    return count
```

#### Variation 2: Sliding Window with Two Arrays

Sometimes you need to find a pattern across two arrays or strings (e.g., "find all anagrams").

**Template modification:** Maintain two frequency maps—one for the target, one for the current window.

#### Variation 3: Sliding Window Maximum/Minimum

When you need the max or min element in each window position, combine sliding window with a **monotonic deque**.

This is a hybrid of Sliding Window + Monotonic Stack. See Chapter 05.

---

### Common Mistakes

#### Mistake 1: Off-by-One Errors

```python
# WRONG: doesn't process the last window
for right in range(len(nums)):
    # ... shrink logic ...
    if right - left + 1 == k:
        update_answer()

# The loop ends, but the window might still be valid.
# Make sure to update the answer INSIDE the loop.
```

#### Mistake 2: Shrinking Too Much

```python
# WRONG: shrinks even when window is valid
while left <= right:
    remove(nums[left])
    left += 1

# RIGHT: only shrink while INVALID
while window_is_invalid():
    remove(nums[left])
    left += 1
```

#### Mistake 3: Forgetting to Handle Empty State

```python
# When removing a character, don't forget to delete it
# from the map when count reaches 0
char_count[c] -= 1
if char_count[c] == 0:
    del char_count[c]  # THIS IS CRITICAL
# Otherwise len(char_count) won't reflect actual distinct chars
```

#### Mistake 4: Using Wrong Update Logic

```python
# For LONGEST: update OUTSIDE the shrink loop
answer = max(answer, right - left + 1)

# For SHORTEST: update INSIDE the shrink loop
while valid():
    answer = min(answer, right - left + 1)
    shrink()
```

#### Mistake 5: Not Considering All Elements

```python
# WRONG: stops processing when condition is met
if found:
    return answer

# RIGHT: continues to find optimal answer
if valid:
    answer = min(answer, current_length)
```

---

### Complexity Analysis

| Aspect | Complexity | Why |
|--------|-----------|-----|
| Time | O(n) | Each element is added once and removed once |
| Space | O(k) | Where k is the size of the character set or constraint value |

**Proof of O(n):**
- `right` pointer moves from 0 to n-1: n steps
- `left` pointer only moves forward: at most n steps total
- Total operations: n + n = 2n = O(n)

---

### Practice Problems

#### Easy (Template Application)

| # | Problem | Key |
|---|---------|-----|
| 1 | [643. Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) | Fixed window, sum |
| 2 | [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Variable window, set |
| 3 | [219. Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/) | Fixed window, set |
| 4 | [1456. Maximum Number of Vowels in a Substring of Given Size](https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-size/) | Fixed window, count |

#### Medium (Pattern Recognition)

| # | Problem | Key |
|---|---------|-----|
| 5 | [424. Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Variable window, max frequency |
| 6 | [567. Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Fixed window, frequency match |
| 7 | [713. Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) | Variable window, product |
| 8 | [904. Fruit Into Baskets](https://leetcode.com/problems/fruits-into-baskets/) | Variable window, at most 2 distinct |
| 9 | [1004. Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Variable window, at most K flips |
| 10 | [209. Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Variable window, minimize length |

#### Medium-Hard (Disguised Pattern)

| # | Problem | Key |
|---|---------|-----|
| 11 | [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Variable window, minimize |
| 12 | [438. Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Fixed window, frequency match |
| 13 | [395. Longest Substring with At Least K Repeating Characters](https://leetcode.com/problems/longest-substring-with-at-least-k-repeating-characters/) | Sliding window on distinct count |
| 14 | [992. Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/) | exactlyK = atMost(K) - atMost(K-1) |
| 15 | [1248. Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) | exactlyK variant |

#### Hard (Advanced)

| # | Problem | Key |
|---|---------|-----|
| 16 | [239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Sliding window + monotonic deque |
| 17 | [480. Sliding Window Median](https://leetcode.com/problems/sliding-window-median/) | Sliding window + two heaps |
| 18 | [30. Substring with Concatenation of All Words](https://leetcode.com/problems/substring-with-concatenation-of-all-words/) | Fixed window, word-level |
| 19 | [862. Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/) | Deque + prefix sum (advanced) |

---

### Recognition Exercises

For each problem below, identify:
1. What are the trigger words?
2. Is this fixed or variable window?
3. What is the window state?
4. What makes it invalid?
5. Are you maximizing or minimizing?

**Exercise 1:** "Find the longest subarray with at most 2 distinct integers."

<details>
<summary>Answer</summary>

- Trigger: "longest subarray", "at most 2 distinct"
- Type: Variable window
- State: frequency count of integers in window
- Invalid: more than 2 distinct integers
- Maximize: longest window

</details>

**Exercise 2:** "Given a string, find the length of the longest substring that contains at most K distinct characters."

<details>
<summary>Answer</summary>

- Trigger: "longest substring", "at most K distinct"
- Type: Variable window
- State: frequency count of characters
- Invalid: more than K distinct characters
- Maximize: longest window

</details>

**Exercise 3:** "Find the minimum size subarray whose sum is ≥ target."

<details>
<summary>Answer</summary>

- Trigger: "minimum size subarray", "sum ≥ target"
- Type: Variable window
- State: running sum
- Invalid: sum < target (window is too small)
- Minimize: when sum ≥ target, try to shrink

Wait—this is actually: expand until sum ≥ target, then shrink while sum ≥ target.

</details>

---

### Related Patterns

| Pattern | When to Use Instead |
|---------|---------------------|
| Two Pointers | When array is sorted and you're looking for pairs |
| Prefix Sum | When you need sum of arbitrary ranges, not just contiguous |
| Monotonic Stack | When you need max/min of sliding window (combine with this) |
| Binary Search | When you're searching for the answer in a range, not scanning |
| HashMap | When elements aren't contiguous |

---

### Interview Tips

1. **Start by identifying the window type.** Fixed or variable? This determines your template.

2. **State what the window represents.** "I'm maintaining a window where all elements satisfy [condition]."

3. **Explain the expand-shrink logic.** "I expand by adding the right element. I shrink by removing the left element when [invalidity condition]."

4. **Call out the invariant.** "At every step, the window [is valid / contains exactly K / has sum ≤ target]."

5. **Mention the time complexity.** "This is O(n) because each element enters and leaves the window at most once."

6. **Test with edge cases:**
   - Empty array
   - Single element
   - All elements the same
   - Window size = array size
   - No valid window exists

7. **If stuck, ask:** "Can I rephrase this as finding the longest/shortest contiguous subarray that satisfies a condition?" If yes, it's sliding window.

---

*Next: [02 — Two Pointers](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/DSA%20Handbook/Two-Pointers.md)*
