# Operating Systems Interview Handbook

## The Complete Guide to Cracking OS Interviews

---

### Why OS Matters for Interviews

Every top company—Google, Amazon, Microsoft, Meta, Apple—tests OS fundamentals. Not just definitions. They want you to **understand how systems work**, **explain trade-offs**, and **connect concepts to real-world scenarios**.

OS knowledge separates candidates who write code from candidates who understand systems.

---

### What Interviewers Actually Ask

They won't ask "Define a process." That's a textbook question.

They'll ask:

> "What happens when you type `ls` in a terminal?"
> "Why do we need both virtual memory and physical memory?"
> "Explain the difference between a process and a thread. When would you use each?"
> "How does the OS prevent deadlocks?"
> "What happens during a context switch?"
> "Explain paging and how it handles page faults."

**This book prepares you for those questions.**

---

### The 16 Chapters

| # | Chapter | Why It Matters |
|---|---------|----------------|
| 1 | [What is an Operating System?](01-What-is-an-OS.md) | Foundation—understand the big picture |
| 2 | [Processes](02-Processes.md) | The fundamental unit of execution |
| 3 | [Threads](03-Threads.md) | Lightweight execution units |
| 4 | [CPU Scheduling](04-CPU-Scheduling.md) | How the OS decides what runs next |
| 5 | [Synchronization](05-Synchronization.md) | Managing concurrent access |
| 6 | [Deadlocks](06-Deadlocks.md) | When everything stops |
| 7 | [Memory Management](07-Memory-Management.md) | How memory is allocated and tracked |
| 8 | [Virtual Memory](08-Virtual-Memory.md) | The illusion of infinite memory |
| 9 | [Paging](09-Paging.md) | The backbone of virtual memory |
| 10 | [Segmentation](10-Segmentation.md) | Memory organization by purpose |
| 11 | [File Systems](11-File-Systems.md) | How data is stored and retrieved |
| 12 | [I/O Systems](12-IO-Systems.md) | How the OS handles devices |
| 13 | [Disk Scheduling](13-Disk-Scheduling.md) | Optimizing disk access |
| 14 | [System Calls](14-System-Calls.md) | The bridge between user and kernel |
| 15 | [Inter-Process Communication](15-IPC.md) | How processes talk to each other |
| 16 | [OS Interview Q&A](16-Interview-QA.md) | Real questions, real answers |

---

### How to Use This Book

**Week 1:** Chapters 1-6 (Core Concepts)
- Processes, Threads, CPU Scheduling
- Synchronization and Deadlocks
- These are the most asked topics

**Week 2:** Chapters 7-11 (Memory & Storage)
- Memory Management, Virtual Memory, Paging
- File Systems
- Understand the memory hierarchy

**Week 3:** Chapters 12-16 (Advanced & Application)
- I/O, Disk Scheduling, System Calls
- IPC
- Practice interview Q&A

---

### The Mental Model

```
                    User Programs
                         │
                    System Calls
                         │
                    ┌────┴────┐
                    │  Kernel  │
                    └────┬────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   Process Mgmt    Memory Mgmt      I/O Mgmt
        │                │                │
   - Processes      - Paging         - Devices
   - Threads        - Virtual Mem    - File Systems
   - Scheduling     - Allocation     - Disk Scheduling
   - Sync           - Segmentation
   - Deadlocks
```

---

### Progress Tracker

```
[ ] Chapter 1: What is an OS
[ ] Chapter 2: Processes
[ ] Chapter 3: Threads
[ ] Chapter 4: CPU Scheduling
[ ] Chapter 5: Synchronization
[ ] Chapter 6: Deadlocks
[ ] Chapter 7: Memory Management
[ ] Chapter 8: Virtual Memory
[ ] Chapter 9: Paging
[ ] Chapter 10: Segmentation
[ ] Chapter 11: File Systems
[ ] Chapter 12: I/O Systems
[ ] Chapter 13: Disk Scheduling
[ ] Chapter 14: System Calls
[ ] Chapter 15: IPC
[ ] Chapter 16: Interview Q&A
```

---

*Built for freshers who want to crack top company interviews.*
