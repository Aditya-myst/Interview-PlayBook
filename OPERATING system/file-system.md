# 11 — File Systems

## How Data is Stored and Retrieved

---

### What is a File System?

A file system is the method and data structure the OS uses to organize, store, retrieve, and manage data on storage devices.

**Without a file system:** Just a long sequence of bytes on disk.
**With a file system:** Organized hierarchy of files and directories.

---

### File System Layers

```
┌──────────────────────────────────┐
│     Application Programs         │
├──────────────────────────────────┤
│     Logical File System          │  ← File names, directories
├──────────────────────────────────┤
│     File Organization Module     │  ← Logical to physical mapping
├──────────────────────────────────┤
│     Basic File System            │  ← Block I/O
├──────────────────────────────────┤
│     Device Drivers               │  ← Hardware communication
├──────────────────────────────────┤
│     Hardware                     │
└──────────────────────────────────┘
```

---

### File Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| **Name** | Human-readable identifier | `report.pdf` |
| **Type** | File type | Text, binary, executable |
| **Size** | Current size | 1024 bytes |
| **Location** | Pointer to file on disk | Block 100-105 |
| **Protection** | Access permissions | rwxr-xr-x |
| **Timestamps** | Creation, modification, access | 2024-01-15 10:30 |

---

### File Operations

| Operation | System Call | Description |
|-----------|-------------|-------------|
| Create | `creat()` | Create new file |
| Open | `open()` | Get file descriptor |
| Read | `read()` | Read data from file |
| Write | `write()` | Write data to file |
| Seek | `lseek()` | Change file position |
| Close | `close()` | Release file descriptor |
| Delete | `unlink()` | Remove file |
| Truncate | `truncate()` | Reduce file size |

```c
// File operations in C
int fd = open("file.txt", O_RDONLY);  // Open for reading
char buffer[100];
int bytes = read(fd, buffer, 100);    // Read 100 bytes
lseek(fd, 0, SEEK_SET);              // Seek to beginning
close(fd);                            // Close file
```

---

### Directory Structure

#### Single-Level Directory
```
/root/
├── file1.txt
├── file2.txt
└── file3.txt
```
**Problem:** All files in one directory. Name conflicts.

#### Two-Level Directory
```
/root/
├── User1/
│   ├── file1.txt
│   └── file2.txt
└── User2/
    ├── file1.txt  ← Different from User1's file1.txt
    └── file3.txt
```

#### Tree-Structured Directory
```
/
├── home/
│   ├── alice/
│   │   ├── documents/
│   │   │   └── report.pdf
│   │   └── photos/
│   │       └── vacation.jpg
│   └── bob/
│       └── code/
│           └── main.c
├── etc/
│   └── config.txt
└── tmp/
```

#### Acyclic Graph Directory (with links)
```
/home/alice/documents/report.pdf
        ↑ (hard link)
/home/bob/shared/report.pdf
```

---

### File Allocation Methods

#### 1. Contiguous Allocation

Each file occupies a contiguous set of blocks.

```
File A: blocks 0-3
File B: blocks 5-7
File C: blocks 10-12

Directory:
┌────────┬───────┬───────┐
│ File   │ Start │ Length│
├────────┼───────┼───────┤
│ A      │ 0     │ 4     │
│ B      │ 5     │ 3     │
│ C      │ 10    │ 3     │
└────────┴───────┴───────┘
```

**Pros:** Fast sequential and random access.
**Cons:** External fragmentation, file size must be known at creation.

#### 2. Linked Allocation

Each file is a linked list of blocks.

```
File A: 0 → 4 → 2 → 7

Block 0: [data | next=4]
Block 4: [data | next=2]
Block 2: [data | next=7]
Block 7: [data | next=-1]  (end of file)

Directory:
┌────────┬───────┬───────┐
│ File   │ Start │ End   │
├────────┼───────┼───────┤
│ A      │ 0     │ 7     │
└────────┴───────┴───────┘
```

**Pros:** No external fragmentation, no size limit.
**Cons:** Slow random access (must follow pointers), pointer overhead.

#### 3. File Allocation Table (FAT)

Like linked allocation, but all pointers stored in a table at the beginning of disk.

```
FAT Table:
┌──────┬───────┐
│ Block│ Next  │
├──────┼───────┤
│  0   │  4    │
│  1   │ -1    │
│  2   │  7    │
│  3   │ -1    │
│  4   │  2    │
│ ...  │ ...   │
│  7   │ -1    │
└──────┴───────┘

File A: start=0, follow chain: 0→4→2→7
```

**Used by:** Windows (FAT12, FAT16, FAT32), USB drives.

