# 13 — Disk Scheduling

## Optimizing Disk Access

---

### Disk Structure

```
┌─────────────────────────────────────────┐
│                Disk                      │
│  ┌─────────────────────────────────┐   │
│  │         Platter 0               │   │
│  │  ┌─────────────────────────┐   │   │
│  │  │      Surface             │   │   │
│  │  │  ┌──┐ ┌──┐ ┌──┐ ┌──┐  │   │   │
│  │  │  │T0│ │T1│ │T2│ │T3│  │   │   │
│  │  │  └──┘ └──┘ └──┘ └──┘  │   │   │
│  │  │      Tracks              │   │   │
│  │  └─────────────────────────┘   │   │
│  │  ┌─────────────────────────┐   │   │
│  │  │      Surface (bottom)   │   │   │
│  │  └─────────────────────────┘   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Spindle ──→ [rotates]                  │
│                                         │
│  Arm ──→ [moves head across tracks]     │
└─────────────────────────────────────────┘
```

**Key components:**
- **Platter:** Circular disk that spins
- **Track:** Concentric circles on platter
- **Sector:** Arc segment of a track (smallest addressable unit)
- **Cylinder:** Same track number across all platters
- **Read/Write Head:** Arm that moves across tracks

---

### Disk Access Time

```
Access Time = Seek Time + Rotational Latency + Transfer Time

Seek Time:      Move head to correct track (5-15ms)
Rotational Latency: Wait for sector to rotate under head (2-6ms)
Transfer Time:  Actually read/write data (~0.01ms per sector)
```

**Seek time is the dominant factor**—this is what disk scheduling optimizes.

---

### Disk Scheduling Algorithms

Given a queue of disk requests, how should the head move?

**Initial head position: 53**
**Request queue: 98, 183, 37, 122, 14, 124, 65, 67**

#### 1. FCFS (First-Come, First-Served)

Process requests in order.

```
53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67

Movement: |98-53| + |183-98| + |37-183| + |122-37| + |14-122| + |124-14| + |65-124| + |67-65|
= 45 + 85 + 146 + 85 + 108 + 110 + 59 + 2
= 640 cylinders
```

**Pros:** Fair, simple
**Cons:** High seek time, wild head movements

#### 2. SSTF (Shortest Seek Time First)

Always move to the nearest request.

```
53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183

Movement: 12 + 2 + 30 + 23 + 84 + 24 + 2 + 59 = 236 cylinders
```

**Pros:** Much better than FCFS
**Cons:** Can starve distant requests

#### 3. SCAN (Elevator Algorithm)

Move in one direction, servicing requests. Reverse at the end.

```
53 → 37 → 14 → 0 → 65 → 67 → 98 → 122 → 124 → 183

Movement: 16 + 23 + 14 + 65 + 2 + 31 + 24 + 2 + 59 = 236 cylinders
```

**Pros:** No starvation, good performance
**Cons:** Returns to edge even if no requests there

#### 4. C-SCAN (Circular SCAN)

Move in one direction only. When reaching the end, jump back to the beginning.

```
53 → 65 → 67 → 98 → 122 → 124 → 183 → 199 → 0 → 14 → 37

Movement: 12 + 2 + 31 + 24 + 2 + 59 + 16 + 199 + 14 + 23 = 382 cylinders
```

**Pros:** More uniform wait times than SCAN
**Cons:** More total movement

#### 5. LOOK

Like SCAN, but doesn't go to the edge—reverses at the last request.

```
53 → 37 → 14 → 65 → 67 → 98 → 122 → 124 → 183

Movement: 16 + 23 + 51 + 2 + 31 + 24 + 2 + 59 = 208 cylinders
```

#### 6. C-LOOK

Like C-SCAN, but doesn't go to the edge.

```
53 → 65 → 67 → 98 → 122 → 124 → 183 → 14 → 37

Movement: 12 + 2 + 31 + 24 + 2 + 59 + 169 + 23 = 322 cylinders
```

---

### Comparison Table

| Algorithm | Total Movement | Starvation | Fairness |
|-----------|---------------|------------|----------|
| FCFS | 640 | No | Fair |
| SSTF | 236 | Yes | Unfair to distant |
| SCAN | 236 | No | Fair |
| C-SCAN | 382 | No | Very fair |
| LOOK | 208 | No | Fair |
| C-LOOK | 322 | No | Very fair |

**Modern systems:** Use variants of LOOK/C-LOOK with additional optimizations.

---

### Solid-State Drives (SSDs)

SSDs have no moving parts, so traditional disk scheduling is less relevant.

| Aspect | HDD | SSD |
|--------|-----|-----|
| Seek time | 5-15ms | ~0.1ms (no seek) |
| Rotational latency | 2-6ms | None |
| Random read | Slow | Fast |
| Wear | No wear leveling | Wear leveling needed |
| Scheduling | Important | Less important |

**SSD optimization:** Minimize write amplification, wear leveling, TRIM.

---

### Interview Questions

**Q: How does disk scheduling work?**

A: "The OS reorders disk I/O requests to minimize seek time (head movement). Algorithms include FCFS (simple but slow), SSTF (nearest first, can starve), SCAN (elevator, sweeps back and forth), and C-SCAN (circular, more uniform). SCAN and C-SCAN are commonly used because they prevent starvation."

**Q: What's the difference between SCAN and C-SCAN?**

A: "SCAN (elevator) moves in one direction servicing requests, then reverses. C-SCAN moves in one direction only—when it reaches the end, it jumps back to the beginning without servicing. C-SCAN provides more uniform wait times because it treats the disk as circular."

**Q: Why is SSTF not used in practice?**

A: "SSTF can cause starvation—requests near the current head position get served repeatedly while distant requests wait indefinitely. SCAN and LOOK avoid this by ensuring the head eventually reaches all requests."

**Q: How do SSDs change disk scheduling?**

A: "SSDs have no moving parts—no seek time or rotational latency. Random access is nearly as fast as sequential. Traditional disk scheduling (minimizing head movement) is irrelevant. SSD optimization focuses on wear leveling, write amplification, and TRIM commands."

---

*Next: [14 — System Calls](14-System-Calls.md)*
