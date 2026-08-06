# 02 — Processes

## The Fundamental Unit of Execution

---

### What is a Process?

A **process** is a program in execution. It's the fundamental unit of work in an operating system.

**Program vs Process:**
- **Program:** Static code on disk (e.g., `chrome.exe` file)
- **Process:** Dynamic instance of a program in memory (running Chrome)

**Analogy:** A recipe (program) vs actually cooking (process). Multiple people can follow the same recipe simultaneously (multiple processes of the same program).

---

### Process Control Block (PCB)

The OS tracks each process using a **Process Control Block** (also called Task Control Block in Linux).

```
┌─────────────────────────────────┐
│      Process Control Block      │
├─────────────────────────────────┤
│ Process ID (PID)       │ 1234  │
│ Process State          │ Running│
│ Program Counter        │ 0x400 │
│ CPU Registers          │ ...   │
│ Memory Limits          │ ...   │
│ Open File Table        │ ...   │
│ Scheduling Info        │ ...   │
│ I/O Status             │ ...   │
│ Accounting Info        │ ...   │
└─────────────────────────────────┘
```

| Field | What It Stores |
|-------|---------------|
| **PID** | Unique process identifier |
| **Process State** | New, Ready, Running, Waiting, Terminated |
| **Program Counter** | Address of next instruction |
| **CPU Registers** | Values of registers when not running |
| **Memory Info** | Base/bounds registers, page tables |
| **Open Files** | List of open file descriptors |
| **Scheduling Info** | Priority, queue pointers |
| **I/O Status** | Pending I/O requests, allocated devices |

---

### Process States

```
                    ┌──────────┐
        ┌──────────>│   New    │
        │           └────┬─────┘
        │                │ admit
        │                ▼
┌───────┴───────┐  ┌──────────┐  ┌──────────────┐
│  Terminated   │  │  Ready   │──>│   Running    │
└───────────────┘  └─────┬────┘  └──────┬───────┘
                         ▲              │
                         │              │ I/O or event wait
                         │              ▼
                         │        ┌──────────┐
                         └────────│ Waiting  │
                          event   └──────────┘
                          complete
```

| State | Description |
|-------|-------------|
| **New** | Process is being created |
| **Ready** | Waiting to be assigned to CPU |
| **Running** | Instructions being executed |
| **Waiting** | Waiting for I/O or event |
| **Terminated** | Finished execution |

---

### Process Creation

When a process creates another process:
- **Parent process** creates **child process**
- Child gets its own PCB, memory space, resources
- Parent may wait for child or continue executing

```c
// Unix/Linux process creation
pid_t pid = fork();

if (pid == 0) {
    // Child process
    printf("I am the child (PID: %d)\n", getpid());
    execvp("/bin/ls", {"ls", "-l", NULL});  // Replace with ls command
} else if (pid > 0) {
    // Parent process
    printf("I am the parent (PID: %d), child PID: %d\n", getpid(), pid);
    wait(NULL);  // Wait for child to finish
} else {
    // Fork failed
    perror("fork failed");
}
```

**Key points about `fork()`:**
- Creates an exact copy of the parent process
- Parent and child continue from the same point
- Child gets a new PID
- Returns 0 to child, child's PID to parent

---

### Process Termination

A process terminates when:
1. It calls `exit()` (normal completion)
2. It's killed by another process (`kill` command)
3. It encounters a fatal error (segmentation fault)
4. The parent process terminates (in some systems)

When a process terminates:
1. Its resources are deallocated (memory, files)
2. Its PCB is removed
3. Its children may become orphans (adopted by `init` process)
4. Its exit status is sent to the parent

---

### Zombie and Orphan Processes

| Type | Description | How It Happens |
|------|-------------|----------------|
| **Zombie** | Terminated but parent hasn't collected exit status | Parent didn't call `wait()` |
| **Orphan** | Parent terminated before child | Parent exits, child still running |

```c
// Creating a zombie (bad practice!)
if (fork() == 0) {
    exit(0);  // Child exits immediately
} else {
    sleep(60);  // Parent sleeps, doesn't call wait()
    // Child is zombie for 60 seconds
}
```

