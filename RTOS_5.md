# Free_RTOS_5.md  
# Introduction to RTOS – Part 5: Queues (Detailed Notes)

## 1. Introduction to Queues
A queue in FreeRTOS is used for inter-task communication and synchronization.
Producer Task → Queue → Consumer Task

## 2. Internal Working
- Queue Control Block
- Circular Buffer
- Task wait lists (send/receive)

## 3. Memory Allocation
Queues are allocated in heap using pvPortMalloc().

## 4. Queue Creation
QueueHandle_t xQueueCreate(UBaseType_t length, UBaseType_t itemSize);

Example:
QueueHandle_t queue;
queue = xQueueCreate(10, sizeof(int));

## 5. Sending Data
xQueueSend(queue, &value, portMAX_DELAY);

## 6. Receiving Data
xQueueReceive(queue, &received, portMAX_DELAY);

## 7. Blocking Mechanism
Tasks block when queue is full/empty instead of polling.

## 8. Tick-Based Timeout
Timeouts are defined in RTOS ticks.

## 9. ISR Usage
Use xQueueSendFromISR() instead of xQueueSend().

## 10. Data Handling
Queues copy data → better to send pointers for large data.

## 11. Scheduling
Highest priority waiting task is unblocked first.

## 12. Producer-Consumer Example
Producer:
while(1){
    xQueueSend(queue, &value, portMAX_DELAY);
}

Consumer:
while(1){
    xQueueReceive(queue, &received, portMAX_DELAY);
}

## 13. Summary
Queue = Communication + Synchronization + Scheduling
