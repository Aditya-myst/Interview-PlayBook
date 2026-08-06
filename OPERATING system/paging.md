# 09 — Paging

## The Backbone of Virtual Memory

---

### What is Paging?

Paging divides physical memory into fixed-size blocks called **frames** and logical memory into blocks of the same size called **pages**. The OS maintains a **page table** that maps virtual pages to physical frames.

```
Virtual Memory:              Physical Memory:
┌──────────────┐            ┌──────────────┐
│ Page 0       │───────────→│ Frame 2      │
├──────────────┤            ├──────────────┤
│ Page 1       │───────────→│ Frame 5      │
├──────────────┤            ├──────────────┤
│ Page 2       │───→ DISK   │ Frame 0      │ ← Page 3
├──────────────┤            ├──────────────┤
│ Page 3       │───────────→│ Frame 1      │ ← Page 4
├──────────────┤            ├──────────────┤
│ Page 4       │───────────→│ Frame 3      │
└──────────────┘            └──────────────┘
```

**Key properties:**
- No external fragmentation (all frames same size)
- Some internal fragmentation (last page may not be full)
- Page size typically 4KB (4096 bytes)

---

### Address Translation

A virtual address is split into **page number** and **page offset**.

```
Virtual Address (32-bit, 4KB pages):
┌─────────────────────┬──────────────┐
│   Page Number (20)  │ Offset (12)  │
└─────────────────────┴──────────────┘

Page number = virtual address / page size
Offset = virtual address % page size

Example: Virtual address 0x3A4F
Page size = 4KB = 4096 = 2^12

Page number = 0x3A4F / 0x1000 = 0x3
Offset = 0x3A4F % 0x1000 = 0xA4F

Page table lookup: Page 3 → Frame 1
Physical address = Frame 1 × 4096 + 0xA4F = 0x1A4F
```

---

### Page Table Structure

#### Single-Level Page Table

```
Virtual Address:
┌──────────────┬──────────┐
│ Page Number  │  Offset  │
└──────┬───────┴──────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│ Page Table                               │
│ ┌──────┬──────┬──────┐                  │
│ │ Page │ Frame│ Valid│                  │
│ ├──────┼──────┼──────┤                  │
│ │  0   │  2   │  1   │ → In memory     │
│ │  1   │  5   │  1   │ → In memory     │
│ │  2   │  -   │  0   │ → On disk       │
│ │  3   │  0   │  1   │ → In memory     │
│ │  4   │  3   │  1   │ → In memory     │
│ └──────┴──────┴──────┘                  │
└──────────────┬──────────────────────────┘
               │
               ▼
Physical Address:
┌──────────────┬──────────┐
│ Frame Number │  Offset  │
└──────────────┴──────────┘
```

**Problem:** 32-bit address space with 4KB pages = 2^20 entries × 4 bytes = 4MB per page table! Too large.

#### Two-Level Page Table

Split the page table into two levels to save space.

```
Virtual Address:
┌──────────┬──────────┬──────────┐
│ P1 (10)  │ P2 (10)  │ Offset(12)│
└────┬─────┴────┬─────┴──────────┘
     │          │
     ▼          │
┌──────────┐    │
│Outer Page│    │
│Table     │    │
│(1024)    │    │
└────┬─────┘    │
     │          │
     ▼          ▼
┌──────────┐
│Inner Page│
│Table     │
│(1024)    │
└────┬─────┘
     │
     ▼
Physical Address
```

**Benefit:** Only the parts of the page table actually used need to be in memory.

#### Inverted Page Table

One entry per physical frame (not per virtual page).

```
Physical Frame → (Process ID, Virtual Page)

Search by physical frame number.
Need to search for (PID, virtual page) to find frame.
Use hash table for O(1) lookup.
```

**Benefit:** Fixed size regardless of virtual address space.
**Used by:** PowerPC, UltraSPARC.

---

### Translation Lookaside Buffer (TLB)

A hardware cache that stores recent page table entries.