**Fix for zombies:** Parent should call `wait()` or use signal handlers.

---

### Process vs Program

| Aspect | Program | Process |
|--------|---------|---------|
| **Nature** | Static (file on disk) | Dynamic (in memory) |
| **Lifetime** | Exists until deleted | Created to terminated |
| **Resources** | None (just code) | Memory, files, CPU time |
| **Multiplicity** | One copy | Multiple processes possible |
| **Location** | Secondary storage | Main memory (RAM) |

---

### Context Switching

When the CPU switches from one process to another:

```
Process A running
    │
    ▼
Save Process A's state (PCB)
    │
    ▼
Load Process B's state (PCB)
    │
    ▼
Process B running
```

**What's saved/loaded:**
- Program counter
- CPU registers
- Memory management info (page tables)
- I/O state

**Cost:** Context switching is pure overhead no useful work is done during the switch. Typical time: 1-1000 microseconds.

---

### Process Scheduling Queues

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  ┌─────────┐     ┌─────────┐     ┌──────────┐         │
│  │ Job     │────>│ Ready   │────>│ CPU      │         │
│  │ Queue   │     │ Queue   │     │          │         │
│  └─────────┘     └────┬────┘     └────┬─────┘         │
│                       │              │                 │
│                       │         ┌────┴─────┐           │
│                       │         │ Running  │           │
│                       │         └────┬─────┘           │
│                       │              │                 │
│                       │    ┌─────────┼─────────┐       │
│                       │    │         │         │       │
│                       │    ▼         ▼         ▼       │
│                       │ ┌──────┐ ┌──────┐ ┌──────┐    │
│                       │ │ I/O  │ │ I/O  │ │Wait  │    │
│                       │ │Queue1│ │Queue2│ │Queue │    │
│                       │ └──┬───┘ └──┬───┘ └──┬───┘    │
│                       │    │         │         │       │
│                       └────┴─────────┴─────────┘       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

| Queue | Purpose |
|-------|---------|
| **Job Queue** | All processes in the system |
| **Ready Queue** | Processes ready to execute |
| **Device Queues** | Processes waiting for I/O devices |

---

### Inter-Process Communication (IPC)

Processes need to communicate. Two models:

#### 1. Shared Memory
```c
// Process A writes
int *shared = shmat(shmid, NULL, 0);
*shared = 42;

// Process B reads
int *shared = shmat(shmid, NULL, 0);
printf("%d\n", *shared);  // Prints 42
```

#### 2. Message Passing
```c
// Process A sends
send(channel, &message);

// Process B receives
receive(channel, &message);
```

| Model | Pros | Cons |
|-------|------|------|
| **Shared Memory** | Fast (once set up) | Synchronization issues |
| **Message Passing** | Easier to use | Slower (kernel involvement) |

---

### Interview Questions

**Q: What is a process?**

A: "A process is a program in execution. It's the fundamental unit of work in an OS. It has its own memory space, program counter, registers, and open files. The OS tracks each process using a Process Control Block (PCB)."

**Q: What's the difference between a process and a program?**

A: "A program is static code on disk. A process is a dynamic instance of that program in memory. One program can have multiple processes—like opening multiple Chrome windows. Each process has its own memory space and state."

**Q: What happens during a context switch?**

A: "The OS saves the current process's state (registers, program counter, page table) to its PCB, then loads another process's state from its PCB. This is pure overhead no useful work happens during the switch. The cost depends on hardware support and memory speed."

**Q: What's a zombie process?**

A: "A zombie is a terminated process whose parent hasn't collected its exit status via wait(). The process is dead but its PCB remains in the process table. Zombies consume no resources but waste a process table slot. They're cleaned up when the parent calls wait() or when the parent terminates."

**Q: What's the difference between fork() and exec()?**

A: "fork() creates a new process by duplicating the parent—it returns twice (once in parent, once in child). exec() replaces the current process's code with a new program. Together: fork() creates the child, then exec() loads the new program into the child."

---

*Next: [03 — Threads](03-Threads.md)*
