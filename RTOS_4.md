# Memory Management in FreeRTOS

## 1. Overview
Understanding where data resides in memory is crucial to prevent memory leaks and ensure efficient system operation.

---

## 2. Types of Memory Allocation

### 2.1 Static Allocation
- Global and static variables are allocated at compile time.
- Stored in the **static memory region**.
- Lifetime: entire program execution.

---

### 2.2 Stack (Automatic Allocation)
- Local variables are stored in the **stack**.
- Allocation happens during function calls.
- Operates in **LIFO (Last In First Out)** manner.
- Stack size can grow, but is limited.
- Also called **automatic allocation**.

---

### 2.3 Heap (Dynamic Allocation)
- Used for **dynamic memory allocation**.
- Memory is allocated at **runtime**.
- Requires manual management (allocation + deallocation).

---

## 3. Dynamic Memory Handling

- Heap memory is used when variable size is not known at compile time.
- Must explicitly free allocated memory to avoid leaks.

```c
void *ptr = pvPortMalloc(size);
vPortFree(ptr);
```

---

## 4. FreeRTOS Task Memory Structure

Each task consists of:

- **Task Control Block (TCB)**
- **Task Stack**

### Key Points:
- Both TCB and stack are allocated from **heap memory**.
- Proper stack size must be assigned to avoid:
  - Stack overflow
  - Memory corruption

---

## 5. Heap Usage in FreeRTOS

Heap is used for:
- Task creation
- Queues
- Semaphores
- Timers

---

## 6. Memory Fragmentation

- Frequent allocation and deallocation can cause **heap fragmentation**.
- Fragmentation leads to:
  - Inefficient memory usage
  - Allocation failures despite available memory

---

## 7. Heap Management Schemes in FreeRTOS

FreeRTOS provides multiple heap implementations:

| Scheme | Description |
|------|------------|
| heap_1 | No free (simple, fast) |
| heap_2 | Allows free but no coalescing |
| heap_3 | Uses standard `malloc/free` |
| heap_4 | Coalescing to reduce fragmentation |
| heap_5 | Supports **non-contiguous memory regions** |

### Recommended:
- **heap_5** → Most advanced  
- Allows combining multiple memory regions into one logical heap

---

## 8. Memory Allocation API (FreeRTOS)

Use FreeRTOS-specific APIs instead of standard `malloc/free`:

```c
void *pvPortMalloc(size_t size);
void vPortFree(void *ptr);
```

---

## 9. Important Design Considerations

- Always check if allocation was successful:
```c
if(ptr == NULL) {
    // Handle error
}
```

- Ensure sufficient **contiguous heap memory**
- Avoid excessive fragmentation
- Use appropriate heap scheme

---

## 10. Error Handling

If sufficient contiguous memory is not available:
- Allocation will fail
- Proper error handling must be implemented

---

## 11. Best Practices

- Minimize dynamic allocation in real-time systems
- Prefer static allocation where possible
- Carefully size task stacks
- Monitor heap usage during runtime

---

## 12. Summary

- Static → Compile-time allocation  
- Stack → Function-level automatic allocation  
- Heap → Runtime dynamic allocation  

Efficient memory management is critical in FreeRTOS to ensure:
- System stability  
- Predictable timing  
- No memory leaks  