```
CPU: Virtual Address
        │
        ▼
┌───────────────┐
│     TLB       │ ← Fast lookup (1 clock cycle)
│  (Cache of    │
│   page table  │
│   entries)    │
└───────┬───────┘
        │
   TLB Hit? ──Yes──→ Physical address (fast!)
        │
        No (TLB Miss)
        │
        ▼
   Page Table Walk (slow!)
        │
        ▼
   Physical address
   (also store in TLB)
```

**TLB characteristics:**
| Feature | Typical Value |
|---------|---------------|
| Size | 64-1024 entries |
| Hit time | 1 clock cycle |
| Miss penalty | 10-100 clock cycles |
| Hit rate | 99%+ (due to locality) |

**TLB Reach:** How much memory TLB can map = entries × page size.
- 1024 entries × 4KB = 4MB
- Use larger pages (2MB, 1GB) for larger TLB reach

---

### Multi-Level TLB

```
CPU → L1 TLB (fast, small) → L2 TLB (slower, larger) → Page Table (slow)
```

---

### Protection Bits

Each page table entry has protection bits:

| Bit | Meaning |
|-----|---------|
| **Valid** | Page is in memory |
| **Read/Write** | Read-only or read-write |
| **User/Supervisor** | User or kernel access only |
| **Dirty** | Page has been modified |
| **Referenced** | Page has been accessed |

```
Page Table Entry:
┌────────┬─────┬──────┬─────┬────────┬──────────────┐
│ Frame #│Valid│R/W   │User │Dirty   │Referenced    │
└────────┴─────┴──────┴─────┴────────┴──────────────┘
```

---

### Shared Pages

Multiple processes can share the same physical frames (for code, shared libraries).

```
Process A Page Table:     Process B Page Table:
Page 0 → Frame 2         Page 0 → Frame 2  (shared!)
Page 1 → Frame 5         Page 1 → Frame 7
Page 2 → Frame 3         Page 2 → Frame 3  (shared!)
```

**Use cases:** Shared libraries (libc.so), shared memory, copy-on-write.

---

### Page Size Trade-offs

| Smaller Pages (4KB) | Larger Pages (2MB/1GB) |
|---------------------|------------------------|
| Less internal fragmentation | More internal fragmentation |
| Larger page tables | Smaller page tables |
| More TLB misses | Fewer TLB misses |
| More page faults | Fewer page faults |
| Better for small allocations | Better for large workloads |

**Modern systems:** Support multiple page sizes (4KB, 2MB, 1GB).

---

### Interview Questions

**Q: What is paging?**

A: "Paging divides physical memory into fixed-size frames and logical memory into pages of the same size. The OS maintains a page table mapping virtual pages to physical frames. This eliminates external fragmentation and enables virtual memory—pages not in use can be stored on disk."

**Q: What's the TLB and why is it important?**

A: "The TLB is a hardware cache storing recent page table translations. Without TLB, every memory access would require a page table lookup (slow). TLB hit rate is typically 99%+, making address translation fast. TLB miss triggers a page table walk, which is expensive."

**Q: What's a multi-level page table?**

A: "A page table hierarchy that saves space by not allocating entries for unused virtual address ranges. A two-level page table splits the page number into two parts: the first indexes the outer table, the second indexes the inner table. Only the parts actually used need to be in memory."

**Q: What's an inverted page table?**

A: "Instead of one entry per virtual page, there's one entry per physical frame. Each entry stores which process and virtual page maps to that frame. This saves space (fixed size regardless of virtual address space) but requires searching. A hash table is used for O(1) lookup."

**Q: Explain page table entry protection bits.**

A: "Each page table entry contains: Valid bit (is page in memory?), Read/Write bit (read-only or read-write?), User/Supervisor bit (user or kernel access?), Dirty bit (has page been modified?), Referenced bit (has page been accessed recently?). These bits enable memory protection and support page replacement algorithms."

---

*Next: [10 — Segmentation](10-Segmentation.md)*
