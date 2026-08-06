# 06 — Deadlocks

## When Everything Stops

---

### What is a Deadlock?

A **deadlock** is a situation where a set of processes are blocked because each process is waiting for a resource held by another process in the set. None can proceed.

**Real-world analogy:** Two people on a narrow bridge. Neither will back up. Both wait forever.

```
Process A holds Resource 1, waits for Resource 2
Process B holds Resource 2, waits for Resource 1

A → (holds) Resource 1 → (waits for) Resource 2 → (held by) B
B → (holds) Resource 2 → (waits for) Resource 1 → (held by) A

Both wait forever = DEADLOCK
```

---

### Necessary Conditions for Deadlock

All four must hold simultaneously:

| Condition | Description | Example |
|-----------|-------------|---------|
| **Mutual Exclusion** | Only one process can use a resource at a time | Printer can only print one job |
| **Hold and Wait** | Process holds resources while waiting for others | A holds R1, waits for R2 |
| **No Preemption** | Resources can't be forcibly taken away | Can't force A to release R1 |
| **Circular Wait** | Circular chain of processes waiting for resources | A waits for B, B waits for A |

**Break any one condition → No deadlock possible.**

---

### Resource-Allocation Graph

```
    ┌───┐         ┌───┐
    │ P1│────────>│ R1│
    └─┬─┘         └─┬─┘
      │              │
      │              │ (assigned)
      ▼              ▼
    ┌───┐         ┌───┐
    │ R2│<────────│ P2│
    └───┘         └───┘

P1 holds R2, requests R1
P2 holds R1, requests R2
= Deadlock (cycle in graph)
```

**If there's a cycle and each resource has one instance → Deadlock exists.**

---

### Handling Deadlocks

#### 1. Deadlock Prevention

Ensure at least one necessary condition can never hold.

| Condition | Prevention Strategy |
|-----------|---------------------|
| Mutual Exclusion | Use shareable resources (not always possible) |
| Hold and Wait | Request all resources at once before execution |
| No Preemption | Allow OS to forcibly take resources |
| Circular Wait | Impose ordering on resource requests |

**Example: Resource Ordering**
```java
// Number resources: R1=1, R2=2, R3=3
// Always request in increasing order

void processA() {
    acquire(R1);  // OK
    acquire(R2);  // OK (R2 > R1)
    // Use resources
    release(R2);
    release(R1);
}

void processB() {
    acquire(R1);  // Must request R1 before R2
    acquire(R2);
    // Use resources
    release(R2);
    release(R1);
}
```

#### 2. Deadlock Avoidance

Make dynamic decisions to avoid unsafe states.

**Banker's Algorithm:** Before granting a resource, check if the system remains in a **safe state**.

```
Safe state: There exists a sequence where all processes can complete.
Unsafe state: Deadlock MIGHT occur (not guaranteed).

Safe → No deadlock possible
Unsafe → Deadlock MAY occur
```

**Banker's Algorithm Example:**

```
3 resource types: A=10, B=5, C=7

Process  Max Need  Current Allocation
  P0      7 5 3      0 1 0
  P1      3 2 2      2 0 0
  P2      9 0 2      3 0 2
  P3      2 2 2      2 1 1
  P4      4 3 3      0 0 2

Available: A=3, B=3, C=2

Check if safe:
1. P1 needs (1,2,2) ≤ (3,3,2)? Yes → Run P1 → Available: (5,3,2)
2. P3 needs (0,1,1) ≤ (5,3,2)? Yes → Run P3 → Available: (7,4,3)
3. P4 needs (4,3,1) ≤ (7,4,3)? Yes → Run P4 → Available: (7,4,5)
4. P0 needs (7,4,3) ≤ (7,4,5)? Yes → Run P0 → Available: (7,5,5)
5. P2 needs (6,0,0) ≤ (7,5,5)? Yes → Run P2 → Available: (10,5,7)

Safe sequence: P1, P3, P4, P0, P2 → System is SAFE
```

#### 3. Deadlock Detection and Recovery

Allow deadlocks to occur, then detect and fix them.

**Detection:**
- Maintain resource-allocation graph
- Periodically check for cycles
- Use wait-for graph (simplified)

**Recovery:**
| Method | Description | Drawback |
|---------|-------------|----------|
| Process Termination | Kill all deadlocked processes | Work lost |
| Resource Preemption | Forcefully take resources from a process | Process may need to restart |
| Rollback | Roll back processes to safe state | Complex |

#### 4. Deadlock Ignorance (Ostrich Algorithm)

Ignore the problem. If deadlocks are rare, the cost of prevention/detection may not be worth it.

**Used by:** Most general-purpose OS (Linux, Windows).
**Reasoning:** Deadlocks are rare. Prevention has overhead. User can kill process.

---

### Livelock vs Deadlock

| Type | Description |
|------|-------------|
| **Deadlock** | Processes blocked, waiting for resources |
| **Livelock** | Processes actively running but making no progress |

```
Livelock example:
Two people in hallway. Both step aside (same side). Both step other side.
They keep moving but never pass each other.
```

---

### Starvation vs Deadlock

| Type | Description |
|------|-------------|
| **Deadlock** | Permanent blocking (cycle) |
| **Starvation** | Indefinite waiting (no cycle, but low priority) |

**Starvation example:** In priority scheduling, low-priority processes may never run.

---

### Interview Questions

**Q: What is a deadlock? What are the necessary conditions?**

A: "A deadlock is when processes are permanently blocked, each waiting for a resource held by another. Four conditions must hold simultaneously: (1) Mutual exclusion—resources can't be shared, (2) Hold and wait—process holds resources while waiting, (3) No preemption—resources can't be forcibly taken, (4) Circular wait—circular chain of dependencies."

**Q: How do you prevent deadlocks?**

A: "Break one of the four conditions. For mutual exclusion: use shareable resources. For hold and wait: request all resources at once. For no preemption: allow OS to reclaim resources. For circular wait: impose an ordering on resource requests and always request in that order."

**Q: What's the Banker's Algorithm?**

A: "A deadlock avoidance algorithm. Before granting a resource, it checks if the system remains in a safe state—one where all processes can eventually complete. If granting would make the system unsafe, the request is denied. It's like a banker ensuring they always have enough cash to satisfy all customers."

**Q: What's the difference between deadlock and starvation?**

A: "Deadlock is permanent blocking—processes are in a circular wait and will never proceed. Starvation is indefinite waiting—a process is ready but never gets CPU or resources due to scheduling decisions. Deadlock requires all four conditions; starvation can occur with priority scheduling."

**Q: How do real operating systems handle deadlocks?**

A: "Most general-purpose OS like Linux and Windows use the ostrich algorithm—they ignore deadlocks. The reasoning is that deadlocks are rare, and the overhead of prevention or detection isn't worth it. If a deadlock occurs, the user can kill the process. Critical systems (medical, aviation) use prevention or avoidance."

---

*Next: [07 — Memory Management](07-Memory-Management.md)*
