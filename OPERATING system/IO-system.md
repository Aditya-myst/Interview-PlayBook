# 12 — I/O Systems

## How the OS Handles Devices

---

### I/O Hardware

```
┌─────────────────────────────────────────────────────┐
│                    CPU                               │
│  ┌──────────┐    ┌──────────┐                      │
│  │Registers │    │   ALU    │                      │
│  └────┬─────┘    └──────────┘                      │
│       │                                             │
│  ┌────┴──────────────────────────────────────────┐ │
│  │              System Bus                        │ │
│  └────┬─────────┬─────────┬─────────┬────────────┘ │
└───────┼─────────┼─────────┼─────────┼──────────────┘
        │         │         │         │
   ┌────┴───┐ ┌───┴───┐ ┌───┴───┐ ┌───┴───┐
   │ Memory │ │ Disk  │ │ Net   │ │ Other │
   │        │ │Controller│Controller│Devices│
   └────────┘ └───────┘ └───────┘ └───────┘
```

---

### I/O Methods

#### 1. Programmed I/O (Polling)

CPU continuously checks device status.

```
CPU: "Is device ready?" → No
CPU: "Is device ready?" → No
CPU: "Is device ready?" → Yes
CPU: Transfer data
CPU: "Is device ready?" → No
... (CPU busy-waiting, wasted!)
```

**Pros:** Simple
**Cons:** CPU wasted polling

#### 2. Interrupt-Driven I/O

Device interrupts CPU when ready.

```
CPU: Start I/O, do other work
    ... (CPU doing useful work) ...
Device: "I'm ready!" → Interrupt
CPU: Save state, handle interrupt, transfer data
CPU: Resume previous work
```

**Pros:** CPU not wasted polling
**Cons:** One interrupt per character/block (overhead for large transfers)

#### 3. Direct Memory Access (DMA)

Device transfers data directly to/from memory without CPU involvement.

```
CPU: Tell DMA controller: "Transfer N bytes from device to memory at address X"
CPU: Do other work
    ... (DMA handles transfer) ...
DMA: "Done!" → Interrupt
CPU: Process data
```

**Pros:** CPU free during transfer, efficient for large transfers
**Cons:** Complex hardware

---

### I/O Buffering

Buffers hold data during transfer between device and application.

| Buffer Type | Description | Use Case |
|-------------|-------------|----------|
| **Single Buffer** | One buffer, alternate between user and OS | Simple |
| **Double Buffer** | Two buffers, one being filled while other is emptied | Overlap I/O and computation |
| **Circular Buffer** | Multiple buffers in a ring | Streaming data |

---

### Device Drivers

Software that communicates with specific hardware.

```
Application: write(fd, data, size)
    │
    ▼
File System: Convert to block operations
    │
    ▼
Device Driver: Translate to device commands
    │
    ▼
Hardware Controller: Execute commands
    │
    ▼
Physical Device
```

**Device drivers are OS-specific**—a Windows driver won't work on Linux.

---

### Blocking vs Non-Blocking I/O

| Type | Description | Use Case |
|------|-------------|----------|
| **Blocking** | Process waits until I/O completes | Simple programs |
| **Non-blocking** | Process continues, checks later | Interactive applications |
| **Asynchronous** | Process continues, notified when complete | High-performance servers |

```c
// Blocking I/O
int bytes = read(fd, buffer, 100);  // Blocks until data available

// Non-blocking I/O
fcntl(fd, F_SETFL, O_NONBLOCK);
int bytes = read(fd, buffer, 100);  // Returns -1 if no data
```

---

### I/O Scheduling

The OS reorders I/O requests to minimize disk seek time.

```
Requests: 98, 183, 37, 122, 14, 124, 65, 67
Current position: 53

FCFS: 53→98→183→37→122→14→124→65→67 (640 cylinders)
SSTF: 53→65→67→37→14→98→122→124→183 (236 cylinders)
SCAN: 53→37→14→0→65→67→98→122→124→183 (236 cylinders)
```

---

### Interview Questions

**Q: What's the difference between polling and interrupts?**

A: "Polling: CPU continuously checks if device is ready—wastes CPU cycles. Interrupts: device signals CPU when ready—CPU does other work meanwhile. Interrupts are more efficient but have overhead (context switch). DMA is even better: device transfers data directly to memory without CPU."

**Q: What is DMA?**

A: "Direct Memory Access allows devices to transfer data directly to/from memory without CPU involvement. CPU sets up the transfer (source, destination, size), then DMA controller handles it. When done, DMA interrupts CPU. This is efficient for large transfers like disk reads."

**Q: What's the difference between blocking and non-blocking I/O?**

A: "Blocking I/O: process waits until operation completes—simple but wastes CPU if device is slow. Non-blocking I/O: operation returns immediately with whatever data is available—process must poll or use select/epoll. Asynchronous I/O: process continues, gets notified when complete—most efficient but complex."

**Q: What's a device driver?**

A: "Software that translates OS commands into device-specific operations. Each device type has a driver. The OS communicates with drivers through a standard interface; drivers handle device-specific details. This abstraction lets the OS work with any device that has a driver."

---

*Next: [13 — Disk Scheduling](13-Disk-Scheduling.md)*
