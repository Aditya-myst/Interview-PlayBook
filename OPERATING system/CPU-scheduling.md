# 04 — CPU Scheduling

## Deciding What Runs Next

---

### Why Scheduling?

The CPU is a shared resource. Multiple processes want to use it. The **scheduler** decides which process gets the CPU and for how long.

**Goals:**
- Maximize CPU utilization
- Maximize throughput (processes completed per time)
- Minimize turnaround time (total time from submission to completion)
- Minimize waiting time (time spent in ready queue)
- Minimize response time (time to first response)

---

### Types of Scheduling

```
┌─────────────────────────────────────────────────────────┐
│                    Long-Term Scheduler                   │
│  (Job Scheduler)                                        │
│  Selects which processes from job pool enter ready queue │
│  Controls degree of multiprogramming                    │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│                  Medium-Term Scheduler                   │
│  (Swapper)                                              │
│  Swaps processes in/out of memory                       │
│  Handles memory overcommitment                          │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│                   Short-Term Scheduler                   │
│  (CPU Scheduler)                                        │
│  Selects which ready process gets CPU next              │
│  Runs frequently (milliseconds)                         │
└─────────────────────────────────────────────────────────┘
```

| Scheduler | Frequency | Speed | Controls |
|-----------|-----------|-------|----------|
| Long-term | Minutes/slow | Slow | Which processes enter system |
| Medium-term | Seconds | Medium | Which processes stay in memory |
| Short-term | Milliseconds | Fast | Which process gets CPU |

---

### Preemptive vs Non-Preemptive

| Type | Description | Example |
|------|-------------|---------|
| **Non-preemptive** | Process keeps CPU until it releases it | FCFS, SJF (non-preemptive) |
| **Preemptive** | OS can forcibly take CPU away | Round Robin, SRTF, Priority |

**Modern systems are preemptive.** A timer interrupt forces the OS to reconsider which process should run.

---

### Scheduling Algorithms

#### 1. First-Come, First-Served (FCFS)

Simplest algorithm. Process that arrives first runs first.

```
Process  Arrival  Burst
  P1       0       4
  P2       1       3
  P3       2       1

FCFS order: P1 → P2 → P3

Timeline:
|P1|P1|P1|P1|P2|P2|P2|P3|
0  1  2  3  4  5  6  7  8

Waiting time: P1=0, P2=3, P3=5
Average waiting time: (0+3+5)/3 = 2.67
```

**Problem: Convoy Effect** — Short processes stuck behind long ones.

#### 2. Shortest Job First (SJF)

Process with shortest burst time runs first.

```
Process  Arrival  Burst
  P1       0       4
  P2       1       3
  P3       2       1

SJF order: P1 → P3 → P2

Timeline:
|P1|P1|P1|P1|P3|P2|P2|P2|
0  1  2  3  4  5  6  7  8

Waiting time: P1=0, P3=2, P2=4
Average waiting time: (0+2+4)/3 = 2.0 (better than FCFS!)
```

**Problem:** Requires knowing burst time in advance. Starvation of long processes.

#### 3. Shortest Remaining Time First (SRTF)

Preemptive version of SJF. If a new process has shorter remaining time, preempt.

```
Process  Arrival  Burst
  P1       0       4
  P2       1       3
  P3       2       1

Timeline:
|P1|P2|P3|P3|P2|P2|P1|P1|P1|
0  1  2  3  4  5  6  7  8

Wait for P3: arrives at 2, runs immediately (shortest)
Wait for P2: runs after P3
Wait for P1: resumes after P2
```

#### 4. Round Robin (RR)

Each process gets a fixed time slice (quantum). After the quantum, the process goes to the back of the queue.

```
Process  Arrival  Burst    Quantum = 2
  P1       0       4
  P2       1       3
  P3       2       1

Timeline:
|P1|P1|P2|P2|P3|P1|P1|P2|
0  1  2  3  4  5  6  7  8

P1: runs 0-2, waits, runs 5-7
P2: runs 2-4, waits, runs 7-8
P3: runs 4-5 (done!)
```

**Quantum too small:** Too many context switches (overhead).
**Quantum too large:** Becomes FCFS.
**Rule of thumb:** 80% of CPU bursts should be shorter than the quantum.

