# 16 — OS Interview Questions & Answers

## Real Questions, Real Answers

---

### Process & Thread Questions

#### Q: What's the difference between a process and a thread?

**A:** "A process is an independent program with its own memory space, program counter, and resources. A thread is a lightweight execution unit within a process—threads share the same memory space but have their own stack, registers, and program counter. Processes are isolated (one crash doesn't affect others); threads are not (one thread crash can kill the process). Context switching between processes is expensive (flush TLB, cache); between threads is cheap (same address space)."

---

#### Q: What happens during a context switch?

**A:** "The OS saves the current process's state (program counter, registers, page table pointer) to its PCB, then loads another process's state from its PCB. Steps: (1) timer interrupt, (2) save current process state, (3) update PCB, (4) run scheduler to pick next process, (5) load next process state, (6) resume execution. Cost: 1-1000 microseconds depending on hardware."

---

#### Q: What's the difference between user-level and kernel-level threads?

**A:** "User-level threads are managed by a library in user space—the OS sees only one thread per process. They're fast to create/switch but can't run in parallel on multiple CPUs. Kernel-level threads are managed by the OS—they can run in parallel but creation/switching involves kernel overhead. Modern systems use one-to-one mapping: each user thread has a kernel thread."

---

#### Q: Explain fork() and exec().

**A:** "fork() creates a new process by duplicating the parent—the child gets a copy of the parent's memory, file descriptors, and execution state. It returns twice: 0 in the child, child's PID in the parent. exec() replaces the current process's code and data with a new program—it doesn't create a new process. Together: fork() creates the child, exec() loads the new program."

---

### Scheduling Questions

#### Q: Compare FCFS, SJF, and Round Robin scheduling.

**A:** "FCFS: simplest, processes in arrival order. Problem: convoy effect—short processes stuck behind long ones. SJF: shortest burst first—optimal average waiting time but requires knowing burst times and can starve long processes. Round Robin: each process gets a time quantum—fair and responsive but context switch overhead. RR is best for interactive systems; SJF for batch systems with known times."

---

#### Q: What's the convoy effect?

**A:** "In FCFS scheduling, one long-running CPU-bound process blocks many short I/O-bound processes behind it. The CPU is busy with the long process while short processes wait, increasing average waiting time and reducing I/O device utilization. Round Robin or SJF avoid this."

---

#### Q: What's starvation in priority scheduling?

**A:** "Low-priority processes may never execute if high-priority processes keep arriving. Solution: aging—gradually increase the priority of waiting processes. After waiting long enough, a low-priority process becomes high-priority and gets executed."

---

### Synchronization Questions

#### Q: What's a race condition?

**A:** "When multiple threads access shared data concurrently and the result depends on execution order. Example: two threads incrementing a counter might both read the same value and write the same result, losing one increment. Solutions: mutexes, semaphores, atomic operations."

---

#### Q: What's the difference between a mutex and a semaphore?

**A:** "A mutex has ownership—the thread that locks it must unlock it. It's binary (locked/unlocked) for mutual exclusion. A semaphore has no ownership—any thread can signal it. It can count (0 to N) for controlling access to multiple resources. Binary semaphores are like mutexes; counting semaphores allow N concurrent accesses."

---

#### Q: Explain the Producer-Consumer problem.

**A:** "Producers create data; consumers process it. They share a bounded buffer. Solution: three semaphores—mutex (binary, protects buffer access), empty (counts empty slots), full (counts full slots). Producer: wait(empty), wait(mutex), add item, signal(mutex), signal(full). Consumer: wait(full), wait(mutex), remove item, signal(mutex), signal(empty)."

---

#### Q: What's a deadlock? How do you prevent it?

**A:** "Deadlock: processes permanently blocked, each waiting for a resource held by another. Four conditions: mutual exclusion, hold and wait, no preemption, circular wait. Prevention: break any condition. Avoidance: Banker's algorithm checks for safe state. Detection: cycle detection in wait-for graph. Recovery: kill process or preempt resource. Most OS ignore it (ostrich algorithm)."

---

### Memory Questions

#### Q: What's virtual memory?

**A:** "Virtual memory gives each process the illusion of a large, private address space. Only pages currently needed are in RAM; the rest are on disk. The MMU translates virtual addresses to physical addresses using page tables. Benefits: programs larger than RAM, process isolation, efficient memory sharing."

---

#### Q: What's a page fault?

**A:** "When a process accesses a page not in RAM. The OS: (1) traps to kernel, (2) finds page on disk, (3) loads into a free frame (evicting a page if needed), (4) updates page table, (5) restarts instruction. Page faults are expensive—disk access is 100,000x slower than RAM."

---

#### Q: Explain LRU page replacement.

**A:** "LRU evicts the page that hasn't been used for the longest time. Assumes recently used pages will be used again (temporal locality). Implementation: counters or stacks. Approximation: clock algorithm (reference bits). LRU performs well but is expensive to implement exactly—most systems use approximations."

---

#### Q: What's thrashing?

