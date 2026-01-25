---
date: "2026-01-25T09:45:27+05:30"
title: "Explicit Freelist Allocator"
weight: 3
toc: true
toc_position: "right"
---

{{< breadcrumbs >}}

---

Please read the [Implicit Freelist Allocator](../implicit-freelist) first.

## Why Upgrade to Explicit?

The Implicit List Bottleneck: In the implicite freelist allocator, find*fit loops through every single block in the heap (allocated or free) to find a free one. Time Complexity: \_O(N)* where _N_ is the total number of heap blocks. So if we have 10GB of data allocated in tiny chunks, malloc will have to scan millions of headers just to find one free slot. This is unacceptably slow.

The Explicit List creates a Doubly Linked List containing only the free blocks.Time Complexity: _O(F)_ where _F_ is the number of free blocks. If memory is full, _F_ is small, so malloc is instant.

## Architecture

- **Payload Overlay:** Free blocks store `next` and `prev` pointers _inside_ the empty payload area.
  - `[ Header | PREV_PTR | NEXT_PTR | ... | Footer ]`
- **List Policy:** **LIFO** - Newly freed blocks are inserted at the root of the list.
- **Search Algorithm:** First Fit on the _Free List_. We only scan free blocks.
- **realloc added**: Added `my_realloc` function which smartly extends or shrinks instead of doing malloc-copy-free
- We need to store `next` and `prev` pointers for our linked list. But we don't want to waste extra RAM for them.
- A free block has no user data. The "Payload" area is empty. The Trick: We store the next and prev pointers inside the empty payload of the free blocks.

Free blocks form a distinct Doubly Linked List. Allocated blocks are ignored during the search. The pointers (`next`, `prev`) are stored inside the empty payload of the free blocks.

```
HEAP START           ROOT (free_list_p)                                    HEAP END
   |                  |                                                       |
   v                  v                                                       v
+------+    +----------------+    +-------------+    +----------------+    +------+
| PROL |    |   Allocated    |    |    FREE     |    |   Allocated    |    | EPIL |
+------+    |                |    |             |    |                |    +------+
            +----------------+    +-------------+    +----------------+
            | Header: 32/1   |    | Header:48/0 |    | Header: 16/1   |
            | [ User Data ]  |    | Next Ptr -----------------------------> NULL
NULL <----------------------------- Prev Ptr    |    | [ User Data ]  |
            |                |    | ...         |    |                |
            | Footer: 32/1   |    | Footer:48/0 |    | Footer: 16/1   |
            +----------------+    +-------------+    +----------------+
                    ^                    ^                   ^
                    |                    |                   |
               (IGNORED)            (VISITED)            (IGNORED)
```

## Pros & Cons

- **Fast Allocation:** _O(F)_ where _F_ is number of free blocks. Much faster than implicit.
- **Splitting:** Implemented block splitting to reduce internal fragmentation.

- **Complexity:** Managing list pointers during splitting and coalescing is error-prone.
- **Min Block Size:** Blocks must be at least 32 bytes (16 overhead + 16 pointers) to be freeable.

## Benchmarking & Performance

The project includes a performance benchmark `benchmark.c` designed to demonstrate the critical difference between the Implicit and Explicit allocator architectures.

### The Workload

The benchmark simulates a high-load system (like a game engine or web server) by performing **50,000 random operations**:

- **Traffic Pattern:** 60% Allocation / 40% Free (Simulates a heap that grows over time).
- **Stress Test:** Random block sizes (1 byte to 1024 bytes) to intentionally cause external fragmentation and "holes" in the heap.
- **Metric:** Measures **Operations Per Second (OPS)** and Total Execution Time.

### Performance Characteristics

- **Implicit List:** Performance degrades quadratically (O(N)) as the heap fills. As the number of allocated blocks increases, `malloc` must scan past all of them to find free space.
- **Explicit List:** Maintains high, constant throughput (O(Free)). Performance remains stable even as the heap grows to 50MB+, as `malloc` only traverses the dedicated list of free blocks.

---

## Next steps

The next obvious step would be a Segregated Freelist allocator, but it operates on the same principles. The core difference is to seperate the freelists into buckets based on size which would lead to a chained hashmap kind of structure. This is more of a debugging job, and I not much to learn in implementation as it is a minor change, still very important ofc.

I will proceed with the Buddy and Slab Allocators.
