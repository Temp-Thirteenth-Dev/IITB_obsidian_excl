## 1. The Trapframe (`TRAPFRAME` Page)

- Where is it? It is mapped into the process's virtual address space at the very top (just below the trampoline).
- Can the user access it? No. When the kernel creates this page mapping in the process's page table, it intentionally leaves the User-accessible bit (`PTE_U`) turned off.
- Why? The CPU can only read or write to this page when it transitions into Supervisor Mode (Kernel Mode). If a user process could access its own trapframe, it could alter its saved registers, fake system call arguments, or modify its own instruction pointer to hijack the OS.

## 2. The Kernel Stack (`kstack`)

- Where is it? It does not exist in the user's page table at all. It lives entirely in the Kernel's private memory space.
- Can the user access it? No. It is completely invisible to the user process. The user process doesn't even have a virtual address that points to it.
- Why? The kernel stack handles sensitive operating system function calls, local variables for drivers, and kernel states. Letting the user touch this would immediately break system security and stability.

## 3. The Context Struct (`struct context`)

- Where is it? It lives inside the kernel's process array (`struct proc`) in the Kernel's private data region.
- Can the user access it? No. Just like the kernel stack, it is completely omitted from the user's page table.
- Why? This holds the registers for the kernel scheduler. The user space has no business knowing or modifying scheduler states.

---

## 🗺️ Visualizing the Separation (xv6 Process Memory Map)

Here is how an xv6 process sees its virtual memory. Notice the strict boundary lines enforced by the hardware:

```text
Virtual Address
MAX_VA ──> ┌──────────────────────────────────────┐
           │     TRAMPOLINE (Shared Code)         │ <── User can EXECUTE, not Read/Write
           ├──────────────────────────────────────┤
           │      TRAPFRAME (Unique Page)         │ <── KERNEL ONLY (No User Access)
           ├──────────────────────────────────────┤
           │               ...                    │
           │        (Unmapped Gap)                │
           ├──────────────────────────────────────┤
           │            User Stack                │ <── User Read/Write/Execute Allowed
           ├──────────────────────────────────────┤
           │            User Heap                 │ <── User Read/Write Allowed
           ├──────────────────────────────────────┤
           │         User Code & Data             │ <── User Read/Write/Execute Allowed
0x0000 ──> └──────────────────────────────────────┘
```

## 🔒 The "Magic" of the Trampoline

You might wonder: _If the trampoline page is execution code for entering the kernel, can the user access it?_

The user process can execute the code on the trampoline page, but it cannot read or write to it (no `PTE_R` or `PTE_W` bits for the user).

When a trap happens, the CPU executes the trampoline code. The very first thing this assembly code does is switch a hardware register to tell the CPU: _"We are now in Supervisor Mode."_ Only _after_ that line executes does the CPU unlock the permission to read and write to the Trapframe page directly below it.

To answer your question directly: No, the user process cannot access its trapframe, kernel stack, or context struct.

Even though parts of these structures are mapped into the process's overall address space, they live in protected physical RAM and are strictly guarded by the CPU hardware using Page Table Permissions. If a user program tries to read or write to them, the CPU will instantly trigger a Page Fault and kill the process.


---
---
Here is the complete, high-precision map of every critical register, structure field, and variable involved in mode switching and context switching.

---

## 1. Hardware Registers (Inside the CPU)

These are physical registers built into the RISC-V CPU core. They change dynamically based on which mode or process is currently active.

- `satp` (Supervisor Address Translation and Protection): Holds the physical memory address of the active page table. Changing this instantly switches the entire virtual memory view from Process A's memory to Process B's memory.
- `sscratch`: A dedicated scratch register used _only_ by the kernel. While a user process is running, the kernel hides a pointer to that process's Trapframe inside `sscratch`.
- `stvec` (Supervisor Trap Vector Base Address Register): Holds the address of the trap handler. In user mode, it points directly to the Trampoline page (`uservec`). When a trap occurs, the CPU automatically jumps to the address stored here.
- `sepc` (Supervisor Exception Program Counter): When a trap occurs, the CPU automatically saves the user's current Program Counter (PC) into this register so the system knows exactly where to resume user code later.
- `sstatus`: Contains control bits that dictate the CPU's current privilege state. It tracks whether the CPU came from User or Supervisor mode, and controls whether interrupts are globally enabled or disabled.
- `sp` (Stack Pointer): Points to the active memory stack. It points to the User Stack in user mode, switches to the Kernel Stack during a trap, and switches to the Scheduler Stack during a context switch.

---

## 2. The Trapframe Fields (Unique page per process)

The `struct trapframe` is used exclusively for Mode Switching (User $\leftrightarrow$ Kernel). It acts as a safety vault for user states.

- `tf->kernel_satp`: Stores the kernel's global page table address. The trampoline reads this to switch the CPU away from the user's page table.
- `tf->kernel_sp`: Stores the top address of this process's dedicated Kernel Stack. The trampoline loads this into the `sp` register so the C kernel has a stack to run on.
- `tf->kernel_trap`: Stores the memory address of the high-level C kernel trap handler function (`usertrap()`).
- `tf->epc`: Used during the return trip. The kernel copies the target user execution address here before copying it back into the hardware `sepc` register.
- `tf->regs[32]` (an array): An array that manually backs up all 32 general-purpose user registers (like `ra`, `a0-a7`, `t0-t6`) while the process is trapped in the kernel.

---

## 3. The Context Struct Fields (Embedded inside `struct proc`)

The `struct context` is used exclusively for Context Switching (Kernel A $\leftrightarrow$ Scheduler $\leftrightarrow$ Kernel B). It only saves callee-saved registers because the compiler handles the rest during a function call.

- `context->ra` (Return Address): Stores the code address where the thread should resume execution when it wakes back up (usually right after the line that called `swtch()`).
- `context->sp` (Stack Pointer): Saves the process's current kernel stack depth.
- `context->s0` through `context->s11`: Saves the 12 callee-saved registers specified by the RISC-V calling convention to preserve local kernel variables.

---

## 4. Kernel Variables (Global Memory)

These are global variables managed by the xv6 operating system C code to orchestrate which process gets the CPU.

- `proc[NPROC]` (The Process Table): A global array of all processes in the system. The scheduler loops through this table looking for processes marked as `RUNNABLE`.
- `myproc()` (Function returning a `struct proc*`): A helper function that looks up the currently executing process on the current CPU core. It reads this information out of a dedicated hardware register (`tp`, the thread pointer), which xv6 uses to store the current CPU ID.
- `cprocs` / `cpus` array: A global array tracking each physical CPU core status. Each CPU core structure contains its own `struct context scheduler` field, representing that specific core's dedicated background scheduler loop.

---

## 🗺️ Cheat Sheet: Who uses what?

|Operation|Critical Handshakes|
|---|---|
|Mode Switch (User $\to$ Kernel)|`stvec` $\to$ `sscratch` $\to$ `trapframe` $\to$ `satp` (Kernel) $\to$ `kernel_sp`|
|Context Switch (Process $\to$ Scheduler)|`swtch()` $\to$ `p->context` $\to$ `cpu->scheduler.context`|

Would you like to focus on the code implementation of one of these transitions? I can:

1. Show the exact RISC-V assembly instructions inside the trampoline (`uservec`) where `sscratch` and `trapframe` interact.
2. Provide the visual layout of how `struct proc` groups both the `context` and `kstack` variables together in memory.

Please choose 1 or 2 to proceed!