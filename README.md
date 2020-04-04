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
*                                                 ------------
### In this article we will focus on how uc-os switches between tasks however,uc/os always executes the highest priority task ready to run in  different cases .for examples :
* 1: When starting multitasking .
* 2:After returnning from ISR which causes set the ready state for task with higher priority than current running task.
* 3: After returnnung from systick timer interrupt if it is noticed that a higher priority task ready than the current running task.
*                                                   -----------
* ##### Before going into how uc/os performs switching between tasks on CM4 ARCH,lets talk about the features provided by CM4 arch which uc/os uses .

*Systick timer :a 24 bit count down timer optionally used uc/os by kernal to provide a regular interupt to serve tasks dly and timing requests.

*Pendsv exception:this type of exception is set with the lowest priority in the system to handle switching between tasks,and triggered by uc/os services when actual context switching is needed.being this exception is set with the lowest priority ,it improves the system respone to hardware interrupts by delaying context switching until all other IRQ handlers have completed their processing.

*Two stack pointers: these pointers provide the usage two separate stacks, one(psp,process stack pointer) used by uc/os tasks and the other (Msp,main stack pointer) used by IRQ handlers,when uc/os starts multitasking it sets psp to be used in threed execution(tasks stack). however,the selection between these pointers is done by modification bit 1 of the CONTROL register (1=psp,0=msp) or modification bit 2 of EXC_RETURN value which is saved in link register register during execption returning as shown in figure 8:10 ,for more information go to "The Definitive Guide toARM Cortex-M3 and Cortex-M4 Processors Third Edition"chapter8


![JVHK](https://user-images.githubusercontent.com/60859162/78461444-3cdf6980-76c9-11ea-96a9-79dd91360137.PNG)

*Cpu automatically performs stacking by pushing R0-R3, R12, LR, and PSR which are called “caller saved registers.”and restore them at exception exit under the control of the processor’s hardware.In this way when returned to the interrupted program,all the registers would have the same value as when the interrupt entry sequence started. In addition, since the value of the return address (PC) is not stored in LR as in normal C function calls (the exceptionmechanism puts an EXC_RETURN code in LR at exception entry, which is used in
exception return), the value of the return address also needs to be saved by the exception sequence. So in total eight registers need to be saved during the exception handling sequence on the  Cortex-M4 processors.

### Now it is time to start context switching 😁😁!
*UC/OS uses set of variables to keep track the operation of multitasking kernal like ospriocur,ospriohighrdy,ostcbcur and ostcbhighrdy.
*ospriocur:the priority of current running task.
*ospriohighrdy:the priority of task has just becomed ready and more than the priority of current running task.
*ostcbcur:is a pointer of tcb type which points to the current running task tcb.
*ostcbhighrdy:is a pointer of tcb type which points to the high priority task tcb rdy to run .

![ghhg](https://user-images.githubusercontent.com/60859162/78461986-2687dc80-76ce-11ea-9c16-c416e6ff952f.PNG)








