# 00 — How to Study This Book

## The Strategy That Separates People Who Improve From People Who Grind

---

### The Problem With "Just Solve More Problems"

Most people prepare for coding interviews like this:

1. Open LeetCode
2. Pick a random problem
3. Struggle for 45 minutes
4. Look at the solution
5. Think "that's clever"
6. Move on

**300 problems later, they still can't solve new ones.**

Why? Because they treated every problem as unique. They memorized *solutions* instead of internalizing *patterns*.

---

### The Pattern-First Approach

Here's what works:

1. **Learn the pattern skeleton ONCE**
2. **Apply it to 5-10 problems immediately**
3. **See a new problem → "This is sliding window" → Write the template → Handle the edge case**

The key insight: **patterns transfer between problems. Individual solutions don't.**

| Approach | Problems Solved | New Medium Solve Rate |
|----------|----------------|----------------------|
| Random grinding | 300+ | ~30% |
| Pattern-first | 80 (5 × 16) | ~75% |

Less work. Better results. Because the pattern is reusable.

---

### The Learning Cycle

For each pattern, follow this exact sequence:

#### Phase 1: Understand (30 minutes)

Read the chapter. Focus on:
- **What** the pattern is
- **When** to recognize it (trigger words)
- **Why** it works (the invariant)

Don't memorize the code. Understand the *shape* of the solution.

#### Phase 2: Template Drill (1 hour)

Solve **3 Easy problems** using ONLY the template.

Rules:
- Don't deviate from the template structure
- Don't optimize
- Don't worry about edge cases yet
- Just get the template working 3 times

This builds muscle memory. Your fingers should learn the pattern before your brain does.

#### Phase 3: Recognition Training (1-2 hours)

Solve **2 Medium problems** where the pattern isn't labeled.

Rules:
- Read the problem WITHOUT knowing which pattern it is
- Spend 5 minutes trying to identify the pattern yourself
- If stuck, check the trigger words (not the solution)
- Implement using the template

This is the most important phase. It trains the skill that actually matters: **recognizing which pattern applies to an unseen problem.**

#### Phase 4: Spaced Repetition (1 week later)

Come back to the pattern WITHOUT looking at the template.

- Try to solve a Medium from scratch
- If you can write the template from memory and solve the problem, you've internalized it
- If you can't, repeat Phase 2 and 3

---

### The 80/20 Rule

Each pattern has an "80% template"—a skeleton that solves roughly 80% of problems in that category.

The remaining 20% is problem-specific: edge cases, modified conditions, hybrid patterns.

**Learn the 80% first.** The 20% comes with practice.

Example:

```
# The 80% Sliding Window Template
left = 0
for right in range(len(nums)):
    add(nums[right])
    while invalid():
        remove(nums[left])
        left += 1
    update_answer()
```

This exact structure (with different `invalid()` and `update_answer()` functions) solves:
- Longest Substring Without Repeating Characters
- Minimum Window Substring
- Longest Repeating Character Replacement
- Max Consecutive Ones III
- Fruits Into Baskets
- And 20+ more problems

Same skeleton. Different validity condition.

---

### How to Read Each Chapter

Each chapter has this structure:

| Section | Purpose | Time |
|---------|---------|------|
| What It Is | Understand the concept | 5 min |
| When to Use It | Learn trigger words | 5 min |
| Mental Model | Visualize the pattern | 5 min |
| Brute Force → Optimized | See the optimization journey | 10 min |
| The 80% Template | Memorize the skeleton | 5 min |
| Variations | Know the sub-patterns | 10 min |
| Common Mistakes | Avoid known bugs | 5 min |
| Dry Run | See it in action | 10 min |
| Code | Reference implementations | Reference |
| Practice Problems | Apply the pattern | 1-2 hours |

**Total per chapter: ~1 hour of reading + 1-2 hours of practice**

---

### The Problem Selection Strategy

Not all problems are equally useful for learning a pattern.

