# FreeRTOS – Detailed Notes

## 1. Accessing Source Code
- FreeRTOS is open-source and available online.
- Official documentation explains:
  - Kernel architecture
  - Porting to new hardware
  - Configuration parameters

---

## 2. Core Kernel Files
The FreeRTOS kernel is composed of several key files:

- `tasks.c` → Task management (creation, deletion, scheduling)
- `queue.c` → Queues, semaphores, mutexes
- `list.c` → Linked list implementation used internally
- `timers.c` → Software timers (if enabled)

Port-specific:
- `port.c` → Hardware-dependent context switching and interrupt handling

---

## 3. Portability Structure
FreeRTOS is highly portable due to its structured design.

### Directory Structure:
```
Source/
 ├── tasks.c
 ├── queue.c
 ├── list.c
 └── portable/
       ├── <compiler>/<architecture>/port.c
       └── MemMang/heap_x.c
```

### Memory Management:
- `heap_1.c` → No free (fast, deterministic)
- `heap_2.c` → Free but no coalescing
- `heap_3.c` → Uses standard malloc/free
- `heap_4.c` → Coalescing (recommended)
- `heap_5.c` → Multiple memory regions

---

## 4. FreeRTOS Plus
Additional libraries extend functionality:

- TCP/IP stack
- UDP support
- File systems
- IoT middleware

---

## 5. Integration into Project
To use FreeRTOS:
1. Add kernel source files
2. Add correct `port.c`
3. Include a heap implementation
4. Configure `FreeRTOSConfig.h`

---

## 6. ESP32 Architecture
- Dual-core (Xtensa architecture)
- Supports SMP (Symmetric Multiprocessing)
- Tasks can be pinned to specific cores
- Can run in single-core mode if required

---

## 7. Development Setup
Typical setup:
- Install Arduino IDE or ESP-IDF
- Install ESP32 board package
- Select correct board
- Configure serial port

---

## 8. Configuration Parameters (FreeRTOSConfig.h)
This file defines system behavior:

- `configMAX_PRIORITIES` → number of priority levels
- `configMINIMAL_STACK_SIZE` → base stack size
- `configTICK_RATE_HZ` → tick frequency
- `configTOTAL_HEAP_SIZE` → total heap size

---

## 9. Tick Timer
- A hardware timer generates periodic interrupts
- Each interrupt = 1 RTOS tick

Scheduler:
- Runs at each tick
- Updates delays and timeouts

### Delay API:
```
vTaskDelay(ticks);
```

---

## 10. Task Creation

### API:
```
xTaskCreate()
```

### Parameters:
- Task function
- Task name
- Stack size
- Priority
- Parameters
- Task handle

### ESP32-specific:
```
xTaskCreatePinnedToCore()
```

---

## 11. Non-Blocking Delay
```
vTaskDelay()
```

- Moves task to Blocked state
- Allows CPU to run other tasks
- Essential for multitasking

---

## 12. Scheduler Start
```
vTaskStartScheduler()
```

- Starts RTOS kernel
- Control passes from main() to scheduler
- Tasks begin execution based on priority

---

## 13. Key Concepts

### Preemptive Scheduling
- Higher priority tasks interrupt lower ones

### Determinism
- Predictable timing behavior

### Context Switching
- Saves/restores:
  - Registers
  - Stack pointer
  - Program counter

---

## 14. Best Practices

- Keep tasks lightweight
- Avoid blocking high-priority tasks unnecessarily
- Use queues/semaphores for communication
- Monitor stack usage

---

## 15. Summary

FreeRTOS provides:
- Task scheduling
- Memory management
- Inter-task communication
- Hardware abstraction

It is widely used in embedded systems due to:
- Efficiency
- Portability
- Real-time guarantees
