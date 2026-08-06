# 08 — Virtual Memory

## The Illusion of Infinite Memory

---

### What is Virtual Memory?

Virtual memory gives each process the illusion that it has its own private, contiguous address space that's much larger than physical RAM.

**Key insight:** Not all of a process's memory needs to be in RAM at once. Only the parts currently being used (working set) need to be in RAM. The rest can stay on disk.

---

### Why Virtual Memory?

| Benefit | Explanation |
|---------|-------------|
| **More processes** | Each process thinks it has lots of memory |
| **Larger programs** | Programs can be larger than physical RAM |
| **Isolation** | Each process has its own address space |
| **Sharing** | Shared libraries mapped once in physical memory |
| **Efficient I/O** | Memory-mapped files |

---

### How It Works

```
Process A sees:        Process B sees:        Physical Memory:
┌────────────┐        ┌────────────┐        ┌────────────┐
│ 0x0000     │        │ 0x0000     │        │ OS         │
│            │        │            │        ├────────────┤
│ Code       │        │ Code       │        │ A's Code   │
│            │        │            │        ├────────────┤
│ Data       │        │ Data       │        │ B's Code   │
│            │        │            │        ├────────────┤
│ Heap       │        │ Heap       │        │ A's Data   │
│            │        │            │        ├────────────┤
│ Stack      │        │ Stack      │        │ B's Data   │
│            │        │            │        ├────────────┤
│ 0xFFFF     │        │ 0xFFFF     │        │ Free       │
└────────────┘        └────────────┘        └────────────┘
   Virtual               Virtual              Physical
   Address Space         Address Space        Memory
```

Each process thinks it has 0x0000 to 0xFFFF. The OS maps virtual pages to physical frames (or disk).

---

### Address Translation

Every memory access goes through translation:

```
CPU: Load from virtual address 0x1234
              │
              ▼
┌─────────────────────────┐
│  MMU (Memory Management │
│       Unit)             │
│                         │
│  Page Table Lookup:     │
│  Virtual Page → Physical│
│  Frame (or Disk)        │
└───────────┬─────────────┘
            │
            ▼
Physical address in RAM (or page fault → load from disk)
```

---

### Demand Paging

Pages are loaded into memory only when accessed (on demand).

```
Process starts
    │
    ▼
Access page 5
    │
    ▼
Is page 5 in RAM? ──Yes──→ Access it
    │
    No (PAGE FAULT)
    │
    ▼
OS: Load page 5 from disk to RAM
    │
    ▼
Update page table
    │
    ▼
Retry instruction
```

**Page fault:** The page isn't in RAM. OS must load it from disk.

**Page fault rate:** If p = 0.001 (1 page fault per 1000 accesses), and disk access = 10ms, memory access = 100ns:

Effective Access Time = (1-p) × memory_access + p × page_fault_time
= 0.999 × 100ns + 0.001 × 10ms
= 99.9ns + 10,000ns
= 10,099.9ns ≈ 100× slower!

**Conclusion:** Page faults are expensive. Minimize them.

---

### Page Replacement Algorithms

When memory is full and a new page is needed, which page to evict?

#### 1. FIFO (First-In, First-Out)

Evict the page that was loaded first.

```
Reference string: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
Frames: 3

Page faults (FIFO): 15
```

**Problem:** Belady's anomaly—more frames can cause more page faults!

#### 2. Optimal (OPT)

Evict the page that won't be used for the longest time.

```
Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1

When page fault occurs, evict the page used furthest in the future.
```

**Problem:** Impossible to implement (requires future knowledge). Used as benchmark.

#### 3. LRU (Least Recently Used)

Evict the page that hasn't been used for the longest time.

```
Reference: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1

Track last use time. Evict the least recently used page.
```

**Implementation:**
| Method | Cost |
|--------|------|
| Counter | O(1) per access, need to find min (O(n)) |
| Stack | O(1) per access, but hardware overhead |
| Approximation | Use reference bits (clock algorithm) |

#### 4. Clock Algorithm (LRU Approximation)