**Best for learning:**
- Problems that are "pure" applications of the pattern
- Problems where the pattern is the main difficulty
- Easy and Medium problems

**Worst for learning (at first):**
- Hard problems with multiple patterns combined
- Problems where the pattern is a minor component
- Problems with heavy implementation details

**Save these for later**, after you've internalized the individual patterns.

---

### When to Move On

You're ready for the next pattern when:

1. ✅ You can write the 80% template from memory
2. ✅ You can identify the pattern from a problem description (without hints)
3. ✅ You can solve a Medium in under 25 minutes using the template
4. ✅ You understand WHY the template works, not just HOW

If any of these are false, spend another hour on the current pattern.

---

### The Recognition Matrix

After learning all 16 patterns, you should be able to look at a problem and think:

```
"What does this problem WANT?"

Minimum/Maximum of something?     → Optimization problem
All possible solutions?           → Backtracking / DFS
Shortest path?                    → BFS / Dijkstra
Can I make a greedy choice?       → Greedy
Does it have overlapping subproblems? → DP
Is the array sorted?              → Binary Search / Two Pointers
Contiguous subarray/substring?    → Sliding Window
Need to track "next greater"?     → Monotonic Stack
Need top K elements?              → Heap
Connected components?             → Union Find / DFS
Dependencies/ordering?            → Topological Sort
```

This decision tree is what separates someone who can solve 30% of new problems from someone who can solve 75%.

---

### The "I've Seen This Before" Feeling

As you progress through the patterns, you'll start having a strange experience:

> "Wait, this new problem... it feels like that other problem I solved."

That feeling is pattern recognition developing. Trust it. It means your brain is starting to see the structure underneath the surface details.

When you feel it:
1. Identify which pattern it reminds you of
2. Write down WHY (what trigger words or structural similarity you noticed)
3. Apply the template and see if it works

Most of the time, it will.

---

### The 6-Week Plan

If you follow this book at 2-3 problems per day:

| Week | Patterns | Problems |
|------|----------|----------|
| 1 | Sliding Window, Two Pointers | 10 |
| 2 | Binary Search, Prefix Sum + HashMap | 10 |
| 3 | Monotonic Stack, Heap, Linked Lists | 15 |
| 4 | Trees, DFS & BFS | 10 |
| 5 | Backtracking, Dynamic Programming | 10 |
| 6 | Greedy, Union Find, Topological Sort, Shortest Path, Bit Manipulation | 25 |
| **Total** | **16 patterns** | **~80 problems** |

That's 80 problems to cover what most people need 300+ for.

---

### What This Book Is NOT

- **Not a replacement for coding practice.** You still need to write code. Reading templates isn't enough.
- **Not a guarantee.** Some interviews test system design, behavioral skills, or domain knowledge that algorithms won't cover.
- **Not comprehensive for competitive programming.** This book targets interview preparation, not ICPC/Codeforces.
- **Not a substitute for understanding fundamentals.** If you don't know what a hash map does or how recursion works, start with a data structures textbook first.

---

### Prerequisites

Before starting this book, you should be comfortable with:

- [ ] Variables, loops, conditionals in at least one language
- [ ] Arrays and strings
- [ ] Hash maps / dictionaries
- [ ] Basic recursion
- [ ] Big-O notation (O(n), O(n²), O(log n))
- [ ] Linked lists, stacks, queues (conceptual understanding)
- [ ] Trees and graphs (conceptual understanding)

If any of these are shaky, spend a week on fundamentals first. This book will still be here.

---

### A Note on Language

All code examples are provided in **Python, Java, C++, and JavaScript**.

Python is used for explanations and dry runs because it's the most concise. The other languages are provided for reference.

Pick one language for your interview prep and stick with it. Don't switch languages mid-preparation.

---

### Let's Begin

Start with [01 — Sliding Window](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/DSA%20Handbook/Sliding-window.md). It's the highest-ROI pattern for interviews and the easiest to learn.

Or jump to whichever pattern you need most. The chapters are self-contained.

Good luck. You're about to see problems differently.
