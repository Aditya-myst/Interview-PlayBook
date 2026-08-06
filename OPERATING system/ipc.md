# 15 — Inter-Process Communication (IPC)

## How Processes Talk to Each Other

---

### Why IPC?

Processes have separate address spaces. They can't directly access each other's memory. IPC provides mechanisms for processes to communicate and synchronize.

---

### IPC Mechanisms

```
┌─────────────────────────────────────────────────────────┐
│                    IPC Methods                           │
├─────────────────┬─────────────────┬─────────────────────┤
│ Shared Memory   │ Message Passing │ Signal              │
├─────────────────┼─────────────────┼─────────────────────┤
│ Pipes           │ Sockets         │ Memory-Mapped Files │
│ Named Pipes     │ Message Queues  │ Semaphores          │
└─────────────────┴─────────────────┴─────────────────────┘
```

---

### 1. Pipes

Unidirectional communication channel between related processes.

```
Process A ──write──> [pipe buffer] ──read──> Process B
```

#### Anonymous Pipes
```c
int pipefd[2];
pipe(pipefd);

if (fork() == 0) {
    // Child: read from pipe
    close(pipefd[1]);  // Close write end
    char buffer[100];
    read(pipefd[0], buffer, 100);
    close(pipefd[0]);
} else {
    // Parent: write to pipe
    close(pipefd[0]);  // Close read end
    write(pipefd[1], "Hello", 5);
    close(pipefd[1]);
}
```

**Limitations:**
- Unidirectional (need two pipes for bidirectional)
- Only works between related processes (parent-child)
- Fixed buffer size (typically 64KB on Linux)

#### Named Pipes (FIFOs)
```bash
# Create named pipe
mkfifo /tmp/mypipe

# Process A writes
echo "Hello" > /tmp/mypipe

# Process B reads
cat /tmp/mypipe
```

**Advantage:** Works between unrelated processes.

---

### 2. Message Queues

Messages are stored in a queue. Processes can send and receive messages.

```c
// Sender
int mqid = msgget(key, IPC_CREAT | 0666);
struct msgbuf {
    long mtype;
    char mtext[100];
};
struct msgbuf msg = {1, "Hello"};
msgsnd(mqid, &msg, sizeof(msg.mtext), 0);

// Receiver
msgrcv(mqid, &msg, sizeof(msg.mtext), 1, 0);
printf("%s\n", msg.mtext);
```