#### 5. Priority Scheduling

Each process has a priority. Highest priority runs first.

```
Process  Arrival  Burst  Priority (lower = higher priority)
  P1       0       4       3
  P2       1       3       1
  P3       2       1       2

Priority order: P2 → P3 → P1

Timeline:
|P1|P2|P2|P2|P3|P1|P1|P1|
0  1  2  3  4  5  6  7  8
```

**Problem:** Starvation of low-priority processes.
**Solution:** **Aging** — Increase priority of waiting processes over time.

#### 6. Multilevel Queue

Separate queues for different process types.

```
┌─────────────────────────────────────────┐
│  Queue 1 (High Priority, Round Robin)   │  ← Foreground/Interactive
├─────────────────────────────────────────┤
│  Queue 2 (Medium Priority, RR)          │  ← Batch
├─────────────────────────────────────────┤
│  Queue 3 (Low Priority, FCFS)           │  ← Background
└─────────────────────────────────────────┘

Priority: Queue 1 > Queue 2 > Queue 3
```

#### 7. Multilevel Feedback Queue

Processes can move between queues based on behavior.

```
New process → Queue 1 (high priority, short quantum)
                │
                │ Uses full quantum?
                ▼
            Queue 2 (medium priority, longer quantum)
                │
                │ Uses full quantum?
                ▼
            Queue 3 (low priority, FCFS)
```

**I/O-bound processes stay in high-priority queues** (they give up CPU quickly).
**CPU-bound processes move to low-priority queues** (they use full quantum).

---

### Comparison Table

| Algorithm | Preemptive | Starvation | Overhead | Best For |
|-----------|-----------|------------|----------|----------|
| FCFS | No | No | Low | Simple batch systems |
| SJF | No | Yes | Low | Known burst times |
| SRTF | Yes | Yes | Medium | Theoretical optimal |
| Round Robin | Yes | No | Medium | Time-sharing systems |
| Priority | Both | Yes | Low | Real-time systems |
| Multilevel | Yes | Possible | High | General-purpose OS |

---

### The "Best" Algorithm

There's no single best algorithm. The choice depends on:
- **Interactive systems:** Round Robin or Multilevel Feedback Queue
- **Batch systems:** SJF or Priority
- **Real-time systems:** Priority with deadline scheduling
- **General purpose:** Multilevel Feedback Queue (Linux CFS)

---

### Linux CFS (Completely Fair Scheduler)

Linux uses a sophisticated scheduler that aims for fairness:

- Each process gets a fair share of CPU based on weight (nice value)
- Uses a red-black tree for O(log n) scheduling decisions
- Tracks **virtual runtime** — processes with less runtime get priority
- Time slice is dynamic based on number of runnable processes

---

### Interview Questions

**Q: What's the difference between preemptive and non-preemptive scheduling?**

A: "In non-preemptive scheduling, a process keeps the CPU until it voluntarily releases it (completes or blocks for I/O). In preemptive scheduling, the OS can forcibly take the CPU away using timer interrupts. Modern systems use preemptive scheduling to ensure fairness and responsiveness."

**Q: Explain the Round Robin algorithm. What's the effect of quantum size?**

A: "Round Robin gives each process a fixed time slice (quantum) and cycles through the ready queue. If the quantum is too small, context switches dominate overhead. If too large, it becomes FCFS. A good quantum balances responsiveness and overhead—typically 10-100ms."

**Q: What's the convoy effect?**

A: "In FCFS scheduling, short processes get stuck behind one long-running process. The CPU is busy with the long process while many short processes wait, increasing average waiting time. SJF or Round Robin avoid this."

**Q: What's starvation in priority scheduling?**

A: "Low-priority processes may never execute if high-priority processes keep arriving. Solution: aging—gradually increase the priority of waiting processes. Eventually, even low-priority processes become high-priority and get executed."

**Q: How does the Multilevel Feedback Queue work?**

A: "It has multiple queues with different priorities and time quanta. New processes enter the highest-priority queue. If a process uses its full quantum (CPU-bound), it moves to a lower queue. If it blocks for I/O (I/O-bound), it stays in a high queue. This naturally favors interactive processes."

---

*Next: [05 — Synchronization](05-Synchronization.md)*
