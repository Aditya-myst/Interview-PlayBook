# 03 — Threads

## Lightweight Execution Within a Process

---

### What is a Thread?

A **thread** is the smallest unit of execution within a process. A process can have multiple threads that share the same memory space but have their own program counter, registers, and stack.

**Analogy:** A process is a house. Threads are people in the house—they share the kitchen (memory) but each has their own bedroom (stack) and current task (program counter).

---

### Process vs Thread

```
┌─────────────────────────────────────────┐
│              Process                     │
│  ┌──────────────────────────────────┐  │
│  │  Code Segment (shared)           │  │
│  │  Data Segment (shared)           │  │
│  │  Heap (shared)                   │  │
│  ├──────────────────────────────────┤  │
│  │  Thread 1    │  Thread 2         │  │
│  │  ┌────────┐  │  ┌────────┐      │  │
│  │  │ Stack  │  │  │ Stack  │      │  │
│  │  │ PC     │  │  │ PC     │      │  │
│  │  │ Regs   │  │  │ Regs   │      │  │
│  │  └────────┘  │  └────────┘      │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

| Aspect | Process | Thread |
|--------|---------|--------|
| **Memory** | Separate address space | Shared address space |
| **Creation** | Slow (fork, allocate memory) | Fast (just create stack/registers) |
| **Switching** | Slow (flush TLB, cache) | Fast (same address space) |
| **Communication** | IPC needed (pipes, shared memory) | Direct (shared variables) |
| **Isolation** | High (one crash doesn't affect others) | Low (one crash kills all threads) |
| **Overhead** | High | Low |

---

### Why Use Threads?

| Benefit | Explanation |
|---------|-------------|
| **Responsiveness** | UI stays responsive while background work continues |
| **Resource Sharing** | Threads share memory, no IPC overhead |
| **Economy** | Cheaper to create and switch than processes |
| **Scalability** | Can run on multiple CPU cores in parallel |

---

### Thread Types

#### User-Level Threads (ULT)
Managed by user-space library, OS doesn't know about them.

```
User Space:  Thread1  Thread2  Thread3
                 \       |       /
                  \      |      /
               Thread Library
                      │
Kernel Space:    Process (OS sees one thread)
```

**Pros:** Fast creation/switching, no kernel mode needed
**Cons:** If one thread blocks, all block; can't use multiple CPUs

#### Kernel-Level Threads (KLT)
Managed by the OS kernel.

```
User Space:  Thread1  Thread2  Thread3
                 \       |       /
                  \      |      /
                    System Calls
                      │
Kernel Space:  KThread1 KThread2 KThread3
```

**Pros:** True parallelism, one thread blocks without affecting others
**Cons:** Slower creation/switching (kernel involvement)

#### Many-to-One Model
```
User Threads:    T1  T2  T3  T4
                  \   |   |   /
                   \  |  |  /
                    Kernel Thread: K1
```
Multiple user threads mapped to one kernel thread. **Problem:** If one blocks, all block.

#### One-to-One Model
```
User Threads:    T1  T2  T3  T4
                  |   |   |   |
Kernel Threads:  K1  K2  K3  K4
```
Each user thread has a kernel thread. **Used by:** Linux, Windows.

#### Many-to-Many Model
```
User Threads:    T1  T2  T3  T4  T5  T6
                  \   /   |   |   /   /
                   \ /    |   |  /   /
                   K1     K2  K3
```
Many user threads multiplexed onto fewer kernel threads.

---

### Thread Control Block (TCB)

Like PCB but for threads:

| Field | Description |
|-------|-------------|
| Thread ID | Unique identifier |
| Program Counter | Next instruction |
| Register Set | CPU register values |
| Stack Pointer | Thread's private stack |
| State | Ready, Running, Blocked |
| Priority | Scheduling priority |

**What's NOT in TCB (shared with process):**
- Code segment
- Data segment
- Open files
- Signals

---

### Thread Creation

```c
// POSIX Threads (pthreads)
#include <pthread.h>