**Properties:**
- Messages have types (selective receive)
- Asynchronous (sender doesn't wait)
- Messages queued in kernel

---

### 3. Shared Memory

Fastest IPC method. Processes share a region of memory.

```c
// Create shared memory
int shmid = shmget(IPC_PRIVATE, 4096, IPC_CREAT | 0666);

// Attach to process
int *shared = (int*)shmat(shmid, NULL, 0);

// Write
*shared = 42;

// Read (in another process after fork)
printf("%d\n", *shared);

// Detach
shmdt(shared);

// Remove
shmctl(shmid, IPC_RMID, NULL);
```

**With synchronization (mutex):**
```c
typedef struct {
    int data;
    pthread_mutex_t mutex;
} SharedData;

SharedData *shared = shmat(shmid, NULL, 0);

// Process A
pthread_mutex_lock(&shared->mutex);
shared->data = 42;
pthread_mutex_unlock(&shared->mutex);

// Process B
pthread_mutex_lock(&shared->mutex);
printf("%d\n", shared->data);
pthread_mutex_unlock(&shared->mutex);
```

**Properties:**
- Fastest (no kernel copy after setup)
- Need explicit synchronization
- Can be complex to use correctly

---

### 4. Sockets

Bidirectional communication, works across networks.

```c
// Server
int server_fd = socket(AF_INET, SOCK_STREAM, 0);
struct sockaddr_in addr = {
    .sin_family = AF_INET,
    .sin_port = htons(8080),
    .sin_addr.s_addr = INADDR_ANY
};
bind(server_fd, (struct sockaddr*)&addr, sizeof(addr));
listen(server_fd, 5);

int client_fd = accept(server_fd, NULL, NULL);
char buffer[100];
read(client_fd, buffer, 100);
write(client_fd, "Hello", 5);
close(client_fd);
close(server_fd);

// Client
int sock = socket(AF_INET, SOCK_STREAM, 0);
struct sockaddr_in addr = {
    .sin_family = AF_INET,
    .sin_port = htons(8080),
    .sin_addr.s_addr = inet_addr("127.0.0.1")
};
connect(sock, (struct sockaddr*)&addr, sizeof(addr));
write(sock, "Hi", 2);
read(sock, buffer, 100);
close(sock);
```

**Types:**
| Type | Description |
|------|-------------|
| **Stream (TCP)** | Reliable, ordered, connection-oriented |
| **Datagram (UDP)** | Unreliable, unordered, connectionless |
| **Unix Domain** | Same machine, faster than TCP |

---

### 5. Signals

Asynchronous notifications sent to processes.

```c
// Send signal
kill(pid, SIGUSR1);

// Handle signal
void handler(int sig) {
    printf("Received signal %d\n", sig);
}
signal(SIGUSR1, handler);
```

**Common signals:**
| Signal | Description | Default Action |
|--------|-------------|----------------|
| SIGINT | Ctrl+C | Terminate |
| SIGKILL | Force kill (can't catch) | Terminate |
| SIGSEGV | Segmentation fault | Terminate + core dump |
| SIGCHLD | Child process terminated | Ignore |
| SIGUSR1 | User-defined | Terminate |

---

### 6. Memory-Mapped Files

Map a file into memory. Multiple processes can map the same file.

```c
// Process A
int fd = open("shared.dat", O_RDWR | O_CREAT, 0666);
ftruncate(fd, sizeof(int));
int *data = mmap(NULL, sizeof(int), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
*data = 42;

// Process B (same or different process)
int fd = open("shared.dat", O_RDWR);
int *data = mmap(NULL, sizeof(int), PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
printf("%d\n", *data);  // Prints 42
```

---

### Comparison Table

| Method | Speed | Complexity | Scope | Synchronization |
|--------|-------|------------|-------|-----------------|
| **Pipes** | Medium | Simple | Related processes | Built-in (blocking) |
| **Message Queues** | Medium | Medium | Any process | Built-in |
| **Shared Memory** | Fastest | Complex | Any process | Manual needed |
| **Sockets** | Slowest | Complex | Any machine | Protocol-dependent |
| **Signals** | Fast | Simple | Any process | Async |
| **Memory-Mapped** | Fast | Medium | Any process | Manual needed |

---

### Choosing the Right IPC

```
Need to communicate?
    │
    ├─ Same machine, related processes?
    │   └─ Pipes (simple) or shared memory (fast)
    │
    ├─ Same machine, unrelated processes?
    │   └─ Named pipes, message queues, or shared memory
    │
    ├─ Different machines?
    │   └─ Sockets (TCP/UDP)
    │
    └─ Just notify (no data)?
        └─ Signals
```

---

### Interview Questions

**Q: What IPC mechanisms are available in Linux?**

A: "Pipes (anonymous and named), message queues, shared memory, sockets, signals, and memory-mapped files. Pipes are simplest for related processes. Shared memory is fastest but needs manual synchronization. Sockets work across networks. Signals are for async notifications."

**Q: What's the difference between pipes and message queues?**

A: "Pipes are byte streams—no message boundaries, unidirectional, typically between parent-child. Message queues are message-oriented—each message has a type and length, can be selectively received, and work between unrelated processes. Message queues are more flexible but have more overhead."

**Q: Why is shared memory the fastest IPC?**

A: "After setup, data transfer happens through direct memory access—no kernel copy. With pipes, data is copied from sender's space to kernel buffer to receiver's space (two copies). With shared memory, both processes access the same physical pages—zero copies."

**Q: How do you synchronize shared memory access?**

A: "Use semaphores or mutexes in the shared memory region. A pthread_mutex_t initialized with PTHREAD_PROCESS_SHARED can be used between processes. Alternatively, use semaphores (semget/semop). Without synchronization, you'll have race conditions."

**Q: What's the difference between TCP and UDP sockets?**

A: "TCP: reliable, ordered, connection-oriented—guarantees delivery, but slower. UDP: unreliable, unordered, connectionless—faster but packets can be lost or arrive out of order. Use TCP for file transfer, HTTP. Use UDP for gaming, video streaming, DNS."

---

*Next: [16 — OS Interview Q&A](16-Interview-QA.md)*
