# Free_RTOS_0.md
# Introduction to RTOS – Detailed Notes

---

## 1. Interrupt Service Routine (ISR)

An Interrupt Service Routine (ISR) is a function executed in response to an interrupt signal.

### Key Features:
- Executes immediately when interrupt occurs
- Preempts currently running task
- Used for time-critical operations

### Constraints:
- Must be very short (< 1 ms typical)
- Should NOT:
  - Perform long computations
  - Call blocking APIs (e.g., vTaskDelay)
  - Use normal RTOS APIs

### ISR-safe APIs:
- xQueueSendFromISR()
- xSemaphoreGiveFromISR()

### Design Principle:
Heavy processing should be deferred to tasks.

---

## 2. Superloop Architecture

Traditional embedded systems use a superloop:

while(1) {
    task1();
    task2();
    task3();
}

### Limitations:
- No prioritization
- Blocking delays affect entire system
- Poor scalability
- Difficult to maintain timing guarantees

---

## 3. RTOS on Single-Core MCUs

RTOS enables multitasking even on a single-core processor.

### Mechanisms:
- Time slicing
- Context switching
- Preemptive scheduling

### Concept:
Multiple tasks appear to run simultaneously through rapid switching.

---

## 4. Task vs Thread

In RTOS:

- Task = Unit of execution
- Thread = Execution context

### In FreeRTOS:
Task ≈ Thread

Each task contains:
- Program Counter (PC)
- Stack
- CPU registers
- Priority

---

## 5. Memory Requirements

RTOS requires additional memory compared to bare-metal systems.

### Components:

#### Kernel Memory:
- Scheduler structures
- Task Control Blocks (TCB)

#### Stack Memory:
- Each task has its own stack
- Stores function calls and local variables

#### Heap Memory:
Used for:
- Task creation
- Queues
- Semaphores

### Risks:
- Stack overflow
- Heap fragmentation

---

## 6. Concurrent Execution

RTOS allows multiple tasks to run concurrently.

### Example:
- Sensor Task
- Communication Task
- Control Task
- Logging Task

### Note:
- Not true parallelism on single-core
- Achieved via context switching

---

## 7. FreeRTOS Overview

FreeRTOS is a lightweight, open-source RTOS.

### Features:
- Deterministic scheduling
- Small footprint
- Portable
- Widely used

### Maintained by:
Amazon (AWS FreeRTOS)

### Supported Platforms:
- ESP32
- ARM Cortex-M
- RISC-V

---

## 8. Why RTOS is Needed

### Without RTOS:
- Blocking delays
- Poor timing control
- Difficult scaling

### With RTOS:
- Priority-based execution
- Predictable timing
- Efficient CPU usage
- Modular design

---

## 9. Core RTOS Concepts

### Scheduling:
- Priority-based
- Preemptive

### Context Switching:
- Saves/restores:
  - Registers
  - Stack pointer
  - Program counter

### Blocking:
- Tasks wait for:
  - Time (vTaskDelay)
  - Events (queues, semaphores)

### Synchronization:
- Prevents race conditions
- Uses mutexes and semaphores

---

## 10. Best Practices

- Keep ISRs short
- Use queues/semaphores for communication
- Avoid blocking high-priority tasks
- Properly size stacks
- Monitor heap usage

---

## 11. Summary

RTOS provides:
- Task management
- Scheduling
- Memory management
- Inter-task communication

### Final Insight:
RTOS = Scheduler + Tasks + Interrupt Handling + Synchronization
