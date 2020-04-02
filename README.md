# How-UC-OS-II-switches-between-tasks-in-ARM-M4-arch-with-gcc
ARM-CM4 provides many facilities to make OS implementation OS implementation easier
and more efficient. For example: 
* Shadowed stack pointer: Two stack pointers are available. The MSP is used
for the OS Kernel and interrupt handlers. The PSP is used by application
tasks.
* SysTick timer: A simple timer included inside the processor. which is considered as 
the heartbeat of os by generating a regular interrupt to switch between tasks if needed.
* SVC and PendSV exceptions: These two exception types are essential for the
operations of embedded OSs such as the implementation of context
switching.
*                                                       ------------
### In this article we will focus on how uc-os switches between tasks however,uc/os always executes the highest priority task ready to run in  different cases .for examples :
* 1: When starting multitasking .
* 2:After returnning from ISR which causes set the ready state for task with higher priority than current running task.
* 3: After returnnung from systick timer interrupt if it is noticed that a higher priority task ready than the current running task.
*                                                   -----------
