# Tracing sysalls: getpid()
The internal implementation of system calls is provided by a kernel function of the form: “sys_<name>”, here it will be sys_getpid

- Declaration: https://elixir.bootlin.com/linux/v5.15.196/source/include/linux/syscalls.h#L783

- Definition: System Calls inside the kernel are not defined directly, instead they are defined via the SYSCALL_DEFINEx(name, args….) macro, where x is the number of argos to the system call. Since, getpid() accepts no args, hence it must be defined using SYSCALL_DEFINE0, which it is here: https://elixir.bootlin.com/linux/v5.15.196/source/kernel/sys.c#L943

- **System Call Number**: A syscall number identifies the system call, the following file corresponds to generic arch, if no arch-specific overrides are available. Here a system call number of 172 is assigned for getpid() and mapped to the implementation of sys_getpid(). Notice the system call numbers are declared using the __NR_<system_call_name> macros, here __NR_getpid. https://elixir.bootlin.com/linux/v5.15.196/source/include/uapi/asm-generic/unistd.h#L521
- The above file is target-generic, this file: https://elixir.bootlin.com/linux/v5.15.196/source/arch/x86/entry/syscalls/syscall_64.tbl is specific to x86, and as can be seen it provides its own syscall numbers, for example “read” system call has the number 0 here (as contrast to 63), and getpid has the number 39:

        39	common	getpid			sys_getpid

- In the definition in sys.c, it can be seen that this function actually returns the tgid, not the pid.
- As can be seen it makes use of the current macro, which tracks the currently running process on each CPU.