#### 4. Indexed Allocation

Each file has an index block containing pointers to all its blocks.

```
Index Block for File A:
┌───────┐
│ Block │
├───────┤
│  4    │  ← First data block
│  7    │  ← Second data block
│  2    │  ← Third data block
│  10   │  ← Fourth data block
└───────┘

Data blocks: 4, 7, 2, 10
```

**Variants:**
| Method | Description |
|--------|-------------|
| **Linked Index** | Index blocks linked together |
| **Multilevel Index** | Index block points to other index blocks |
| **Combined** | Direct + indirect pointers (Unix inode) |

---

### Unix inode (Index Node)

The core data structure in Unix/Linux file systems.

```
┌─────────────────────────────────────┐
│ inode                                │
├─────────────────────────────────────┤
│ File type (regular, directory, etc.) │
│ Permissions (rwxr-xr-x)             │
│ Owner (UID)                          │
│ Group (GID)                          │
│ Size                                  │
│ Timestamps (create, modify, access)  │
│ Link count                            │
│ Direct pointers (12 blocks)          │
│ Single indirect pointer              │
│ Double indirect pointer              │
│ Triple indirect pointer              │
└─────────────────────────────────────┘

Direct blocks: 12 blocks (48KB with 4KB blocks)
Single indirect: 1 block of pointers (1024 pointers = 4MB)
Double indirect: 1024 × 1024 = 4GB
Triple indirect: 1024 × 1024 × 1024 = 4TB
```

---

### Free Space Management

How does the OS track which blocks are free?

| Method | Description | Pros | Cons |
|--------|-------------|------|------|
| **Bit Vector** | 1 bit per block (0=free, 1=used) | Simple, fast to find free block | Large for big disks |
| **Linked List** | Linked list of free blocks | No extra space | Slow to traverse |
| **Grouping** | First free block stores addresses of next N free blocks | Fast | Complexity |
| **Counting** | Store (start, count) of consecutive free blocks | Good for contiguous | Variable size |

---

### Journaling File Systems

Record changes before applying them to prevent corruption.

```
Write operation:
1. Write intent to journal (log)
2. Write data to actual location
3. Mark journal entry as complete

If crash occurs:
- Incomplete journal entries → rollback
- Complete journal entries → replay
```

**Used by:** ext3/ext4 (Linux), NTFS (Windows), HFS+ (Mac).

---

### Common File Systems

| File System | OS | Max File Size | Max Volume | Features |
|-------------|-----|---------------|------------|----------|
| ext4 | Linux | 16 TB | 1 EB | Journaling, extents |
| NTFS | Windows | 16 EB | 256 TB | Journaling, ACLs, compression |
| APFS | macOS | 8 EB | 8 EB | Cloning, encryption, snapshots |
| FAT32 | Cross | 4 GB | 8 TB | Simple, compatible |
| exFAT | Cross | 16 EB | 128 PB | Flash drives |

---

### Interview Questions

**Q: What's the difference between contiguous and linked allocation?**

A: "Contiguous allocation stores file blocks consecutively on disk—fast for sequential and random access but suffers from external fragmentation. Linked allocation stores blocks anywhere with pointers linking them—no fragmentation but slow random access. Indexed allocation (like Unix inodes) combines benefits: index block contains pointers to all data blocks."

**Q: What's an inode?**

A: "An inode is a data structure in Unix/Linux file systems that stores file metadata—permissions, owner, timestamps, size, and pointers to data blocks. It contains 12 direct pointers, plus single, double, and triple indirect pointers for large files. The inode doesn't store the filename—filenames are stored in directory entries that point to inodes."

**Q: What's a journaling file system?**

A: "A file system that records changes to a journal (log) before applying them. If the system crashes during a write, the journal can be replayed to recover. This prevents file system corruption and makes recovery fast (just replay the journal) instead of scanning the entire disk (like fsck)."

**Q: What's the difference between hard links and symbolic links?**

A: "A hard link is another directory entry pointing to the same inode (same file). Deleting one link doesn't affect the file until all links are removed. A symbolic (soft) link is a file containing the path to another file. It can span file systems and can point to non-existent files (dangling link)."

**Q: How does the OS find a file given its path?**

A: "Starting from the root directory, the OS resolves each path component: (1) Read root directory to find 'home' → get its inode, (2) Read home's data blocks to find 'alice' → get its inode, (3) Read alice's data blocks to find 'report.pdf' → get its inode, (4) Use inode to access file data. Directory entries map names to inodes."

---

*Next: [12 — I/O Systems](12-IO-Systems.md)*
