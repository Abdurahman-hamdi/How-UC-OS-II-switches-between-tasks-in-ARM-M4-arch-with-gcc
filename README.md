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
 

![JVHK](https://user-images.githubusercontent.com/60859162/78462385-bd09cd00-76d1-11ea-87b8-7c790eacec40.PNG)

*Cpu automatically performs stacking by pushing R0-R3, R12, LR,PC and PSR which are called “caller saved registers.”and restore them at exception exit under the control of the processor’s hardware.In this way when returned to the interrupted program,all the registers would have the same value as when the interrupt entry sequence started. In addition, since the value of the return address (PC) is not stored in LR as in normal C function calls (the exceptionmechanism puts an EXC_RETURN code in LR at exception entry, which is used in
exception return), the value of the return address also needs to be saved by the exception sequence. So in total eight registers need to be saved during the exception handling sequence on the  Cortex-M4 processors.

### Now it is time to start context switching 😁😁!.

*UC/OS uses set of variables to keep track the operation of multitasking kernal like ospriocur,ospriohighrdy,ostcbcur and ostcbhighrdy.
*ospriocur:the priority of current running task,as shown in figure 3.13.

*ospriohighrdy:the priority of task has just becomed ready and more than the priority of current running task.

*ostcbcur:is a pointer of tcb type which points to the current running task tcb.

*ostcbhighrdy:is a pointer of tcb type which points to the higher priority task tcb rdy to run .

![ghhg](https://user-images.githubusercontent.com/60859162/78462211-402a2380-76d0-11ea-9800-0fac9f2f6158.PNG)


*suppose cpu is process TASK_A.the systic timer generates an interrupt and notices that thier is another task (TASK_B), with high priority than TASK_A rdy to run,the pseudocode for timer isr is :if(new_high_prio_tsk()==true)
                                                   {set pending state of pendsv}.
                                                   
                                                                                                      
*The figure below shows the system state while TASK_A running before TASK_B becomes ready.


![78463240-d5321a00-76da-11ea-87d6-913333809096](https://user-images.githubusercontent.com/60859162/78463891-e2eb9d80-76e2-11ea-8faf-fd15b00bf8a0.jpg)

*Once new_high_prio_tsk() has detected TASK_B is ready to run ,the timer isr sets the pending state for pendsv exception ,after timer isr finishes and thier are no other pended exceptions, the sytem enters pendsv isr to do context switching ,the system during pendsv handler enterance(autmatic stacking/push R0-R3,LR,PC,PSR using psp into task_A stack  ) is described as .


![4](https://user-images.githubusercontent.com/60859162/78464025-b769b280-76e4-11ea-9ebd-935da1d7f7c2.jpg)


* Pendsv exception isr starts by:
* 1:disable interrupts and save the remained TASK_A context by saving R4-R11 on its stack .
* 2:store current psp in TASK_A TCB STK_PTR(top of the task_A stack).
* as shown in the following figures.


![pens](https://user-images.githubusercontent.com/60859162/78473751-44435900-7743-11ea-97bd-31d511dae2e4.PNG)


![4](https://user-images.githubusercontent.com/60859162/78473844-01ce4c00-7744-11ea-9572-33dd773ee064.jpg)


* The second part of this isr stats by:
* 1:calling(if needed)OSTaskSwHook which is  user defined routine to to perform additional operations when UC/OS-II performs a context switch
* 2:assign the proirity of TASK_B to current priority ,OSPrioCur = OSPrioHighRdy. 
* 3:the task_B TCB address which OSTCBHighRdy points to it ,is assigned to the current task TCB OSTCBCur  = OSTCBHighRdy.
* 4: RO now holds the the address of the first member of task_b tcb which contains the task_b stack top pointer.
* 5: assign the task_b stack top pointer address to  R2 and assign the content of[R2] to R0  .


![pen2](https://user-images.githubusercontent.com/60859162/78498788-a94f7c80-774c-11ea-81b4-ce5803dccb68.PNG)


![4](https://user-images.githubusercontent.com/60859162/78499132-34317680-774f-11ea-9e78-a5c35a8d7b7c.jpg)


* 6: Load TASK_B R4-R11 from its stack using R0 for addressing the stack  .
* 7: assign RO to psp .


![pen2](https://user-images.githubusercontent.com/60859162/78499322-48c23e80-7750-11ea-89ec-0c448d778885.PNG)


![4](https://user-images.githubusercontent.com/60859162/78499520-7065d680-7751-11ea-855a-1547b97128c4.jpg)


* 8: Now context switching is about to finish ,but an important thing before exit from context swiching is to ensure that exception return uses process stack by OR bit 2 of EXC_RETURN which is hold by Link Register with "1".
* 9: enable interrupts and return from pendsv handler(automatic restoring task_b context( R0-R3,R12,LR,PC and PSR) when  task_b get preempted by task_A and put in wait state) to resume the execution of TASK_B.


![pen2](https://user-images.githubusercontent.com/60859162/78500362-31864f80-7756-11ea-9f3b-7e49f3b44526.PNG)


![1](https://user-images.githubusercontent.com/60859162/78500374-41059880-7756-11ea-8d30-778845370044.jpg)



*                                                     Thanks!















