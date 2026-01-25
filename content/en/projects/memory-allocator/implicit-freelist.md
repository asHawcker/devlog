---
date: "2026-01-25T09:45:19+05:30"
title: "Implicit Freelist Allocator"
weight: 2
toc: true
toc_position: "right"
---

{{< breadcrumbs >}}

---

## Why "Implicit"?

We don't store pointers to the next free block. The list is implied by the size in the header. To find a free block, we must visit every block (allocated or free) sequentially until we find one that fits.

## The Architecture:

- Block Format: Header + Payload + Footer (Boundary Tags).
- Traversal: Linear Scan (O(N)).
- Freeing: Very fast (just flip a bit).
- Coalescing: We will implement immediate coalescing using boundary tags.

Key Characteristic: The search path includes allocated blocks. This makes malloc slow (_O(N)_) when the heap is full of allocated chunks.

```
HEAP START                                                                 HEAP END
   |                                                                          |
   v                                                                          v
+------+    +----------------+    +------+    +----------------+    +------+
| PROL | -> |   Allocated    | -> | FREE | -> |   Allocated    | -> | EPIL |
+------+    |  Size: 32      |    | S:48 |    |  Size: 16      |    +------+
            +----------------+    +------+    +----------------+
            | Header: 32/1   |    | H:48/0|   | Header: 16/1   |
            | [ User Data  ] |    | [ ?  ]|   | [ User Data ]  |
            | [ User Data  ] |    | [ ?  ]|   |                |
            | Footer: 32/1   |    | F:48/0|   | Footer: 16/1   |
            +----------------+    +------+    +----------------+
                    ^                 |               ^
                    |                 |               |
              (Must Visit)       (Is Free)      (Must Visit)
```

### Pros & Cons

- **Simple:** Easy to implement and debug.
- **Low Overhead:** Only 8 bytes overhead per block.

- **Slow Allocation:** $O(N)$ where $N$ is total blocks. As heap fills, malloc becomes incredibly slow.
- **No Splitting (Naive):** The naive implementation wastes memory by returning the entire free block even if the user asked for a tiny slice.

## The changes I had to adopt

- I am not using normal variables (like struct Block) because we cannot afford the memory overhead.
- I could use inline functions in C and the compiler would optimise them and it would work well too but here are the more reasons for using macros.
  1. Polymorphism: I don't need to worry about input and output types with macros. This would have been an annoying and repititive task with functions.
  2. these macros act as my custom assembly like commands.

## Learnings

- I learned that we can't just slice memory anywhere. You must align to 8 or 16 bytes to satisfy hardware requirements.
- Knuth's Boundary Tags Trick: I learned how replicating the header at the end of the block (the Footer) allows for O(1) bidirectional traversal. This turned coalescing from a linear search problem into a constant-time operation.
- Now I know the problem is with traversing every block. free or not. So now I can work on that and that is exactly what the explicit freelist allocator is.
