# 05 — Synchronization

## Managing Concurrent Access to Shared Resources

---

### The Problem: Race Conditions

When multiple threads access shared data concurrently, the result depends on the order of execution. This is a **race condition**.

```java
// Shared counter
int counter = 0;

// Thread 1                    // Thread 2
counter++;                      counter++;
// Read counter (0)            // Read counter (0)
// Add 1 (1)                   // Add 1 (1)
// Write counter (1)           // Write counter (1)

// Expected: 2, Actual: 1 (lost update!)
```

**Solution:** Synchronization ensures only one thread accesses the critical section at a time.

---

### Critical Section Problem

The **critical section** is the code that accesses shared resources.

```
do {
    entry section      ← Request permission to enter
    critical section   ← Access shared resource
    exit section       ← Release permission
    remainder section  ← Non-critical code
} while (true);
```

**Requirements for a solution:**
1. **Mutual Exclusion:** Only one process in critical section at a time
2. **Progress:** If no one is in CS, a waiting process must be allowed to enter
3. **Bounded Waiting:** A process waiting to enter must eventually get in (no starvation)

---

### Synchronization Mechanisms

#### 1. Mutex (Mutual Exclusion Lock)

Simplest mechanism. A lock that can be either locked or unlocked.

```java
// Java
private final Lock lock = new ReentrantLock();

public void criticalSection() {
    lock.lock();        // Acquire lock
    try {
        // Access shared resource
        counter++;
    } finally {
        lock.unlock();  // Always release lock!
    }
}
```

```python
# Python
import threading

lock = threading.Lock()

def critical_section():
    with lock:  # Automatically acquires and releases
        # Access shared resource
        global counter
        counter += 1
```

```c
// C (pthreads)
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void* critical_section(void* arg) {
    pthread_mutex_lock(&lock);
    // Access shared resource
    pthread_mutex_unlock(&lock);
    return NULL;
}
```

**Properties:**
- Binary: locked or unlocked
- Only the thread that locked can unlock (ownership)
- If locked, other threads block (busy-wait or sleep)

#### 2. Semaphore

A semaphore is a counter that controls access to a shared resource.

**Binary Semaphore:** Like a mutex (0 or 1).
**Counting Semaphore:** Value can be any non-negative integer.

```java
// Java Semaphore
Semaphore semaphore = new Semaphore(3);  // Allow 3 concurrent accesses

public void accessResource() {
    semaphore.acquire();  // Decrements count (blocks if 0)
    try {
        // Access resource (up to 3 threads simultaneously)
    } finally {
        semaphore.release();  // Increments count
    }
}
```

```c
// C (pthreads)
sem_t semaphore;
sem_init(&semaphore, 0, 3);  // Initial value = 3

void* access_resource(void* arg) {
    sem_wait(&semaphore);    // P operation (decrement)
    // Access resource
    sem_post(&semaphore);    // V operation (increment)
    return NULL;
}
```

**Key difference from mutex:** Semaphore has no ownership—any thread can signal it. Mutex must be unlocked by the thread that locked it.

| Feature | Mutex | Semaphore |
|---------|-------|-----------|
| Ownership | Yes (locking thread must unlock) | No (any thread can signal) |
| Value | Binary (locked/unlocked) | Integer (0 to N) |
| Use case | Protect critical section | Control access to N resources |
| Reentrant | Can be (Java ReentrantLock) | No |

#### 3. Monitor

A high-level synchronization construct that combines mutual exclusion and condition variables.

```java
// Java's synchronized keyword uses monitors
public class SharedCounter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// Or with wait/notify
public class BoundedBuffer {
    private Queue<Integer> queue = new LinkedList<>();
    private int capacity = 10;
    
    public synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait();  // Release lock and wait
        }
        queue.add(item);
        notifyAll();  // Wake up waiting consumers
    }
    
    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();  // Release lock and wait
        }
        int item = queue.poll();
        notifyAll();  // Wake up waiting producers
        return item;
    }
}
```

#### 4. Condition Variables

Used with mutexes to wait for specific conditions.

```python
import threading

lock = threading.Lock()
condition = threading.Condition(lock)
items = []

def producer():
    with condition:
        items.append(1)
        condition.notify()  # Signal consumer

def consumer():
    with condition:
        while not items:
            condition.wait()  # Wait for producer
        item = items.pop()
```

---

### Classic Synchronization Problems

#### 1. Producer-Consumer Problem

A producer produces data; a consumer consumes it. They share a bounded buffer.

```java
class BoundedBuffer {
    private int[] buffer;
    private int in = 0, out = 0, count = 0;
    private Semaphore mutex = new Semaphore(1);
    private Semaphore empty;  // Tracks empty slots
    private Semaphore full;   // Tracks full slots
    
    public BoundedBuffer(int size) {
        buffer = new int[size];
        empty = new Semaphore(size);
        full = new Semaphore(0);
    }
    
    public void produce(int item) throws InterruptedException {
        empty.acquire();  // Wait for empty slot
        mutex.acquire();  // Enter critical section
        buffer[in] = item;
        in = (in + 1) % buffer.length;
        count++;
        mutex.release();  // Exit critical section
        full.release();   // Signal full slot
    }
    
    public int consume() throws InterruptedException {
        full.acquire();   // Wait for full slot
        mutex.acquire();  // Enter critical section
        int item = buffer[out];
        out = (out + 1) % buffer.length;
        count--;
        mutex.release();  // Exit critical section
        empty.release();  // Signal empty slot
        return item;
    }
}
```

