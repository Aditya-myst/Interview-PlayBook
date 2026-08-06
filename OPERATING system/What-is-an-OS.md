# 01 — What is an Operating System?

## The Foundation of Everything

---

### Definition

An Operating System (OS) is **system software that manages computer hardware and software resources** and provides common services for application programs.

**In simple terms:** The OS is the middleman between you and the hardware. When you click "Save," the OS figures out how to write bytes to a specific location on your disk.

---

### What the OS Does

```
User Applications (Browser, Editor, Games)
            │
    ┌───────┴───────┐
    │  Operating     │
    │  System        │
    │  (Kernel)      │
    └───────┬───────┘
            │
    ┌───────┴───────┐
    │   Hardware     │
    │ (CPU, RAM,     │
    │  Disk, I/O)    │
    └───────────────┘
```

### The Four Core Functions

| Function | What It Does | Example |
|----------|--------------|---------|
| **Process Management** | Create, schedule, terminate processes | Running Chrome, VS Code, Terminal |
| **Memory Management** | Allocate and track RAM usage | Giving Chrome 2GB, VS Code 512MB |
| **File System Management** | Store and retrieve data on disk | Reading/writing files, directories |
| **I/O Management** | Handle input/output devices | Keyboard, mouse, screen, network |

---

### Types of Operating Systems

| Type | Description | Example |
|------|-------------|---------|
| **Batch OS** | Executes jobs in batches, no interaction | Old mainframes |
| **Time-Sharing OS** | Multiple users share CPU via time slices | Unix, Linux |
| **Real-Time OS** | Guaranteed response within deadline | Embedded systems, medical devices |
| **Distributed OS** | Manages multiple networked computers | Google's Borg, Apache Mesos |
| **Embedded OS** | Runs on specialized hardware | IoT devices, routers |

---

### Kernel: The Heart of the OS

The **kernel** is the core component of the OS that has complete control over the system.

```
┌─────────────────────────────────────┐
│           User Space                │
│  ┌──────────┐  ┌──────────┐        │
│  │ Process A │  │ Process B │        │
│  └────┬─────┘  └────┬─────┘        │
│       │              │              │
├───────┼──────────────┼──────────────┤
│       │    System    │              │
│       │    Calls     │              │
│  ┌────┴─────────────┴────┐         │
│  │     Kernel Space       │         │
│  │  ┌──────────────────┐ │         │
│  │  │ Process Scheduler │ │         │
│  │  │ Memory Manager    │ │         │
│  │  │ File System       │ │         │
│  │  │ Device Drivers    │ │         │
│  │  └──────────────────┘ │         │
│  └───────────────────────┘         │
├─────────────────────────────────────┤
│           Hardware                  │
│   CPU    RAM    Disk    I/O        │
└─────────────────────────────────────┘
```

**User Space:** Where applications run. Limited privileges.
**Kernel Space:** Where the OS runs. Full access to hardware.

---

### Kernel Architectures

| Architecture | Description | Pros | Cons |
|--------------|-------------|------|------|
| **Monolithic** | All OS services in kernel space | Fast | Bug in one module crashes everything |
| **Microkernel** | Minimal kernel, services in user space | Stable, modular | Slower (more context switches) |
| **Hybrid** | Mix of both | Balance | Complexity |

**Linux** = Monolithic (with loadable modules)
**Windows** = Hybrid
**macOS/iOS** = Hybrid (XNU kernel)

---

### What Happens When You Run a Program

```
1. You type ./program in terminal
2. Shell calls fork() system call → creates new process
3. Shell calls exec() system call → loads program into memory
4. OS allocates memory (virtual address space)
5. OS loads code and data from disk to memory
6. OS creates process control block (PCB)
7. OS adds process to ready queue
8. CPU scheduler picks the process
9. CPU executes the program
10. Program makes system calls for I/O, file access, etc.
11. Program calls exit() → OS reclaims resources
```

---

### Interview Questions

**Q: What is an operating system?**

A: "An OS is system software that manages hardware and software resources. It provides services like process management, memory management, file systems, and I/O handling. The kernel is the core component with full hardware access, while applications run in user space with limited privileges."

**Q: What's the difference between kernel space and user space?**

A: "Kernel space is where the OS kernel runs with full access to hardware and memory. User space is where applications run with restricted access. Applications must use system calls to request kernel services. This separation protects the system from malicious or buggy applications."

**Q: What's the difference between a monolithic and microkernel?**

A: "A monolithic kernel runs all OS services in kernel space—it's fast but a bug in any module can crash the system. A microkernel runs only essential services in kernel space and moves everything else to user space—it's more stable but slower due to more context switches. Linux is monolithic; MINIX is microkernel."

---

*Next: [02 — Processes](02-Processes.md)*
