# 14 — System Calls

## The Bridge Between User and Kernel

---

### What is a System Call?

A system call is the mechanism by which a user program requests a service from the OS kernel. It's the **only way** to access hardware or privileged operations.

**Why?** User programs run in user mode with limited access. System calls switch to kernel mode to perform privileged operations.

---

### User Mode vs Kernel Mode

| Mode | Privileges | Can Do | Can't Do |
|------|-----------|--------|----------|
| **User Mode** | Limited | Run application code | Access hardware, modify page tables |
| **Kernel Mode** | Full | Everything | - |

```
User Program
    │
    ▼ System Call (trap)
┌──────────────────────────┐
│ Kernel Mode               │
│ - Validate arguments      │
│ - Perform operation       │
│ - Return result           │
└──────────────────────────┘
    │
    ▼ Return to User Mode
User Program continues
```

---

### System Call Flow

```
1. User program calls library function (e.g., write())
2. Library function puts syscall number in register
3. Library executes trap instruction (switch to kernel mode)
4. Kernel looks up syscall number in syscall table
5. Kernel executes the system call handler
6. Kernel returns result to user program
7. Library returns to user program
```

---

### Categories of System Calls

#### 1. Process Control
| Call | Description |
|------|-------------|
| `fork()` | Create new process |
| `exec()` | Execute a program |
| `exit()` | Terminate process |
| `wait()` | Wait for child process |
| `getpid()` | Get process ID |

#### 2. File Management
| Call | Description |
|------|-------------|
| `open()` | Open file |
| `close()` | Close file |
| `read()` | Read from file |
| `write()` | Write to file |
| `lseek()` | Reposition file pointer |
| `unlink()` | Delete file |

#### 3. Device Management
| Call | Description |
|------|-------------|
| `ioctl()` | Control device |
| `read()` | Read from device |
| `write()` | Write to device |

#### 4. Information Maintenance
| Call | Description |
|------|-------------|
| `time()` | Get time |
| `getpid()` | Get process ID |
| `alarm()` | Set alarm |

#### 5. Communication
| Call | Description |
|------|-------------|
| `pipe()` | Create pipe |
| `shmget()` | Shared memory |
| `mmap()` | Memory-mapped file |
| `socket()` | Create socket |

#### 6. Protection
| Call | Description |
|------|-------------|
| `chmod()` | Change permissions |
| `chown()` | Change owner |

---

### Common System Calls in Detail

#### fork()
```c
pid_t pid = fork();
// Returns: 0 in child, child's PID in parent, -1 on error

if (pid == 0) {
    // Child process
} else if (pid > 0) {
    // Parent process
} else {
    // Error
}
```

#### exec()
```c
// Replace current process with new program
execl("/bin/ls", "ls", "-l", NULL);
// If successful, doesn't return (process is replaced)
```

#### open() and read()
```c
int fd = open("file.txt", O_RDONLY);
if (fd == -1) {
    perror("open failed");
    exit(1);
}

char buffer[100];
int bytes = read(fd, buffer, 100);
if (bytes == -1) {
    perror("read failed");
}

close(fd);
```

#### write()
```c
// Write to stdout
char *msg = "Hello, World!\n";
write(STDOUT_FILENO, msg, strlen(msg));
```

#### pipe()
```c
int pipefd[2];
pipe(pipefd);  // pipefd[0]=read end, pipefd[1]=write end

if (fork() == 0) {
    // Child: write to pipe
    close(pipefd[0]);
    write(pipefd[1], "hello", 5);
    close(pipefd[1]);
} else {
    // Parent: read from pipe
    close(pipefd[1]);
    char buffer[10];
    read(pipefd[0], buffer, 5);
    close(pipefd[0]);
}
```

---

### What Happens When You Type `ls`

```
1. Terminal reads keystrokes
2. Shell parses command "ls"
3. Shell calls fork() → creates child process
4. Child calls exec("/bin/ls") → replaces with ls program
5. ls calls open(".") → opens current directory
6. ls calls getdents() → reads directory entries
7. ls calls write() → prints file names to stdout
8. ls calls exit() → terminates
9. Shell calls wait() → reaps child
10. Shell displays prompt again
```

---

### System Call vs Library Function

| Aspect | System Call | Library Function |
|--------|-------------|------------------|
| **Mode** | Kernel mode | User mode |
| **Overhead** | High (context switch) | Low |
| **Example** | `read()`, `write()` | `printf()`, `malloc()` |
| **Implementation** | In kernel | In libc |

**`printf()` calls `write()` internally**—it's a library function that makes a system call.

---

### System Call Overhead

```
User program → Trap to kernel → Execute syscall → Return to user

Cost: ~100-1000 nanoseconds (context switch + kernel execution)
```

**Why system calls are expensive:**
1. Mode switch (user → kernel → user)
2. Save/restore registers
3. Validate arguments
4. Potential TLB flush

---

### Interview Questions

**Q: What is a system call?**

A: "A system call is the mechanism for user programs to request services from the OS kernel. It switches from user mode to kernel mode, performs the requested operation, and returns to user mode. Examples: file I/O, process creation, memory allocation. System calls are the only way to access hardware or privileged operations."

**Q: What's the difference between a system call and a library function?**

A: "System calls invoke kernel code—they require a mode switch (user→kernel→user) and are expensive. Library functions run in user space—they're fast but eventually may make system calls. Example: printf() is a library function that buffers output and calls write() system call when needed."

**Q: What happens during a context switch for a system call?**

A: "When a system call occurs: (1) CPU switches to kernel mode, (2) saves user process state (registers, PC), (3) jumps to syscall handler, (4) executes kernel code, (5) restores user state, (6) switches back to user mode. The overhead is typically 100-1000 nanoseconds."

**Q: What's the difference between user mode and kernel mode?**

A: "User mode has limited privileges—can't access hardware, modify page tables, or execute privileged instructions. Kernel mode has full access. The CPU has a mode bit: 0 for kernel, 1 for user. System calls are the controlled entry point to kernel mode."

**Q: Give examples of common system calls.**

A: "Process control: fork(), exec(), exit(), wait(). File management: open(), read(), write(), close(). Device management: ioctl(). Communication: pipe(), socket(), shmget(). Information: time(), getpid()."

---

*Next: [15 — Inter-Process Communication](15-IPC.md)*
