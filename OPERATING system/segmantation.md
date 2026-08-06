# 10 — Segmentation

## Memory Organization by Logical Purpose

---

### What is Segmentation?

Segmentation divides memory into **variable-size segments** based on logical divisions: code, data, stack, heap. Each segment has a name and length.

**Unlike paging:** Segments are logical units (code, data, stack), not arbitrary fixed-size blocks.

```
Process View:
┌────────────────────────┐
│ Segment 0: Main        │ (code)
├────────────────────────┤
│ Segment 1: Function X  │ (code)
├────────────────────────┤
│ Segment 2: Global Data │ (data)
├────────────────────────┤
│ Segment 3: Stack       │ (stack)
└────────────────────────┘
```

---

### Segmented Address

A logical address is a **tuple: (segment_number, offset)**.

```
Logical Address: (2, 150)

Segment Table:
┌──────┬────────┬────────┐
│ Seg  │ Base   │ Length │
├──────┼────────┼────────┤
│  0   │ 4000   │ 1000   │
│  1   │ 6000   │ 500    │
│  2   │ 8000   │ 400    │  ← Segment 2
│  3   │ 10000  │ 2000   │
└──────┴────────┴────────┘

Check: offset 150 < length 400? Yes
Physical address = base + offset = 8000 + 150 = 8150
```

---

### Segmentation vs Paging

| Aspect | Paging | Segmentation |
|--------|--------|--------------|
| **Size** | Fixed | Variable |
| **Divided by** | Physical convenience | Logical purpose |
| **Fragmentation** | Internal only | External |
| **Address** | (page, offset) | (segment, offset) |
| **User visible** | No (transparent) | Yes (logical units) |
| **Sharing** | Difficult | Easy (by segment name) |

---

### Segmentation with Paging

Modern systems combine both:

```
Logical Address: (segment, page, offset)

Segment Table → Page Table → Physical Address

Segment points to page table
Page table maps to physical frames
```

**Used by:** Intel x86 (segments + pages), MULTICS.

---

### Interview Questions

**Q: What's segmentation?**

A: "Segmentation divides memory into variable-size segments based on logical purpose—code, data, stack, heap. Each segment has a base address and length. The logical address is (segment_number, offset). Segments make it easy to share code and enforce protection (read-only code, read-write data)."

**Q: What's the difference between paging and segmentation?**

A: "Paging divides memory into fixed-size physical blocks (frames) for convenience. Segmentation divides memory into variable-size logical units. Paging is transparent to the user; segmentation reflects the program's logical structure. Paging has internal fragmentation; segmentation has external fragmentation."

**Q: What's external fragmentation?**

A: "Free memory is scattered in small, non-contiguous blocks. Even if total free memory is enough, a large allocation might fail because no single contiguous block is large enough. Solutions: compaction (move processes to consolidate free memory) or paging (don't need contiguous physical memory)."

---

*Next: [11 — File Systems](11-File-Systems.md)*