void* thread_function(void* arg) {
    int id = *(int*)arg;
    printf("Thread %d running\n", id);
    return NULL;
}

int main() {
    pthread_t threads[3];
    int ids[3] = {1, 2, 3};
    
    // Create threads
    for (int i = 0; i < 3; i++) {
        pthread_create(&threads[i], NULL, thread_function, &ids[i]);
    }
    
    // Wait for threads to finish
    for (int i = 0; i < 3; i++) {
        pthread_join(threads[i], NULL);
    }
    
    return 0;
}
```

```java
// Java Thread
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getId());
    }
}

// Or implement Runnable
class MyTask implements Runnable {
    public void run() {
        System.out.println("Task running");
    }
}

// Usage
Thread t1 = new MyThread();
t1.start();  // Calls run() in new thread

Thread t2 = new Thread(new MyTask());
t2.start();

// Wait for completion
t1.join();
t2.join();
```

---

### Thread Synchronization Issues

Since threads share memory, concurrent access can cause problems:

```java
// Race Condition Example
class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // NOT atomic! Read, modify, write
    }
}

// Two threads calling increment() simultaneously:
// Thread 1: reads count=0
// Thread 2: reads count=0
// Thread 1: writes count=1
// Thread 2: writes count=1  (should be 2!)
```

**Solution:** Synchronization primitives (covered in Chapter 5).

---

### Thread Safety

A function is **thread-safe** if it works correctly when called from multiple threads simultaneously.

| Approach | Description |
|----------|-------------|
| **Reentrant functions** | Don't use shared data |
| **Mutual exclusion** | Lock shared data (mutex) |
| **Thread-local storage** | Each thread has its own copy |
| **Atomic operations** | Hardware-supported indivisible operations |

---

### Cooperative vs Preemptive Multitasking

| Type | Description | Example |
|------|-------------|---------|
| **Cooperative** | Thread voluntarily yields CPU | Windows 3.1, early Mac OS |
| **Preemptive** | OS forcibly switches threads | Modern Linux, Windows, macOS |

**Problem with cooperative:** One misbehaving thread can freeze the entire system.

---

### Green Threads vs Native Threads

| Type | Description | Example |
|------|-------------|---------|
| **Green threads** | User-space threads, OS unaware | Early Java, Go goroutines |
| **Native threads** | Kernel-managed threads | Modern Java, C++ threads |

**Go's goroutines:** Millions of green threads multiplexed onto fewer OS threads. Very lightweight.

---

### Interview Questions

**Q: What's the difference between a process and a thread?**

A: "A process is an independent program with its own memory space. A thread is a lightweight execution unit within a process that shares the process's memory. Threads are faster to create and switch because they share code, data, and files. The trade-off is that threads have less isolation—one thread crashing can kill the entire process."

**Q: When would you use threads vs processes?**

A: "Use threads when you need shared memory and fast communication—like a web server handling multiple requests. Use processes when you need isolation—like Chrome running each tab in a separate process so one tab crashing doesn't affect others."

**Q: What's a race condition?**

A: "When multiple threads access shared data concurrently and the result depends on the order of execution. Example: two threads incrementing a counter simultaneously might both read the same value and write the same result, losing one increment. Solutions include mutexes, semaphores, and atomic operations."

**Q: What's the difference between user-level and kernel-level threads?**

A: "User-level threads are managed by a library in user space—the OS doesn't know about them. They're fast to create and switch but can't run in parallel on multiple CPUs. Kernel-level threads are managed by the OS—they can run in parallel but are slower to create and switch. Modern systems use one-to-one mapping: each user thread has a kernel thread."

**Q: What's thread-local storage?**

A: "Thread-local storage (TLS) gives each thread its own private copy of a variable. Even though threads share the same code, each thread has its own instance of the TLS variable. This avoids synchronization issues for per-thread data like error codes or transaction IDs."

---

*Next: [04 — CPU Scheduling](04-CPU-Scheduling.md)*
