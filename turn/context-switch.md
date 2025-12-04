## Saving and Loading Registers
The syscall instruction traps into the kernel to the system call handler. As part of this switch, the user registers are saved onto the kernel stack, or more precisely the user registers are saved onto the kernel stack for the process (since kernel stacks are organized per-process). These registers include: RIP (instruction pointer), SS (stack pointer) etc.

Note:
- On x86_64, ring 0 represents kernel mode, while ring 3 is user mode.
- Saving and restoring registers is important, as it makes context switching reversible, i.e. a process can be switched out in favour of another and be rescheduled later. Here in the case of syscall instruction, saving registers like RIP ensures that the CPU correctly returns to the next instruction after the syscall instruction has executed and continues executing from there on.
- The syscall instruction on x86_64 saves the RIP (instruction pointer) into RCX, RFLAGS into R11 and then switches to kernel mode (ring 0).
- The kernel then pushes additional registers (state) onto the kernel stack for the process. This is needed as during the course of its execution a process updates and uses many registers. The RCX register (saved RIP) is also pushed onto the kernel stack. When the processor traps back to the user mode, the kernel will restore the registers. From the process perspective the environment will be exactly the same as when it was switched out.

## Notes on Kernel Stack:
As mentioned before kernel stack lives in the kernel space and is used by a process when it executes in kernel mode (for example during a system call or interrupt).
- Kernel stack is per-thread, not global.
- The kernel cannot simply use the user stack for running privileged instructions, as the user stack could be corrupted or swapped out (demand paging). Hence the kernel uses its own stack stored in protected memory, which cannot be swapped out.
- Like a user stack, the kernel stack too is used to keep track of the call chain (in kernel code), and like a regular user stack it stores local / automatic variables, function parameters and return addresses.
- In addition to the above common functionalities, Registers are saved to the kernel stack during syscall entry.

The last point is important, we can save all the registers part of the process's execution context to the kernel stack, including registers like RCX, RIP etc. Since each thread or task has its own kernel stack, hence when the scheduler decides to perform a context switch.
- It can save the CPU registers of the current process to its kernel stack and saves the kernel stack pointer itself in the process descriptor.
- Next, it switches the stack pointer to use the kernel stack of the next task, and subsequently restores the saved registers from the kernel stack.

As mentioned before the kernel stack pointer is saved in task_struct itself (https://elixir.bootlin.com/linux/v6.18-rc4/source/arch/arc/include/asm/thread_info.h#L40). 

- When a context switch happens, the kernel (scheduler) saves the current CPU registers onto the kernel stack and saves the current kernel stack pointer to the process descriptor.
- Then it updates the stack pointer register (CPU’s SP register), so as to use the kernel stack of the next task.

## Process Context vs Interrupts Context
Some differences b/w process and interrupt context have already been noted earlier. One of the major differences is that code in the process context can sleep, while the handler in interrupt context cannot sleep, it must run quickly.

As can be seen fabove, the system call handler runs in the process context.
- It uses the kernel stack
- The handler can sleep, by handler we mean the execution context (i.e. the process) running the handler code. Such a process can sleep in waitqueue, waiting for some condition or event.
- It can be preempted.

Thus system calls can be seen as synchronous interrupts. Page fault handlers similar to system calls run in the process context. Page faults happen when a process tries to access a page, which is currently not resident in memory (demand paging), if this happens the hardware raises a page fault exception and traps into the Kernel handler. The page fault exception is a  synchronous exception. Hence the handler runs in the process context. As part of bringing the page into memory the handler needs to bring in pages from the disk, i.e. disk I/O and other complex operations which require blocking, and as discussed above blocking or sleeping is only allowed in process context.

So in general, if the handler is performing any blocking operations like I/O then it is running in the process context not the interrupt context.

A timer interrupt handler on the other hand runs in the interrupt context.
- Handlers running in this context have their own stack.
- The handler running in interrupt context is not associated with any specific process.

Interrupt context does not allow any sleeping (cannot block or schedule). The handler must finish quickly because interrupts are high-priority.

- Process context means the kernel is executing code on behalf of a process, example: syscall, page fault exception.
- Interrupt context is independent of any process, it's triggered by hardware.

In the above example we saw that page fault is a synchronous exception, hence handled in the process context, an exception is a type of synchronous interrupt.
- Synchronous Interrupt
  - Directly caused by the current process’s own actions, for example syscall, page fault (accessing an unmapped page), illegal actions like: addressing invalid memory, or memory out of bounds for this process, division by zero etc.
  - Also known as exceptions or traps.
  - The kernel handles these interrupts on behalf of the process, i.e. in process context, hence the kernel stack for the process is used.
- Asynchronous Interrupts
  - These interrupts are generated by external hardware devices, like disk, timer etc.
  - They occur independently of any process.
  - The kernel handles these interrupts in the special interrupt context.
  - Examples: Disk IO completion, network packet arriving, keyboard input, mouse movement.

In general synchronous interrupts are software based and are handled in the context of that process. Whereas asynchronous interrupts are hardware based, unpredictable and are handled in the interrupt context. In the process context, complex operations like memory allocation and Disk I/O can be performed because the process context allows blocking.

syscall and sysret: These are special instructions on x86 used for trapping into the kernel mode to run the syscall handler and for returning to user mode respectively.
When syscall instructions executes:
- Hardware saves RIP into RCX register.
- Hardware saves RFLAGS into R11
- Immediately after entry into the syscall handler:
- The kernel pushes the RCX (saved RIP), RSP (stack pointer) and other registers onto the kernel stack.
- These registers are not pushed directly to the kernel stack, instead they are copied to a pt_regs structure, which in turn is stored on the stack.
- The pt_regs structure stores all the different registers.

When sysret executes:
- Kernel restores the registers from pt_regs (kernel stack of the process).
- The processor switches back to user mode (i.e. ring 0 to ring 3).

## Flow:
- Save user-space registers like RIP, RSP, base and bounds in a pt_reg struct, store it on top of the per-task kernel stack.
- Store a pointer to the process’s kernel stack in task_struct itself.
- Then switch (context) to another process.When scheduler picks a process to run, it sets CPUs kernel stack pointer register to this task's kernel stack.
- Pops the pt_reg struct entry from the kernel stack, gets the register values and restores them (including RIP)
- Trap back to user-mode to that RIP.

Note: Kernel code, just like user code, is preemptable. An interrupt, like the timer interrupt can preempt kernel code.

Important Register:
- RIP: Instruction Pointer, address of the next instruction to execute.
- RSP: Stack Pointer, points to the top of the current stack (user stack in user mode, kernel stack in kernel mode).
- RAX: Holds syscall number (on entry) and return code (on exit).