**A:** "When a process spends more time page-faulting than executing. Its working set exceeds allocated frames. CPU utilization drops, OS may add more processes (making it worse). Prevention: working set model (allocate enough frames), page fault frequency monitoring, reduce multiprogramming."

---

#### Q: What's the difference between internal and external fragmentation?

**A:** "Internal fragmentation: wasted space within an allocated block (process doesn't fully use partition). External fragmentation: free memory scattered in small pieces (enough total free memory but not contiguous). Fixed partitioning → internal fragmentation. Dynamic partitioning → external fragmentation. Paging eliminates external fragmentation."

---

### File System Questions

#### Q: What's an inode?

**A:** "An inode stores file metadata—permissions, owner, timestamps, size, and pointers to data blocks. Contains 12 direct pointers, plus single/double/triple indirect pointers for large files. Doesn't store filename—directory entries map names to inodes. This allows hard links (multiple names for same inode)."

---

#### Q: What's the difference between hard links and symbolic links?

**A:** "Hard link: another directory entry pointing to the same inode. Both entries are equal—deleting one doesn't affect the file until all links are removed. Symbolic link: a file containing the path to another file. Can span file systems, can point to non-existent files (dangling). Deleting the original makes the symlink invalid."

---

#### Q: What's a journaling file system?

**A:** "Records changes to a journal before applying them. If crash during write: incomplete entries rolled back, complete entries replayed. Prevents corruption and makes recovery fast (replay journal) instead of scanning entire disk (fsck). Used by ext3/ext4, NTFS, HFS+."

---

### I/O Questions

#### Q: What's the difference between polling and interrupts?

**A:** "Polling: CPU continuously checks device status—wastes CPU. Interrupts: device signals CPU when ready—CPU does other work. DMA: device transfers directly to memory, interrupts CPU when done—most efficient for large transfers."

---

#### Q: What is DMA?

**A:** "Direct Memory Access: devices transfer data directly to/from memory without CPU. CPU sets up transfer (source, destination, size), DMA handles it, then interrupts CPU when done. Efficient for large transfers like disk reads."

---

### Advanced Questions

#### Q: What's a system call?

**A:** "Mechanism for user programs to request kernel services. Switches from user mode to kernel mode. Examples: fork(), read(), write(), exec(). Steps: put syscall number in register, execute trap instruction, kernel looks up handler, executes, returns to user mode."

---

#### Q: What's the difference between user mode and kernel mode?

**A:** "User mode: limited privileges, can't access hardware or modify page tables. Kernel mode: full access. System calls switch to kernel mode. Hardware has a mode bit: 0 for kernel, 1 for user. This separation protects the system from buggy or malicious applications."

---

#### Q: Explain the Banker's Algorithm.

**A:** "Deadlock avoidance algorithm. Before granting a resource, checks if system remains in a safe state (all processes can complete). Maintains: available resources, maximum need, current allocation. For each request: simulate granting, check if safe sequence exists. If safe, grant; otherwise, deny."

---

#### Q: What's copy-on-write?

**A:** "Optimization where parent and child share memory pages after fork(). Pages are read-only. When either process writes, a copy is made. This makes fork() fast—no actual copying until modification. Used in Unix fork(), virtual memory, and some file systems."

---

#### Q: What's a race condition vs deadlock?

**A:** "Race condition: result depends on execution order—data corruption, incorrect values. Fix: synchronization (mutexes, semaphores). Deadlock: processes permanently blocked waiting for each other—system hangs. Fix: prevention, avoidance, detection. Race conditions are bugs in code; deadlocks are design issues."

---

### The "I Don't Know" Strategy

If you're stumped:

1. **Start with what you know.** "I know that in general..."
2. **Think out loud.** "Let me think about this..."
3. **Give an analogy.** "This is similar to..."
4. **Ask for clarification.** "Can you clarify what aspect you're asking about?"

---

### Quick Reference: Key Numbers

| Metric | Typical Value |
|--------|---------------|
| RAM access | 100 ns |
| SSD access | 100 μs |
| HDD access | 10 ms |
| Context switch | 1-10 μs |
| System call | 100-1000 ns |
| TLB lookup | 1 clock cycle |
| Page fault | 10 ms |
| Thread creation | 10-100 μs |
| Process creation | 100 μs - 1 ms |

---

### The 60-Second OS Summary

If asked to summarize OS in 60 seconds:

"An OS manages hardware and provides services to applications. Key concepts: Processes—independent programs with their own memory. Threads—lightweight execution units sharing memory. CPU Scheduling—deciding what runs next (Round Robin, SJF). Synchronization—mutexes and semaphores prevent race conditions. Deadlocks—four conditions, prevent by breaking one. Virtual Memory—gives each process illusion of large address space using demand paging. Paging—divides memory into fixed-size pages, page table maps virtual to physical. File Systems—organize data on disk using inodes, directories, allocation methods. System Calls—the bridge between user programs and kernel."

---

*Good luck with your OS interviews!*