```
Circular buffer with reference bits:

    ┌───┐
    │ 1 │ ← Clock hand
    ├───┤
    │ 0 │
    ├───┤
    │ 1 │
    ├───┤
    │ 0 │
    └───┘

When page fault:
1. Check reference bit at clock hand
2. If 1: set to 0, advance hand (second chance)
3. If 0: evict this page, advance hand
```

**Used in:** Most real operating systems (modified clock).

---

### Frame Allocation

How many frames does each process get?

#### Equal Allocation
Each process gets same number of frames.
- 100 frames, 5 processes → 20 frames each

#### Proportional Allocation
Frames proportional to process size.
- Process A: 100 pages, Process B: 200 pages
- Total: 300 pages, 100 frames
- A gets 33, B gets 67

#### Priority-Based
Higher-priority processes get more frames.

---

### Thrashing

When a process spends more time paging than executing.

```
Process needs more frames than allocated
    │
    ▼
Constantly page-faulting
    │
    ▼
CPU utilization drops
    │
    ▼
OS thinks it needs more processes (to keep CPU busy)
    │
    ▼
Even fewer frames per process
    │
    ▼
More thrashing!
```

**Cause:** Process's working set is larger than allocated frames.

**Solutions:**
1. **Working Set Model:** Track each process's working set, allocate enough frames
2. **Page Fault Frequency:** Monitor page fault rate, adjust allocation
3. **Swap out processes:** Reduce degree of multiprogramming

---

### Copy-on-Write (COW)

Optimization when forking processes.

```
Parent process:
┌──────────────┐
│ Page A (data) │
└──────────────┘

After fork(), before modification:
Parent ──→ Page A ←── Child (shared, read-only)

After child modifies Page A:
Parent ──→ Page A (unchanged)
Child  ──→ Page A' (copy, modified)
```

**Benefit:** fork() is fast—pages are shared until one process modifies them.

---

### Memory-Mapped Files

Map files directly into virtual memory.

```c
// mmap example (Linux)
void *addr = mmap(NULL, length, PROT_READ, MAP_PRIVATE, fd, offset);
// Now access file like memory:
char c = ((char*)addr)[0];  // Reads first byte of file
```

**Benefit:** File I/O through memory operations. OS handles page faults to load file data.

---

### Interview Questions

**Q: What is virtual memory?**

A: "Virtual memory gives each process the illusion of a large, private address space. Only parts currently needed are in RAM; the rest is on disk. The MMU translates virtual addresses to physical addresses using page tables. This allows programs larger than physical RAM, process isolation, and efficient memory sharing."

**Q: What's a page fault?**

A: "A page fault occurs when a process accesses a page that's not in physical RAM. The OS must: (1) trap to kernel, (2) find the page on disk, (3) load it into a free frame (or evict a page if memory is full), (4) update the page table, (5) restart the instruction. Page faults are expensive—disk access is 100,000x slower than RAM."

**Q: Explain LRU page replacement.**

A: "LRU evicts the page that hasn't been used for the longest time. It assumes recently used pages will be used again soon. Implementation uses counters or stacks. It's a good approximation of optimal but expensive to implement exactly. Most systems use approximations like the clock algorithm."

**Q: What's thrashing? How do you prevent it?**

A: "Thrashing is when a process spends more time page-faulting than executing. It occurs when the process's working set exceeds its frame allocation. Prevention: (1) working set model—allocate enough frames for each process's working set, (2) page fault frequency—monitor and adjust allocations, (3) reduce multiprogramming—swap out some processes."

**Q: What's copy-on-write?**

A: "An optimization where parent and child processes share the same memory pages after fork(). Pages are marked read-only. When either process modifies a page, a copy is made first (write triggers copy). This makes fork() fast because no actual copying happens until needed."

**Q: What's the difference between paging and swapping?**

A: "Swapping moves entire processes between memory and disk. Paging moves individual pages—only the pages currently needed are in RAM. Modern systems use demand paging, not whole-process swapping. Paging is more efficient because you only transfer what's needed."

---

*Next: [09 — Paging](09-Paging.md)*