#### 2. Readers-Writers Problem

Multiple readers can read simultaneously, but writers need exclusive access.

```java
class ReadersWriters {
    private int readers = 0;
    private Semaphore mutex = new Semaphore(1);      // Protects readers count
    private Semaphore writeLock = new Semaphore(1);   // Exclusive write access
    
    public void startRead() throws InterruptedException {
        mutex.acquire();
        readers++;
        if (readers == 1) {
            writeLock.acquire();  // First reader locks out writers
        }
        mutex.release();
        // Read
    }
    
    public void endRead() throws InterruptedException {
        mutex.acquire();
        readers--;
        if (readers == 0) {
            writeLock.release();  // Last reader allows writers
        }
        mutex.release();
    }
    
    public void startWrite() throws InterruptedException {
        writeLock.acquire();
        // Write
    }
    
    public void endWrite() throws InterruptedException {
        writeLock.release();
    }
}
```

**Problem:** Writers can starve if readers keep arriving.

#### 3. Dining Philosophers Problem

Five philosophers sit around a table. Each needs two forks to eat.

```
        P1
    F5      F1
  P5          P2
    F4      F2
        P3
```

**Solution with semaphore:**

```java
Semaphore[] forks = new Semaphore[5];
for (int i = 0; i < 5; i++) forks[i] = new Semaphore(1);

void philosopher(int i) {
    while (true) {
        think();
        forks[i].acquire();           // Pick up left fork
        forks[(i+1) % 5].acquire();   // Pick up right fork
        eat();
        forks[(i+1) % 5].release();   // Put down right fork
        forks[i].release();           // Put down left fork
    }
}
```

**Problem:** Deadlock if all philosophers pick up left fork simultaneously.
**Solutions:**
- Allow at most 4 philosophers to sit
- Pick up forks only if both available
- Asymmetric solution (odd picks left first, even picks right first)

---

### Busy Waiting vs Blocking

| Type | Description | Pros | Cons |
|------|-------------|------|------|
| **Busy Waiting** | Continuously check condition | Fast when wait is short | Wastes CPU |
| **Blocking** | Sleep until signaled | Doesn't waste CPU | Context switch overhead |

**Spinlock:** A mutex that uses busy waiting. Good for short waits on multiprocessor systems.

```c
// Spinlock implementation
volatile int lock = 0;

void acquire() {
    while (__sync_lock_test_and_set(&lock, 1)) {
        // Busy wait (spin)
    }
}

void release() {
    __sync_lock_release(&lock);
}
```

---

### Hardware Support for Synchronization

#### Test-and-Set

Atomic hardware instruction that tests and sets a value in one operation.

```c
// Hardware atomic operation
boolean test_and_set(boolean *target) {
    boolean old = *target;
    *target = TRUE;
    return old;
}

void acquire_lock(boolean *lock) {
    while (test_and_set(lock)) {
        // Busy wait
    }
}

void release_lock(boolean *lock) {
    *lock = FALSE;
}
```

#### Compare-and-Swap (CAS)

Another atomic instruction used in lock-free programming.

```c
// Hardware atomic operation
int compare_and_swap(int *value, int expected, int new_value) {
    int old = *value;
    if (old == expected) {
        *value = new_value;
    }
    return old;
}
```

**Used in:** Java's `AtomicInteger`, `ConcurrentHashMap`, lock-free data structures.

---

### Interview Questions

**Q: What's a race condition?**

A: "A race condition occurs when multiple threads access shared data concurrently, and the result depends on the order of execution. Example: two threads incrementing a counter might both read the same value and write the same result, losing an increment. Solutions include mutexes, semaphores, and atomic operations."

**Q: What's the difference between a mutex and a semaphore?**

A: "A mutex has ownership—the thread that locks it must unlock it. It's binary (locked/unlocked) and used for mutual exclusion. A semaphore has no ownership—any thread can signal it. It can count (0 to N) and is used for controlling access to multiple resources. Binary semaphores are similar to mutexes, but counting semaphores can allow N concurrent accesses."

**Q: What's a monitor?**

A: "A monitor is a high-level synchronization construct that combines a lock with condition variables. Only one thread can execute a monitor method at a time. Java's synchronized keyword implements monitors. Condition variables (wait/notify) allow threads to wait for specific conditions within the monitor."

**Q: Explain the Producer-Consumer problem and solution.**

A: "Producers create data and put it in a shared buffer; consumers take data from the buffer. The buffer has limited capacity. Solution: use semaphores—one tracking empty slots, one tracking full slots, and a mutex for mutual exclusion. Producers wait if buffer is full; consumers wait if buffer is empty."

**Q: What's the difference between busy waiting and blocking?**

A: "Busy waiting continuously checks a condition in a loop—wastes CPU but fast when wait is short. Blocking puts the thread to sleep until signaled—doesn't waste CPU but has context switch overhead. Spinlocks use busy waiting for very short critical sections on multiprocessor systems."

---

*Next: [06 — Deadlocks](06-Deadlocks.md)*
    