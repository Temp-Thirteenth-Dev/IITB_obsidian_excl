# Ch 0

Here is a detailed breakdown of **Operating System Interfaces** (traditionally **Chapter 0** in x86 versions of the xv6 text, and **Chapter 1** in the RISC-V revision).

---

### **1. Core Role of the Operating System & Interface Design**

- **Primary Objective**: The operating system shares a computer among multiple programs, provides hardware abstractions (so applications don't need to know low-level disk details), and enables controlled interaction between processes.
- **Kernel & User Space Isolation**:
    - **Kernel**: A special privileged program that manages hardware resources.
    - **Process**: A running program with its own memory containing instructions, data, and a stack.
    - **Hardware Privileges**: Hardware protection mechanisms (e.g., RISC-V supervisor vs. user mode) ensure user-space processes cannot directly access kernel memory or restricted instructions.
    - **System Calls**: When a user process requires a kernel service, it executes a system call (via instructions like `ecall`), which raises the hardware privilege level and jumps to a pre-arranged kernel entry point.

---

### **2. Summary of xv6 System Calls**

|System Call|Description|
|:--|:--|
|**`fork()`**|Creates a new process (child) as an exact copy of the parent. Returns child's PID to parent, `0` to child.|
|**`exit(int status)`**|Terminates the calling process and releases resources; reports status to `wait()`.|
|**`wait(int *status)`**|Blocks until a child process exits; returns child's PID and copies exit status into `*status`.|
|**`exec(char *file, char *argv[])`**|Loads an ELF binary file and overwrites the current process memory image to run it.|
|**`sbrk(int n)`**|Adjusts the process memory allocation by `n` bytes.|
|**`open(char *file, int flags)`**|Opens a file or device; returns a small integer file descriptor.|
|**`read(int fd, char *buf, int n)`**|Reads up to `n` bytes from `fd` into `buf`; returns count read (or `0` at EOF).|
|**`write(int fd, char *buf, int n)`**|Writes `n` bytes from `buf` to `fd`.|
|**`close(int fd)`**|Releases the specified open file descriptor.|
|**`dup(int fd)`**|Duplicates an existing file descriptor, returning a new descriptor pointing to the same file.|
|**`pipe(int p[])`**|Creates a pipe; populates `p` (read end) and `p` (write end).|
|**`chdir(char *dir)`**|Changes the process's current working directory.|
|**`mkdir(char *dir)`**|Creates a new directory.|
|**`mknod(char *file, int, int)`**|Creates a device file.|
|**`fstat(int fd, struct stat *st)`**|Populates `*st` with inode metadata for open file `fd`.|
|**`link(file1, file2)`**|Creates a new hard link (`file2`) pointing to existing `file1`.|
|**`unlink(char *file)`**|Removes a file link from the directory.|

---

### **3. Processes and Memory Management**

- **Process State**: Comprises user-space memory (text, data, stack, heap) and kernel-managed per-process state.
- **Process Creation with `fork()`**:
    - Creates an exact memory copy (instructions, variables, stack).
    - Parent receives the child’s PID; child receives `0`.
    - Both processes run with separate address spaces and independent registers.
- **Program Execution with `exec()`**:
    - Overwrites the caller’s user memory image with an ELF executable binary loaded from disk.
    - Re-initializes stack and registers, starting execution at the entry point declared in the ELF header.
- **The UNIX Shell Pattern**:
    - The shell reads a user command with `getcmd`, calls `fork()`, calls `wait()` in the parent, and calls `exec()` in the child to launch the program.
    - Separating process creation (`fork`) from execution (`exec`) gives the shell an operational window to adjust file descriptors and setup I/O redirection before the executable runs.
- **Dynamic Memory**: `sbrk(n)` expands or shrinks user heap memory dynamically.

---

### **4. I/O and File Descriptors**

- **File Descriptor Abstraction**:
    - A file descriptor is an integer index into a private per-process table that abstracts files, directories, pipes, and devices into plain byte streams.
    - **Standard Descriptors**: By convention, `0` is standard input (`stdin`), `1` is standard output (`stdout`), and `2` is standard error (`stderr`).
- **Allocation Rule**: `open()`, `dup()`, or `pipe()` always allocates the **lowest-numbered unused descriptor** in the process table.
- **I/O Redirection Mechanism**:
    1. The child process forks.
    2. The child closes descriptor `0` (or `1`) via `close(0)`.
    3. Opening a file (e.g., `open("input.txt", O_RDONLY)`) automatically reassigns descriptor `0` to the new file.
    4. The child calls `exec()`, preserving the updated descriptor table while loading the target binary.
- **Shared Offsets**: `fork()` and `dup()` duplicate file descriptor references such that parent and child processes share the exact same underlying file offset.

---

### **5. Pipes**

- **Concept**: A small kernel buffer exposed as a pair of file descriptors (`p` for reading, `p` for writing) used for Inter-Process Communication (IPC).
- **Blocking & EOF**: If no data is available, `read()` blocks until bytes are written or until all write descriptors referencing the pipe are closed (returning `0` for EOF).
- **Advantages over Temporary Files**:
    1. **Automatic Cleanup**: Disappears when descriptors close without leaving stale disk files.
    2. **Unlimited Stream Size**: Transfers arbitrary amounts of data without requiring disk storage.
    3. **Parallel Execution**: Allows pipeline stages to run concurrently across CPUs.

---

### **6. File System**

- **Tree Structure**: Organized as uninterpreted byte arrays (files) and directory trees starting from the root directory (`/`).
- **Inodes & Metadata**: An **inode** holds file metadata including file type (file, directory, or device), length, link count, and disk block mapping.
- **File Status**: `fstat()` fills a `struct stat` with fields such as `dev`, `ino`, `type`, `nlink`, and `size`.

---

### **7. Real-World Context & Limitations**

- **POSIX Alignment**: xv6 provides a simplified subset of the standard POSIX UNIX call interface (omitting calls like `lseek`).
- **Security Model**: xv6 does not enforce multi-user isolation; all xv6 process operations run with root privileges.

---

Here is a detailed breakdown of **Chapter 1: Operating system interfaces** from the _xv6-riscv_ book.

---
# ch 1

### **1. Overview of Chapter 1**

An operating system shares hardware among multiple applications, provides abstractions (so applications do not need to manage raw disk or memory hardware directly), and enables controlled inter-process communication.

- **User Space vs. Kernel Space**:
    - **Process**: A running program with its own private memory containing instructions, data, and a stack.
    - **Kernel**: A special privileged program that manages hardware resources.
    - **System Calls**: When a user process requires a service from the kernel, it issues a system call, crossing the boundary from user space into kernel space.
- **Design Philosophy**: xv6 implements a narrow, simplified subset of the classic Unix system call interface (pioneered by Ken Thompson and Dennis Ritchie), which relies on a few simple mechanisms that combine with high generality.

---

### **2. Detailed Section Breakdown**

#### **1.1 Processes and Memory**

- **Process State**: Consists of user-space memory (instructions, data, stack, heap) and per-process kernel state (saved registers, PID, run state).
- **Process Creation (`fork`)**:
    - `int fork()` creates a new **child process** as an exact copy of the parent process's memory.
    - **Return Values**: Returns the child's PID to the parent process, and `0` to the child process.
    - Parent and child execute with separate memory address spaces and separate registers; modifying a variable in one does not affect the other.
- **Process Termination (`exit`) and Waiting (`wait`)**:
    - `exit(int status)` stops execution, releases resources (memory and open files), and reports the status integer (0 = success, 1 = failure).
    - `wait(int *status)` blocks the parent until a child exits, returning the exited child's PID and copying its exit status into `*status`.
- **Program Execution (`exec`)**:
    - `exec(char *file, char *argv[])` loads an ELF executable binary from disk, overwriting the calling process's current memory image.
    - Does **not** return upon success; execution begins at the entry point specified in the ELF header.
- **Why Separate `fork` and `exec`?**:
    - Decoupling creation (`fork`) from execution (`exec`) gives the Unix shell a window of opportunity to manipulate file descriptors (for I/O redirection and pipelines) in the child process before running the new binary.
- **Dynamic Memory (`sbrk`)**:
    - `char *sbrk(int n)` grows or shrinks process data memory by `n` bytes on the heap and returns the address of the newly allocated region.

---

#### **1.2 I/O and File Descriptors**

- **File Descriptor Abstraction**:
    - A file descriptor is a small integer representing an in-kernel object (a file, directory, device, or pipe) abstracted as a stream of bytes.
    - Every process has a private per-process file descriptor table indexed starting at 0.
    - **Standard Descriptors**: By convention, `0` is standard input (`stdin`), `1` is standard output (`stdout`), and `2` is standard error (`stderr`).
- **Lowest-Numbered Unused Descriptor Rule**:
    - Kernel functions like `open()`, `dup()`, or `pipe()` always allocate the **lowest available integer index** in the process file table.
- **Basic I/O Calls**:
    - `read(fd, buf, n)` reads up to `n` bytes at the current file offset and advances the offset accordingly; returns `0` at End-of-File (EOF).
    - `write(fd, buf, n)` writes `n` bytes at the current file offset and advances the offset.
    - `close(fd)` releases the descriptor for reuse.
- **I/O Redirection in Shell**:
    1. The shell forks a child process.
    2. The child closes standard input (`close(0)`).
    3. Opening a file (`open("input.txt", O_RDONLY)`) automatically reuses descriptor `0`.
    4. The child calls `exec("cat", argv)`—`cat` reads from fd 0 without knowing or caring that input is coming from a file rather than the console.
- **Shared Offsets via `fork` and `dup`**:
    - `fork()` copies the file descriptor table, but parent and child share the underlying file offset.
    - `dup(fd)` creates a duplicate file descriptor pointing to the same file and sharing the same offset (e.g., `2>&1`).

---

#### **1.3 Pipes**

- **Pipe Abstraction**: A small kernel buffer exposed as a pair of file descriptors (`p` for reading, `p` for writing) for Inter-Process Communication (IPC).
- **Blocking & EOF**:
    - Reading from an empty pipe blocks until data is written or until all write descriptors referencing the pipe are closed (returning `0` for EOF).
    - It is critical for processes to close unused write ends of a pipe so readers do not wait indefinitely for EOF.
- **Advantages over Temporary Disk Files**:
    1. **Automatic Cleanup**: Disappears automatically when descriptors close.
    2. **Unlimited Data Stream**: Handles arbitrary data stream sizes without consuming disk space.
    3. **Parallel Execution**: Allows stages of a pipeline (e.g., `grep | wc`) to run concurrently across CPUs.

---

#### **1.4 File System**

- **Hierarchical Tree**: Organized as files (uninterpreted byte arrays) and directories starting from the root (`/`).
- **Working Directory**: Paths not starting with `/` are evaluated relative to the current directory (`chdir()`).
- **System Calls**: `mkdir()` creates directories; `open(..., O_CREATE)` creates files; `mknod()` creates special device files (associated with major/minor numbers).
- **Inodes and Hard Links**:
    - File contents reside in an **inode** (holding metadata like file type, length, and disk block mapping).
    - A file's name is distinct from the file itself; directory entries (links) associate filenames with inode numbers (`link()`, `unlink()`).
- **`fstat(fd, struct stat *st)`**: Retrieves inode metadata for an open file descriptor into a `struct stat` structure.

---

#### **1.5 Real World**

- **POSIX Standard**: Modern operating systems (Linux, BSD, macOS) standardize these Unix system calls via POSIX.
- **xv6 Simplifications**: xv6 is not fully POSIX-compliant; it omits calls like `lseek`, lacks multi-user permission isolation (all processes run as root), and lacks networking or graphical user interfaces.

---

### **3. Figures and Tables in Chapter 1**

When documenting or referencing Chapter 1, note the following figures, tables, and code pointers:

1. **Figure 1.1: User space, kernel space, and system calls**
    
    - **Location**: Page 10 of the text.
    - **Description**: A structural diagram illustrating the boundary between user space and kernel space:
        - Shows user processes (`shell` and `cat`) executing in **user space** above the red user/kernel line.
        - Shows an arrow representing a **system call** entering the **Kernel** in **kernel space** below the line.
2. **Figure 1.2: Xv6 system calls (Summary Table)**
    
    - **Location**: Page 11 of the text.
    - **Description**: A summary table listing all 21 xv6 system calls, their function signatures, and brief descriptions:
        - Process control: `fork()`, `exit()`, `wait()`, `kill()`, `getpid()`, `pause()`, `exec()`, `sbrk()`.
        - File I/O & descriptors: `open()`, `write()`, `read()`, `close()`, `dup()`, `pipe()`.
        - File system & directory: `chdir()`, `mkdir()`, `mknod()`, `fstat()`, `link()`, `unlink()`.
        - _Note_: Unless otherwise stated, all system calls return `0` on success and `-1` on error.
3. **Source Code References (`xv6-src-booklet.pdf` Line Numbers)**:
    
    - Shell main loop (`main`): line `8001`
    - Shell command parser/runner (`runcmd`): line `7903`
    - Shell execution (`exec` call): line `7927`
    - Shell I/O redirection setup: line `7931`
    - Shell pipeline setup: line `7950`

---

💡 **Next Step**: Would you like to move on to **Chapter 2: Operating system organization** to see how RISC-V user/supervisor modes, monolithic kernels, and process memory address spaces are structured?

---

# Chapter 2: Operating system organization

---

### **1. Core Role & Key Requirements of an OS**

An operating system must support multiple concurrent activities (such as running a compiler and a text editor simultaneously) while fulfilling three primary requirements:

1. **Multiplexing**: Efficiently time-sharing physical hardware resources (CPUs, RAM, storage) among multiple processes.
2. **Isolation**: Ensuring that a bug or crash in one process cannot corrupt or disturb unrelated processes or the kernel itself.
3. **Interaction**: Providing mechanisms for processes to intentionally communicate and share data (e.g., shell pipelines).

---

### **2. Detailed Section Breakdown**

#### **2.1 Abstracting Physical Resources**

- Instead of allowing applications to access physical hardware directly (which makes isolation impossible), the OS provides abstract resources:
    - **File Systems** abstract disk sectors into files and directories.
    - **Processes** abstract physical CPUs into virtual CPUs.
    - **Address Spaces** abstract physical RAM into private virtual memory.

#### **2.2 User Mode, Supervisor Mode, and System Calls**

- **Hardware Privilege Levels**: RISC-V CPUs provide three execution modes to enforce strong isolation:
    - **Machine Mode**: Highest privilege level. Executes brief boot code to set up hardware and transitions quickly to supervisor mode.
    - **Supervisor Mode**: Kernel mode. Allows execution of privileged hardware instructions (e.g., enabling/disabling interrupts, writing the page table root register `satp`).
    - **User Mode**: Restricted execution mode for user applications. Attempting to execute a privileged instruction in user mode triggers a hardware **trap** into supervisor mode.
- **System Calls**: When a user-space program requires kernel services, it executes an `ecall` instruction, raising CPU privileges and transferring control to a pre-arranged kernel handler.

#### **2.3 Kernel Organization: Monolithic vs. Microkernel**

- **Monolithic Kernel** (used by xv6 and Linux/Unix):
    - The entire operating system (file system, process manager, memory manager, drivers) runs inside supervisor mode as a single unified program.
    - _Advantages_: Easy for OS components to share data structures (e.g., sharing a disk block cache between virtual memory and file system) and low system-call overhead.
- **Microkernel**:
    - Minimizes kernel code in supervisor mode to a tiny core (IPC, basic thread scheduling, address spaces).
    - OS services (file servers, device drivers, shells) run as separate user-space server processes communicating via IPC messages.
    - _Advantages_: Better modularity and fault tolerance, but higher message-passing overhead.

#### **2.4 Code: xv6 Organization**

- xv6 runs on a simulated 64-bit RISC-V multi-core CPU (via QEMU `-machine virt`).
- It is written in **LP64 C** (pointers `P` and longs `L` are 64 bits, integers are 32 bits).
- All kernel source files reside in the `kernel/` directory and are organized by functional area (booting, processes, traps, memory, drivers, file system, locks).

#### **2.5 Process Overview & Address Space**

- **The Process Abstraction**: The fundamental unit of isolation in xv6. Bundles two key ideas:
    1. **An Address Space**: Gives the program the illusion of a private memory system.
    2. **A Thread of Control**: Gives the program the illusion of a private CPU.
- **Virtual Address Space Layout** (Figure 2.3):
    - Starts at virtual address `0` containing **user text and data**, followed by the **user stack**, and the **heap** (grows upward).
    - RISC-V Sv39 uses 38 bits for virtual addresses in xv6, making the maximum virtual address \(\text{MAXVA} = 2^{38}-1 = \text{0x3fffffffff}\).
    - At the very top of every process address space reside two special unmapped/kernel-accessible pages: the **trampoline page** (contains code to switch into kernel mode) and the **trapframe page** (holds saved user registers).
- **Two Stacks per Process**:
    - **User Stack**: Used while executing user instructions.
    - **Kernel Stack (`p->kstack`)**: Used whenever the process enters kernel space (via system call or interrupt). Having a separate kernel stack protects the kernel if a program corrupts its user stack.
- **Kernel State (`struct proc`)**:
    - Defined in `kernel/proc.h` (line 2034).
    - Maintains fields like `p->pagetable` (pointer to page table), `p->kstack` (kernel stack pointer), `p->state` (`UNUSED`, `EMBRYO`, `SLEEPING`, `RUNNABLE`, `RUNNING`, `ZOMBIE`), and `p->pid`.

#### **2.6 Code: Starting xv6 & The First Process**

- **Boot Sequence**:
    1. RISC-V CPU powers on in **Machine Mode** with paging disabled.
    2. ROM boot loader copies the xv6 kernel to physical RAM address `0x80000000` (addresses `0x0` to `0x80000000` are reserved for memory-mapped I/O devices).
    3. Boot loader jumps to `_entry` in `kernel/entry.S` (line 1006).
    4. `_entry` configures the initial stack pointer `sp` using `stack0 + 4096` in `kernel/start.c` (line 1060) and calls `start()` (line 1064).
    5. `start()` performs machine-mode setup, configures timer interrupts, sets CPU mode to Supervisor Mode, and jumps to `main()` in `kernel/main.c`.
    6. `main()` initializes physical memory (`kinit`), page tables (`kvminit`), process table (`userinit`), and launches the first user process (`user/init.c`), which spawns the shell.

#### **2.7 Security Model**

- xv6 assumes a simplified security model: it does not enforce multi-user permissions or user isolation; all xv6 processes effectively execute as `root`.

---

### **3. Figures, Tables, and Referenced Items in Chapter 2**

When summarizing or documenting Chapter 2, refer to the following figures, tables, and source code locations:

1. **Figure 1.1: A kernel and two user processes** _(referenced from Chapter 1)_
    
    - **Description**: Diagrams the fundamental boundary separating **User Space** (user mode, running processes like `shell` and `cat`) from **Kernel Space** (supervisor mode, single monolithic kernel), connected via **system calls**.
2. **Microkernel vs. Monolithic Diagram** _(Page 24 text diagram)_
    
    - **Description**: Shows user-space processes (e.g., `shell`, `File server`) sending IPC messages through a minimal **Microkernel** in kernel space.
3. **Figure 2.2: Xv6 kernel source files**
    
    - **Description**: A comprehensive summary table categorizing all kernel source files by responsibility:
        - **Boot**: `entry.S` (first instructions), `start.c` (machine-mode boot), `main.c` (C initialization).
        - **Processes**: `proc.c` / `proc.h` (process control), `exec.c` (`exec` call), `swtch.S` (context switching), `sysproc.c` (process system calls).
        - **Traps**: `trampoline.S` / `kernelvec.S` (trap vectors), `trap.c` (trap handling), `syscall.c` (system call dispatch).
        - **Memory**: `vm.c` (page tables), `kalloc.c` (physical allocator).
        - **Devices & FS**: `console.c`, `uart.c`, `virtio_disk.c`, `bio.c`, `fs.c`, `log.c`, `file.c`, `pipe.c`.
        - **Misc**: `spinlock.c`, `sleeplock.c`.
4. **Figure 2.3: Layout of a process's virtual address space**
    
    - **Description**: Illustrates the virtual address space map from virtual address `0` up to `MAXVA` (`0x3fffffffff`):
        - `0`: `user text and data`.
        - Next: `user stack`.
        - Next: `heap` (grows upward).
        - Top of space: `trapframe` page followed by `trampoline` page.
5. **Key Code References (`xv6-src-booklet.pdf` Line Numbers)**:
    
    - `MAXVA` constant definition: line `0899`
    - `_entry` boot point (`kernel/entry.S`): line `1006`
    - `stack0` boot stack (`kernel/start.c`): line `1060`
    - `start()` function (`kernel/start.c`): line `1064`
    - `struct proc` definition (`kernel/proc.h`): line `2034`

---

💡 **Next Step**: Would you like to move on to **Chapter 3: Page tables** to explore Sv39 3-level page table walks, `walk()`, `mappages()`, and `exec()` address space creation in detail?

---

# Chapter 3: Page tables

---

### **1. Overview & Purpose of Page Tables**

Page tables provide each process with its own private, isolated virtual address space and memory. They translate virtual addresses used by RISC-V instructions into physical addresses sent to RAM hardware.

By providing a level of indirection, page tables enable key OS mechanisms:

- **Isolation & Protection**: Preventing processes from reading or altering each other's memory.
- **Multiplexing**: Mapping multiple sparse virtual address spaces onto a single physical memory.
- **Memory Tricks**: Mapping the same physical page (e.g., the trampoline page) into multiple address spaces, protecting stacks with unmapped guard pages, and allocating heap memory lazily.

---

### **2. Detailed Section Breakdown**

#### **3.1 Paging Hardware**

- **Sv39 RISC-V Mode**: xv6 uses RISC-V's Sv39 configuration:
    - Virtual addresses are 64 bits wide, but only the bottom **39 bits** are used (top 25 bits are ignored). The maximum virtual address is \(\text{MAXVA} = 2^{38}-1 = \text{0x3fffffffff}\).
    - Physical addresses are **56 bits** wide (44-bit Physical Page Number / PPN + 12-bit offset).
    - Pages are fixed-sized chunks of **4096 bytes** (\(2^{12}\) bytes).
- **3-Level Tree Page Table** (Figure 3.2):
    - A flat page table would require \(2^{27}\) entries (~134 million PTEs consuming 4 MB per process). Instead, RISC-V uses a **3-level tree**.
    - The 27-bit Virtual Page Number (VPN) is split into three 9-bit indices: `L2` (top 9 bits), `L1` (middle 9 bits), and `L0` (bottom 9 bits).
    - Each level directory page is 4096 bytes containing \(512\) (\(2^9\)) Page Table Entries (PTEs).
    - **Memory Savings**: Unallocated ranges of virtual memory omit entire intermediate/bottom page directories, saving significant RAM for sparse address spaces.
- **Page Table Entries (PTEs) & Flags**:
    - Each 64-bit PTE contains a 44-bit PPN and permission/status flags:
        - **`PTE_V`**: Valid bit (if clear, accessing the address triggers a page fault).
        - **`PTE_R`**: Readable.
        - **`PTE_W`**: Writable.
        - **`PTE_X`**: Executable (allows CPU to execute instructions from the page).
        - **`PTE_U`**: User accessible (if clear, usable only in supervisor mode).
- **`satp` Register & TLB**:
    - The OS writes the physical address of a process's root page table into the CPU's **`satp`** (Supervisor Address Translation and Protection) register. Each CPU has its own `satp` register.
    - CPUs cache PTE translations in a **Translation Look-aside Buffer (TLB)**.
    - When changing page tables, xv6 executes the **`sfence.vma`** instruction to flush cached TLB entries on the local CPU.

---

#### **3.2 Kernel Address Space**

- **Single Kernel Page Table**: xv6 creates a single page table for the kernel used by all CPUs executing in supervisor mode.
- **Direct Mapping** (Figure 3.3):
    - Physical RAM starts at `0x80000000` (`KERNBASE`) and extends to `0x88000000` (`PHYSTOP`).
    - The kernel direct-maps RAM and I/O device registers: virtual address \(X\) maps to physical address \(X\).
    - Device control registers below `0x80000000` (e.g., UART0, VIRTIO disk, PLIC, CLINT, boot ROM) are mapped to identical virtual addresses so the kernel can interact with hardware via simple loads and stores.
- **Non-Direct Mapped Regions in Kernel Virtual Memory**:
    1. **Trampoline Page**: Mapped at the very top of the virtual address space (`MAXVA`), mapped to the same physical page holding trap-handling code in both kernel and user page tables.
    2. **Kernel Stacks (`KSTACK`)**: Each process receives its own kernel stack mapped high in kernel memory. Below each stack, xv6 leaves an unmapped **guard page** (`PTE_V` cleared). If a kernel stack overflows, accessing the guard page immediately triggers a kernel panic rather than corrupting adjacent memory.

---

#### **3.3 Code: Creating an Address Space (`kernel/vm.c`)**

- **Key Data Structure**: `pagetable_t` (a pointer to a root 4096-byte page table page).
- **Core Functions**:
    - **`walk(pagetable, va, alloc)`**: Software implementation of the RISC-V hardware page walk. Descends the 3-level tree using the 9-bit L2, L1, and L0 indices. If an intermediate directory page is missing and `alloc != 0`, it allocates a new page directory via `kalloc()`. Returns a pointer to the target PTE.
    - **`mappages(pagetable, va, size, pa, perm)`**: Maps a range of virtual addresses to physical addresses by calling `walk()` for each page and populating the PTE with PPN, `perm` flags, and `PTE_V`.
    - **`kvmmake()` / `kvminit()`**: Allocates the root kernel page table and installs direct mappings for kernel code/data, RAM up to `PHYSTOP`, I/O devices, and process kernel stacks (`proc_mapstacks`).
    - **`kvminithart()`**: Writes the physical root page table address into `satp` and executes `sfence.vma`.
    - **`copyin()` / `copyout()`**: Explicitly translates user virtual addresses to physical addresses using `walkaddr()` to safely copy system call arguments between user and kernel memory.

---

#### **3.4 & 3.5 Physical Memory Allocation (`kernel/kalloc.c`)**

- **Page Allocator**: Allocates and frees physical RAM between the end of the kernel binary (`end`) and `PHYSTOP` in whole 4096-byte pages.
- **Free List**:
    - Threaded linked list of `struct run` entries stored directly inside the free pages themselves.
    - Protected against concurrent multi-CPU access by **`kmem.lock`**.
- **Functions**:
    - `kinit()`: Initializes `kmem.lock` and adds all physical pages between `end` and `PHYSTOP` (~128 MB) to the free list via `freerange()` and `kfree()`.
    - `kalloc()`: Pops a free page from the linked list, zeroes it out, and returns its physical address.
    - `kfree()`: Casts the freed memory address, writes a `struct run` pointer into it, and pushes it onto the free list.

---

#### **3.6 Process Address Space**

- Each process has its own page table. Switch process \(\rightarrow\) write new page table root to `satp`.
- **User Layout** (Figure 3.4):
    - **Virtual Address `0`**: Program text (mapped `PTE_R`, `PTE_X`, `PTE_U`—read-only code prevents self-modification bugs).
    - **Data**: Pre-initialized program variables (`PTE_R`, `PTE_W`, `PTE_U`).
    - **Guard Page**: Placed directly below the user stack with `PTE_U` cleared to catch stack overflows.
    - **User Stack**: Single page allocated by `exec` (`PTE_R`, `PTE_W`, `PTE_U`).
    - **Heap**: Grows upward dynamically via `sbrk()`.
    - **High Virtual Memory**: `TRAPFRAME` page (`0x3fffffe000`) followed by `TRAMPOLINE` page (`0x3ffffff000`).

---

#### **3.7 Code: `exec` (`kernel/exec.c`)**

- Loads an ELF binary file from disk to replace the calling process's user memory image.
- **Steps**:
    1. Opens file via `namei()` and reads the ELF header (`struct elfhdr`).
    2. Validates ELF magic number (`0x7F 'E' 'L' 'F'`).
    3. Creates an empty user page table via `proc_pagetable()`.
    4. Iterates over program segment headers (`struct proghdr`):
        - Calls `uvmalloc()` to allocate physical pages.
        - Calls `loadseg()` to read segment contents from disk into physical memory using `walkaddr()`.
    5. Allocates 2 pages at the top of user space: 1 unmapped guard page and 1 user stack page.
    6. Copies command-line strings (`argv`) onto the top of the user stack, pushes pointers to `argv[]` strings, sets up `main(argc, argv)` stack parameters.
    7. Frees the old user page table and installs the new page table in the process structure.

---

### **3. Figures and Tables in Chapter 3**

If you are documenting or summarizing Chapter 3, refer to the following figures and diagrams:

1. **Figure 3.1: An abstract view of a flat page table mapping virtual to physical addresses**
    
    - **Location**: Page 31 of the text.
    - **Description**: A conceptual diagram showing a 39-bit virtual address (27-bit VPN + 12-bit offset) indexing into a flat array of \(2^{27}\) PTEs to form a 56-bit physical address (44-bit PPN + 12-bit offset).
2. **Figure 3.2: RISC-V address translation details**
    
    - **Location**: Page 32–33 of the text.
    - **Description**: Detailed hardware translation diagram showing:
        - Virtual address breakdown: 9-bit `L2`, 9-bit `L1`, 9-bit `L0`, and 12-bit `Offset`.
        - `satp` register pointing to the Level 2 root Page Directory \(\rightarrow\) Level 1 Page Directory \(\rightarrow\) Level 0 Page Directory \(\rightarrow\) Physical Page Frame.
        - PTE Bit-level layout: 44-bit PPN field, Reserved bits (`RSW`), and Flag bits: `G` (Global), `D` (Dirty), `A` (Accessed), `U` (User), `X` (Executable), `W` (Writable), `R` (Readable), `V` (Valid).
3. **Figure 3.3: Kernel virtual address space vs. RISC-V physical address space**
    
    - **Location**: Page 34 of the text.
    - **Description**: Side-by-side comparison diagram:
        - **Left (Kernel Virtual Address Space)**: Top `Trampoline` (`R-X`), Guard page, Kernel stacks (`Kstack 0`, `Kstack 1`...), Free memory (`RW-`), Kernel data (`RW-`), Kernel text (`R-X`), PLIC (`RW-`), CLINT, boot ROM, UART0 (`RW-`), VIRTIO disk (`RW-`).
        - **Right (Physical Address Space)**: RAM from `0x80000000` (`KERNBASE`) to `0x88000000` (`PHYSTOP`), and memory-mapped I/O devices below `0x80000000`.
4. **Figure 3.4: A process’s user address space, with its initial stack**
    
    - **Location**: Page 38–39 of the text.
    - **Description**: Structural diagram of a process's user virtual memory space from `0` to `MAXVA`:
        - Virtual address `0` \(\rightarrow\) `text` (`R-XU`) \(\rightarrow\) `data` (`R-WU`) \(\rightarrow\) unmapped gap \(\rightarrow\) `guard page` (inaccessible) \(\rightarrow\) `stack` (`R-WU`) \(\rightarrow\) `heap` (`R-WU`) \(\rightarrow\) `trapframe` (`R-W-`) \(\rightarrow\) `trampoline` (`R-X-`) at `MAXVA`.
        - **Inset detail**: Zoomed-in layout of the initial user stack created by `exec()`, showing command line strings (`argument 0` ... `argument N`), `0` terminator, `argv` pointer array (`argv[N]` ... `argv`), address of `argv`, and unallocated stack frame space.

---

💡 **Next Step**: Would you like to move on to **Chapter 4: Traps and system calls** to examine RISC-V control registers (`stvec`, `sepc`, `scause`, `sscratch`), `trampoline.S`, `uservec`, and system call handling?


----

# Ch 4 Traps and system calls

### **1. Overview & Core Purpose of Chapter 4**

A **trap** occurs when an event forces the CPU to interrupt normal instruction execution and transfer control to special kernel handler code. xv6 categorizes traps into three distinct types:

1. **System calls**: Initiated when user code executes the `ecall` instruction to request a kernel service.
2. **Exceptions**: Triggered when user or kernel instructions perform illegal actions, such as referencing an unmapped virtual address.
3. **Device interrupts**: Raised by hardware devices (e.g., disk I/O completion or UART serial input) requesting kernel attention.

Traps are designed to be **transparent**, meaning the interrupted code eventually resumes without being aware of the interruption. xv6 handles traps across four stages: hardware actions by the RISC-V CPU, assembly vector setup, a C handler function, and the underlying system call or driver service routine.

---

### **2. Detailed Section Breakdown**

#### **4.1 RISC-V Trap Machinery**

The RISC-V CPU uses several privileged **Control and Status Registers (CSRs)** to manage trap handling:

- **`stvec`**: Holds the address of the kernel trap handler (`uservec` or `kernelvec`).
- **`sepc`**: Stores the saved program counter (`pc`) when a trap occurs. Executing `sret` copies `sepc` back to `pc`.
- **`scause`**: Stores a numeric code indicating the reason for the trap.
- **`sscratch`**: A scratch register used by assembly trap code to hold a pointer before saving user registers.
- **`sstatus`**: Contains status flags, including **`SIE`** (enables/disables supervisor interrupts) and **`SPP`** (records the previous privilege mode).

**Hardware Actions Taken on a Trap**:

1. Disables interrupts by clearing `sstatus.SIE`.
2. Saves `pc` into `sepc`.
3. Saves current CPU privilege mode in `sstatus.SPP`.
4. Writes the trap cause into `scause`.
5. Sets CPU privilege mode to supervisor mode.
6. Copies `stvec` to `pc` and begins executing instructions at the new address.

_Note_: The hardware does **not** switch page tables, switch stacks, or save general registers (other than `pc`)—these tasks must be performed by software.

---

#### **4.2 Traps from User Space**

- **Page Table Constraint & Trampoline Page**: The RISC-V hardware does not switch page tables when a trap occurs. Therefore, the handler address in `stvec` must be mapped in the user page table. xv6 maps a single **trampoline page** holding `uservec` at virtual address `TRAMPOLINE` (`0x3ffffff000`) in both user and kernel address spaces.
- **`uservec` (`kernel/trampoline.S`)**:
    - Uses `csrw` to swap register `a0` with `sscratch`, freeing `a0` to hold a memory pointer.
    - Sets `a0` to `TRAPFRAME` (`0x3fffffe000`), a page mapped directly below the trampoline page.
    - Saves all 32 user registers into the process's `p->trapframe` structure.
    - Retrieves saved kernel state from the trapframe: kernel stack pointer, CPU `hartid`, address of `usertrap()`, and root kernel page table address.
    - Writes the kernel page table root into `satp` and jumps to `usertrap()`.
- **`usertrap()` (`kernel/trap.c`)**:
    - Sets `stvec` to `kernelvec` so any subsequent trap while inside the kernel will be handled as a kernel trap.
    - Saves `sepc` in the process structure.
    - If the trap is a system call (`ecall`), increments saved `sepc` by 4 (so return execution resumes at the instruction _after_ `ecall`) and calls `syscall()`.
    - If a device interrupt, calls `devintr()`; if a page fault, calls `vmfault()`; otherwise terminates the faulting process.
- **`prepare_return()` & `userret` (`kernel/trampoline.S`)**:
    - `prepare_return()` sets `stvec` back to `uservec`, sets up trapframe fields, restores `sepc`, and calls `userret`.
    - `userret` writes the process's user page table into `satp`, restores user registers from `TRAPFRAME`, restores user `a0`, and executes `sret` to return to user mode.

---

#### **4.3 Code: Calling System Calls**

- User programs invoke system calls via C library wrappers (e.g., in `user/usys.S`).
- For example, `write(2, "$ ", 2)` loads arguments into `a0`, `a1`, and `a2`, places system call number `SYS_write` (16) into `a7`, and executes `ecall`.
- `syscall()` in `kernel/syscall.c` fetches `a7` from `p->trapframe->a7`, indexes into the `syscalls[]` function table, calls `sys_write()`, and places the return value into `p->trapframe->a0`.

---

#### **4.4 Code: System Call Arguments**

- Kernel helper functions `argint()`, `argaddr()`, and `argfd()` retrieve system call arguments from `p->trapframe`.
- **Validating User Pointers**: User programs may pass invalid or malicious pointers. Kernel functions like `copyin()`, `copyout()`, and `copyinstr()` safely transfer data between user virtual addresses and kernel memory.
- **Address Translation (`walkaddr`)**: `copyinstr()` uses `walkaddr()` (which calls `walk()`) to inspect the user page table and translate user virtual addresses into physical addresses, checking that virtual addresses fall within valid user memory boundaries.

---

#### **4.5 Traps from Kernel Space**

- While executing in kernel mode, `stvec` points to `kernelvec` in `kernel/kernelvec.S`.
- `kernelvec` relies on `satp` already pointing to the kernel page table and `sp` pointing to a valid kernel stack.
- Saves all 32 registers onto the current thread's kernel stack.
- Jumps to `kerneltrap()` in `kernel/trap.c`.
- Handles device interrupts via `devintr()`. If a timer interrupt occurs and a process thread is running, `kerneltrap()` calls `yield()` to force a context switch.
- If an unexpected exception occurs inside kernel code, `kerneltrap()` calls `panic()`.
- After handling the trap, `kernelvec` pops registers off the kernel stack and executes `sret` to resume the interrupted kernel code.

---

#### **4.6 Real World**

- Many commercial operating systems map kernel memory directly into user page tables (with user permissions `PTE_U` cleared) to eliminate page table switches and trampoline pages during traps.
- xv6 keeps kernel and user page tables separate to increase security and reduce the likelihood of kernel security vulnerabilities resulting from inadvertent pointer dereferences.

---

### **3. Figures, Tables, and Referenced Items in Chapter 4**

Refer to the following figures, registers, and source code references when reviewing Chapter 4:

1. **Figure 4.1: Outline of how a trap from user code is handled**
    
    - **Location**: Page 45 of the xv6 text.
    - **Description**: A flow diagram outlining the high-level execution path for traps originating from user code: \[\text{User Code} \rightarrow \text{trampoline uservec} \rightarrow \text{usertrap} \rightarrow \text{syscall or device driver} \rightarrow \text{trampoline userret} \rightarrow \text{User Code}\]
2. **Core RISC-V CSR Registers (Section 4.1)**:
    
    - `stvec` (handler vector address)
    - `sepc` (saved user/kernel program counter)
    - `scause` (trap cause identifier)
    - `sscratch` (scratch register for register saving)
    - `sstatus` (`SIE` interrupt enable and `SPP` privilege state)
3. **Key Source Code References (`xv6-src-booklet.pdf` Line Numbers)**:
    
    - `uservec` (`kernel/trampoline.S`): line `3071`
    - `userret` (`kernel/trampoline.S`): line `3151`
    - `usertrap()` (`kernel/trap.c`): line `3337`
    - `prepare_return()` (`kernel/trap.c`): line `3404`
    - `syscall()` (`kernel/syscall.c`): line `3731`
    - `kernelvec` (`kernel/kernelvec.S`): line `3211`
    - `kerneltrap()` (`kernel/trap.c`): line `3453`
    - `devintr()` (`kernel/trap.c`): line `3506`
    - `walkaddr()` (`kernel/vm.c`): line `1520`
    - `copyinstr()` (`kernel/vm.c`): line `1833`

---

💡 **Next Step**: Would you like to review **Chapter 5: Page faults** to examine lazy allocation, Copy-On-Write (COW) fork, and demand paging?

---
# ch 5

### **1. Overview & Hardware Machinery of Page Faults**

- **Page-Fault Exceptions**: A RISC-V CPU raises a page-fault exception whenever an instruction uses a virtual address that has no mapping in the page table, has a Page Table Entry (PTE) with `PTE_V == 0` (invalid mapping), or violates access permissions (e.g., trying to write to a read-only page).
- **Types of Page Faults**: RISC-V distinguishes between three specific types of page faults:
    1. **Load page faults** (caused by load instructions).
    2. **Store page faults** (caused by store instructions).
    3. **Instruction page faults** (caused by instruction fetches).
- **Hardware Registers**: Upon a page fault, the hardware updates two key control registers:
    - **`scause`**: Contains a number indicating the specific cause/type of page fault.
    - **`stval`**: Holds the exact faulting virtual address that failed translation.
- **The Power of Indirection**: Combining page tables with page-fault handling gives the kernel a powerful level of indirection. It allows the operating system to intercept memory reads and writes and adjust page table mappings dynamically on the fly.

---

### **2. Lazy Allocation**

- **The Concept**: In standard eager allocation, calling `sbrk(n)` immediately allocates physical RAM via `kalloc()` and sets up page table entries for the whole requested range. However, programs often request memory for worst-case scenarios that they may not fully use. Under **lazy allocation** (`SBRK_LAZY`), the kernel simply increments the process's recorded size (`myproc()->sz`) without allocating physical memory or creating PTEs upfront.
- **Page Fault Handling (`vmfault`)**:
    - When a process accesses an address in the lazily grown region for the first time, the CPU generates a page fault.
    - `usertrap()` catches the page fault and calls `vmfault()`.
    - `vmfault()` verifies that the faulting address lies within the valid range granted by `sbrk()`.
    - It then calls `kalloc()` to allocate a 4096-byte physical page, zeros it out, and maps it into the user page table using `mappages()` with permissions `PTE_W`, `PTE_R`, `PTE_U`, and `PTE_V`.
    - Control returns to user space, and the instruction that caused the fault is re-executed, succeeding smoothly this time.
- **Trade-Offs**: Lazy allocation spreads the allocation cost over time and saves RAM for unused pages. However, if physical memory is exhausted when a page fault occurs, the kernel cannot easily return an allocation error to the application and must kill the process. Applications that require guaranteed memory can request eager allocation (`SBRK_EAGER`) via `growproc()`.

---

### **3. Implementation Details (`sys_sbrk` & Memory Deallocation)**

- **Growing/Shrinking Memory**: The system call `sbrk(n)` is implemented by `sys_sbrk()`. If \(n < 0\), it calls `shrinkproc()`, which delegates the actual deallocation to `uvmunmap()`.
- **Skipping Unmapped Pages**: Because lazily allocated pages may never have been accessed or faulted in, `uvmunmap()` walks the page table and explicitly skips PTEs where `PTE_V` is not set. For valid PTEs (`PTE_V` set), it calls `kfree()` to free the physical page.
- **Page Table as Truth**: xv6 relies on the user page table as its **only record** of which physical RAM pages are currently allocated to a process.

---

### **4. Real-World Applications of Page Faults**

#### **A. Copy-On-Write (COW) Fork**

- **The Problem**: Standard `fork()` copies the entire address space of the parent process into new physical pages for the child, which is slow and memory-intensive.
- **The COW Strategy**: The parent and child share the same physical pages initially, but both map them as **read-only** (clearing `PTE_W`).
- **Handling Writes**: If either process attempts to write to a shared page, a store page fault is raised. The COW page-fault handler allocates a new physical page, copies the shared content into it, sets `PTE_W` (read/write), updates the faulting process's PTE to point to its private copy, and resumes execution.
- **Optimization**: COW makes `fork()` virtually instantaneous and avoids unnecessary copying—especially when followed immediately by `exec()`, where the child discards inherited memory anyway.

#### **B. Demand Paging & Disk Swapping**

- **Demand Paging**: Instead of loading an entire executable binary from disk during `exec()`, the kernel creates the page table with all PTEs marked invalid. When the program executes, pages of code and data are read from disk into RAM on demand as page faults occur.
- **Paging to Disk (Swapping)**: When physical RAM is full, the kernel can evict less frequently used pages to a dedicated paging/swap area on disk and mark their PTEs invalid. Accessing an evicted page triggers a page fault, prompting the kernel to read it back into RAM (evicting another page first if necessary).

#### **C. Memory-Mapped Files (`mmap`) & Stack Extension**

- **Memory-Mapped Files**: Programs map file contents directly into their virtual address space using `mmap()`. Applications read and write the file using standard load and store instructions, while page faults fetch missing file blocks from disk on demand.
- **Automatic Stack Growth**: The kernel can automatically allocate additional stack pages when a user stack overflow triggers a page fault just below the current stack boundary.

---
# Ch 6

### **1. Overview of Chapter 6**

A **driver** is the kernel code that manages a specific hardware device: it configures device hardware, issues commands, handles interrupts, and manages interaction between user processes and devices.

Chapter 6 focuses on how xv6 handles hardware devices concurrently and safely using **top-half and bottom-half driver architectures**, memory-mapped I/O, the RISC-V **PLIC** (Platform-Level Interrupt Controller), UART serial devices, and timer interrupts.

---

### **2. Driver Architecture: Top Half vs. Bottom Half**

Device drivers in xv6 operate across two execution contexts:

1. **Top Half**:
    - Runs in a process's kernel thread when a process issues system calls like `read` or `write`.
    - Asks the hardware to start an I/O operation (e.g., requesting a disk block or reading a console line) and puts the calling thread to sleep while waiting for completion.
2. **Bottom Half**:
    - Executes at **interrupt time** when the hardware device finishes an operation and raises a hardware interrupt.
    - The trap handler calls `devintr()`, which identifies the interrupting device, copies the data into an in-kernel buffer, wakes up the sleeping top-half thread, and initiates the next pending I/O operation.

---

### **3. Section Breakdown**

#### **Section 6.1: Console Input**

- **UART Hardware**: xv6 talks to an emulated **16550 UART** serial-port chip.
- **Memory-Mapped Control Registers**: UART control registers are accessed via memory-mapped physical addresses starting at `0x10000000` (`UART0`).
    - **`LSR` (Line Status Register)**: Indicates whether input characters are waiting to be read.
    - **`RHR` (Receive Holding Register)**: Holds waiting input characters.
    - **`THR` (Transmit Holding Register)**: Buffer where software writes bytes to be transmitted out over the serial line.
- **Input Flow**:
    1. User types a key \(\rightarrow\) UART hardware signals a hardware interrupt.
    2. The RISC-V trap handler invokes `devintr()`.
    3. `devintr()` checks `scause` and queries the **PLIC** to identify the source as `UART0`, then calls `uartintr()`.
    4. `uartintr()` reads characters from the UART `RHR` register and calls `consoleintr()`.
    5. `consoleintr()` accumulates characters in `cons.buf`. When a newline (`\n`) arrives, it wakes up the process waiting in `consoleread()`.
    6. `consoleread()` copies the line to user space and returns from the system call.

#### **Section 6.2: Console Output**

- When a process writes to the console via `write()`, execution flows to `uartputc()`.
- `uartputc()` appends characters to an in-kernel output buffer (`uart_tx_buf`) and calls `uartstart()`.
- `uartstart()` sends the first byte to the UART `THR` register.
- As the UART finishes sending each byte, it fires a transmit-complete interrupt. `uartintr()` catches this interrupt and calls `uartstart()` to feed the next buffered character to the hardware.
- **I/O Concurrency**: Decoupling process execution from device operations via buffering allows processes to run in parallel with slow device I/O.

#### **Section 6.3: Concurrency in Drivers**

- **Spinlocks**: Drivers use locks (e.g., `cons.lock`) to protect buffer structures against three main concurrency hazards:
    1. Two processes on different CPUs calling `consoleread()` simultaneously.
    2. A hardware interrupt interrupting a CPU while it is executing inside `consoleread()`.
    3. A hardware interrupt delivered on a different CPU while `consoleread()` is running.
- **Interrupt Restrictions**: Interrupt handlers run without a dedicated process context; they cannot safely dereference user virtual memory or call functions like `copyout()` using the current process's page table.

#### **Section 6.4: Timer Interrupts**

- **Hardware Timer**: RISC-V clock hardware generates periodic timer interrupts to maintain system time and drive process scheduling.
- **Timer Registers**:
    - `time`: Hardware register holding a continuously incrementing tick count.
    - `stimecmp`: Software writes a target time into `stimecmp`. When `time` matches `stimecmp`, the CPU raises a timer interrupt.
- **Handler Flow**:
    1. Timer interrupt sets `scause` low bits to 5.
    2. `devintr()` detects this and invokes `clockintr()`.
    3. `clockintr()` increments the global `ticks` variable (on a single CPU only), wakes processes sleeping in `pause()`, and schedules the next timer interrupt by updating `stimecmp`.
    4. `devintr()` returns `2`, signaling `usertrap()` or `kerneltrap()` to call `yield()` and force a context switch.

#### **Section 6.5: Real World Comparison**

- **Programmed I/O (PIO) vs. DMA**: The UART driver reads/writes data byte-by-byte using CPU registers (Programmed I/O). High-speed devices (disks, network cards) use **Direct Memory Access (DMA)** to transfer data directly between RAM and the device.
- **Real-Time Systems**: xv6 is not a real-time OS because its scheduler ignores deadlines and kernel execution paths keep interrupts disabled for extended periods.

---

### **4. Figures, Tables, and Referenced Items in Chapter 6**

If you are summarizing or documenting this chapter, note the following regarding figures, tables, and hardware/code references:

1. **No Standalone Figures/Tables in Chapter 6**:
    
    - Unlike Chapters 1–4, **Chapter 6 contains no dedicated figure numbers or table numbers** inside its text [88–105].
2. **Cross-Referenced Figures & Hardware Maps**:
    
    - **Kernel & Physical Memory Map (Chapter 3, Figure 3.3)**: Chapter 6 references the physical memory offsets where memory-mapped hardware devices reside:
        - **PLIC (Platform-Level Interrupt Controller)**: Physical address `0x0C000000`.
        - **UART0 (Serial Port)**: Physical address `0x10000000`.
        - **VIRTIO Disk**: Physical address `0x10001000`.
3. **Key Hardware Registers & Data Structures to Reference**:
    
    - **UART 16550 Control Registers** (offsets from `0x10000000`):
        - `LSR`: Line Status Register (checks if characters are ready or device is busy).
        - `RHR`: Receive Holding Register (reads incoming characters).
        - `THR`: Transmit Holding Register (writes outgoing characters).
    - **RISC-V Control and Status Registers (CSRs)**:
        - `scause`: Identifies external device interrupts vs. timer interrupts (`scause == 5`).
        - `sstatus`: Contains the `SIE` bit used to enable/disable supervisor interrupts.
        - `stimecmp` / `time`: Controls timer interrupt scheduling.
4. **Code References (`xv6-src-booklet.pdf` Line Numbers)**:
    
    - `devintr()`: line `3506`
    - `consoleinit()`: line `7154`
    - `consoleread()`: line `7040`
    - `consoleintr()`: line `7107`
    - `uartintr()`: line `7354`
    - `uartputc()`: line `7309`
    - `clockintr()`: line `3482`
    - `start.c` timer setup: line `1102`

---
# **Chapter 7: Locking**

Here is a detailed breakdown of **Chapter 7: Locking** from the _xv6-riscv_ book.

---

### **1. Overview & Purpose of Locking**

In a multiprocessor kernel like xv6, multiple CPUs execute concurrently and share physical RAM. Concurrency arises from three sources:

1. **Multiprocessor Parallelism**: Multiple CPUs executing instructions simultaneously on shared physical memory.
2. **Thread Switching**: The kernel interleaving execution among multiple threads on a single CPU.
3. **Interrupts**: Device interrupt handlers interrupting executing kernel code and accessing the same data structures.

Without coordination, concurrent reads and writes to shared data structures yield corrupted memory or incorrect results.

- **The Lock Abstraction**: A lock provides **mutual exclusion**, ensuring that only one CPU at a time can hold the lock and enter a **critical section** to modify or read protected data.
- **Protecting Invariants**: Locks protect data structure **invariants**—properties that must hold true across operations (e.g., a linked list's head pointing to the first node). While a critical section may temporarily break an invariant during updates, mutual exclusion prevents other CPUs from observing the data in an inconsistent state.
- **Trade-Off**: Locks reduce performance because they **serialize** concurrent operations (introduce lock contention).

---

### **2. Detailed Section Breakdown**

#### **7.1 Races**

- **Race Condition**: Occurs when multiple instruction streams read/write shared memory concurrently without synchronization, causing non-deterministic results depending on exact instruction timing.
- **Example**: Two parent processes calling `wait()` on separate CPUs simultaneously. Both call `kfree()` to push freed pages onto the kernel's shared free list (`kmem.lock`).
- Without a lock, both CPUs execute `l->next = list; list = l;` at the same time. Both read the same initial `list` address, leading to a memory leak where one CPU's freed page is overwritten and lost.

#### **7.2 Code: Spinlocks (`spinlock.c` & `spinlock.h`)**

- **Structure**: Represented by `struct spinlock`, containing a `locked` word (`0` when free, `1` when held).
- **Why C Loops Fail**: A naive loop like `if (lk->locked == 0) lk->locked = 1;` fails on a multiprocessor because two CPUs can read `0` simultaneously and both enter the critical section.
- **Atomic Hardware Primitive**: xv6 uses the RISC-V atomic swap instruction (`amoswap`), exposed in C via `__sync_lock_test_and_set(&lk->locked, 1)`.
- **`acquire()` Implementation**:
    1. Disables interrupts via `push_off()`.
    2. Executes a `while` loop running `__sync_lock_test_and_set()`, atomically swapping `1` into `lk->locked` and returning the old value. It loops (spins) until the old value returned is `0`.
    3. Records the acquiring CPU (`lk->cpu`) for debugging.
- **`release()` Implementation**:
    1. Clears `lk->cpu`.
    2. Uses `__sync_lock_release(&lk->locked)` (or `amoswap`) to set `lk->locked = 0` atomically.
    3. Re-enables interrupts via `pop_off()`.

#### **7.3 Code: Using Locks & Granularity**

- **Rules for Locking**:
    1. Any variable written by one CPU while readable/writable by another must be protected by a lock.
    2. If an invariant involves multiple memory locations, all of them must be protected by the same lock.
- **Lock Granularity**:
    - **Coarse-Grained Locking**: Uses a single lock for large data structures (e.g., `kmem.lock` protecting the entire memory allocator). Simple, but causes high lock contention on multi-core systems.
    - **Fine-Grained Locking**: Uses distinct locks for smaller components (e.g., separate locks per block buffer or per inode). Increases parallelism but adds complexity and potential for deadlock.

#### **7.4 Deadlock and Lock Ordering**

- **Deadlock**: A scenario where CPU 1 holds Lock A and waits for Lock B, while CPU 2 holds Lock B and waits for Lock A. Neither can proceed, and the system freezes.
- **Deadlock Prevention Rule**: All code paths that acquire multiple locks **must acquire them in the exact same global lock order**.
- **Lock Chains in xv6**: File system operations exhibit xv6's longest lock chains. For example, file creation requires acquiring: \[\text{Directory Inode Lock} \rightarrow \text{File Inode Lock} \rightarrow \text{Block Buffer Lock} \rightarrow \text{vdisk_lock} \rightarrow \text{proc's } \text{p->lock}\].
- **Non-Reentrant Locks**: xv6 forbids recursive/re-entrant locking (a CPU attempting to re-acquire a spinlock it already holds). `acquire()` calls `holding()` and panics if recursive locking is detected.

#### **7.5 Locks and Interrupts**

- **The Interrupt Hazard**: If a thread holds a spinlock (e.g., `sys_pause` holding `tickslock`) and a timer interrupt occurs on the _same CPU_, the interrupt handler (`clockintr`) will attempt to acquire `tickslock`. Because the thread cannot run until the interrupt handler returns, the CPU deadlocks itself.
- **The Rule**: A CPU must **never hold a spinlock with interrupts enabled** on that local CPU.
- **Nesting Control**: `push_off()` disables interrupts and increments a per-CPU nesting counter (`c->noff`). `pop_off()` decrements the counter and restores the original interrupt state (`sstatus.SIE`) only when the count reaches zero. `push_off()` is called _strictly before_ setting `lk->locked = 1`.

#### **7.6 Instruction and Memory Ordering**

- **Reordering Hazards**: Compilers and out-of-order CPUs may reorder memory loads and stores to optimize execution, breaking concurrent assumptions.
- **Memory Barrier**: xv6 issues `__sync_synchronize()`, which emits a RISC-V hardware memory fence instruction. This forces the compiler and CPU not to reorder loads or stores across the barrier during `acquire()` and `release()`.

#### **7.7 Sleep Locks (`sleeplock.c` & `sleeplock.h`)**

- **Limitations of Spinlocks**: Spinlocks waste CPU cycles if held for lengthy operations (e.g., disk I/O taking tens of milliseconds) and cannot yield the CPU while held.
- **Sleep-Lock Abstraction**:
    - Represented by `struct sleeplock`, containing a `locked` flag, a name, and an underlying spinlock `lk`.
    - `acquiresleep()` yields the CPU (`sleep()`) while waiting, releasing the internal spinlock atomically so other processes can run.
    - Leaves interrupts enabled, permitting disk I/O and process context switching while the sleep-lock is held.
    - **Restrictions**: Sleep-locks **cannot** be used inside interrupt handlers (which cannot sleep), nor can spinlocks be acquired while holding a sleep-lock if it leads to yield-related deadlocks.

---

### **3. Figures and Tables in Chapter 7**

Here are the specific figures and tables in Chapter 7:

1. **Figure 7.1: Simplified SMP Architecture**
    
    - **Location**: Page 65 of the xv6 text.
    - **Description**: Diagrams two CPUs connected via a shared memory bus attempting to execute `l->next = list` and `list = l` concurrently. It illustrates how race conditions occur on a shared linked list when two CPUs update shared memory without mutual exclusion.
2. **Figure 7.3: Locks in xv6 (Summary Table)**
    
    - **Location**: Page 68 of the xv6 text.
    - **Description**: A summary table listing all 15 spinlocks and sleep-locks in the xv6 kernel and what each protects:
        - `bcache.lock`: Protects allocation of block buffer cache entries.
        - `cons.lock`: Serializes read processing of console input.
        - `tx_lock`: Serializes access to console (UART) output hardware.
        - `ftable.lock`: Serializes allocation of a `struct file` in the global file table.
        - `itable.lock`: Protects allocation of in-memory inode entries.
        - `vdisk_lock`: Serializes access to disk hardware and queue of DMA descriptors.
        - `kmem.lock`: Serializes allocation of physical memory pages.
        - `log.lock`: Serializes operations on the file system transaction log.
        - `pipe's pi->lock`: Serializes operations on each pipe.
        - `pid_lock`: Serializes increments of `next_pid`.
        - `proc's p->lock`: Serializes changes to a process's state.
        - `wait_lock`: Helps `wait()` avoid lost wakeups.
        - `tickslock`: Serializes operations on the `ticks` counter.
        - `inode's ip->lock`: Serializes operations on each inode and its content.
        - `buf's b->lock`: Serializes operations on each block buffer.

---
----











 %%
# Chapter 8: Scheduling
 
Here is a detailed breakdown of **Chapter 8: Scheduling** from the _xv6-riscv_ book.

---

### **1. Overview & Purpose of Scheduling**

In any operating system, the number of active processes typically exceeds the number of physical CPU cores. The kernel must time-share (multiplex) physical CPUs among processes in a way that is transparent to user applications, giving each process the illusion of having its own dedicated virtual CPU.

---

### **2. Detailed Section Breakdown**

#### **8.1 Multiplexing**

- **Switching Conditions**: xv6 multiplexes CPUs by switching from one process to another under two conditions:
    1. **Voluntary Switches**: A process makes a blocking system call (e.g., waiting for I/O via `read()`, or waiting for a child process via `wait()`).
    2. **Involuntary Switches**: The kernel forces a switch via periodic timer interrupts to prevent compute-bound processes from monopolizing the CPU.
- **Key Challenges of Multiplexing**:
    - **Register Preservation**: Saving and restoring CPU register states safely when transitioning between threads.
    - **Transparency**: Using hardware timer interrupts to drive context switches without user programs noticing.
    - **Mutual Exclusion & Locking**: Preventing two CPUs from picking and running the exact same process simultaneously.
    - **Resource Reclamation**: Safely freeing exiting process memory without running code on a stack that is being deallocated.
    - **CPU Identification**: Ensuring each CPU on a multi-core system accurately tracks which process it is currently executing (`mycpu()` and `myproc()`).

---

#### **8.2 Context Switch Overview**

- **Definition**: A **context switch** is the sequence of steps involved in a CPU stopping execution of one kernel thread and resuming execution of another kernel thread.
- **Indirect Thread Switching**: xv6 does **not** context-switch directly from process kernel thread A to process kernel thread B. Instead:
    1. Process A's kernel thread context-switches to the current CPU's dedicated **scheduler thread**.
    2. The CPU's scheduler thread selects a `RUNNABLE` process (Process B).
    3. The scheduler thread context-switches to Process B's kernel thread.
- **Full Process-to-Process Execution Path**: \[\text{User Process A} \xrightarrow{\text{Trap/ecall}} \text{Kernel Thread A} \xrightarrow{\text{swtch()}} \text{CPU Scheduler Thread} \xrightarrow{\text{swtch()}} \text{Kernel Thread B} \xrightarrow{\text{userret/sret}} \text{User Process B}\]

---

#### **8.3 Code: Context Switching (`swtch.S`)**

- **The `swtch()` Function**: Written in RISC-V assembly (`kernel/swtch.S`), `swtch(struct context *old, struct context *new)` saves the current CPU registers into `old` and loads the previously saved registers from `new`.
- **Callee-Saved Registers**: `swtch()` only explicitly saves and restores RISC-V **callee-saved registers**: `ra` (return address), `sp` (stack pointer), and `s0`–`s11` (saved registers).
- **Caller-Saved Registers**: General-purpose caller-saved registers are automatically saved on the stack by the C compiler prior to invoking `swtch()`, or saved in the process's `trapframe` during the initial trap into the kernel.
- **Stack Switching**: Overwriting the CPU's `sp` register with `new->sp` switches execution from one stack to another instantly. When `swtch()` executes the `ret` instruction, execution resumes at the address stored in `new->ra`.

---

#### **8.4 Code: Scheduling (`proc.c`)**

- **Per-CPU Scheduler Thread**: Each CPU runs a separate scheduler thread executing `scheduler()` in `kernel/proc.c`. Having a separate scheduler thread per CPU allows cores to look for runnable processes concurrently without sharing execution stacks.
- **`scheduler()` Loop**:
    1. Iterates through the global `proc` table looking for a process with `p->state == RUNNABLE`.
    2. Acquires `p->lock` to protect state transitions.
    3. Sets process state to `RUNNING` and assigns `c->proc = p`.
    4. Calls `swtch(&c->context, &p->context)` to run the process thread.
    5. When the process yields and switches back, `scheduler()` clears `c->proc` and releases `p->lock`.
- **`sched()` and `yield()`**:
    - A running kernel thread calls `yield()`, `sleep()`, or `kexit()`, which invokes `sched()`.
    - `sched()` checks preconditions (holds `p->lock`, interrupts disabled nesting count `noff == 1`, process state is not `RUNNING`), then executes `swtch(&p->context, &c->context)` to return to the scheduler thread.
- **Hand-Off Locking Pattern (`p->lock`)**:
    - xv6 holds `p->lock` **across calls to `swtch()`**.
    - The thread calling `swtch()` acquires `p->lock`, but the target thread releases it after `swtch()` returns on the destination stack.
    - **Why This Is Necessary**: `p->state` and `p->context` must be updated atomically across CPUs. If `p->lock` were released before calling `swtch()`, another CPU could see `p->state == RUNNABLE` and attempt to run the process before its registers and stack are fully saved by the first CPU.

---

#### **8.5 Code: `mycpu` and `myproc`**

- **Hardware Thread Pointer (`tp`)**: RISC-V assigns each CPU a unique `hartid`. During boot (`start.c`), xv6 writes the CPU's `hartid` into the `tp` register.
- **`mycpu()`**: Reads the `tp` register and uses it to index the global `cpus[]` array, returning a pointer to the current CPU's `struct cpu`.
- **`myproc()`**: Disables interrupts briefly (`push_off()`), calls `mycpu()`, extracts `c->proc`, and re-enables interrupts (`pop_off()`). Disabling interrupts prevents a timer interrupt from context-switching the process to a different CPU midway through reading `c->proc`.
- **Preserving `tp`**: User code can freely overwrite registers. Therefore, `uservec` saves `tp` into the trampoline page when entering the kernel, and `userret` restores `tp` before returning to user space.

---

#### **8.6 Real World Comparison**

- **xv6 Scheduling Policy**: Simple **Round Robin** (scans the process table sequentially and picks the next available `RUNNABLE` process).
- **Real-World Schedulers**: Modern kernels employ priority-based scheduling algorithms (e.g., Linux's Completely Fair Scheduler / CFS, or Multi-Level Feedback Queues / MLFQ) to balance latency, interactivity, fairness, and throughput across complex workloads.

---

### **3. Figures and Tables to Reference in Chapter 8**

1. **Figure 8.1: Switching from one user process to another**
    
    - **Location**: Page 76 of the xv6 text.
    - **Description**: A diagram illustrating the complete context-switch flow between two user processes (`shell` and `cat`) on a single CPU system:
        - User Space (`shell`) \(\rightarrow\) Trap to Kernel Stack (`kstack shell`).
        - `swtch()` from `kstack shell` \(\rightarrow\) Scheduler Stack (`kstack scheduler`).
        - `swtch()` from `kstack scheduler` \(\rightarrow\) Target Kernel Stack (`kstack cat`).
        - Trap return from `kstack cat` \(\rightarrow\) User Space (`cat`).
2. **Figure 8.2: Execution pattern of `swtch` / scheduling loop**
    
    - **Location**: Page 78 of the xv6 text.
    - **Description**: Illustrates the lock hand-off pattern between a yielding process thread and the CPU scheduler thread, showing how `p->lock` remains held across the `swtch()` call boundary until execution safely transitions to the new stack.

---

💡 **Next Step**: Would you like to move on to **Chapter 9: Sleep and Wakeup** to examine sequence coordination, wait channels, and avoiding lost wakeups?

---

# Chapter 9: Sleep and Wakeup
### **1. Overview & Purpose of Sleep and Wakeup (Section 9.1)**

While scheduling and locking conceal the internal actions of threads from one another, operating systems also require abstractions that allow threads to **intentionally coordinate and wait** for specific conditions to become true (often referred to as _sequence coordination_ or _conditional synchronization_). Examples include:

- A pipe reader waiting for a writer to produce data.
- A parent process waiting in `wait()` for a child process to exit.
- A process waiting for disk hardware to complete an I/O operation.

#### **The Interface**

```
void sleep(void *chan, struct spinlock *lk);
void wakeup(void *chan);
```

- **Wait Channel (`chan`)**: An opaque 64-bit identifier (typically the memory address of the shared data structure, e.g., `&pipe`). `sleep` marks the calling process as `SLEEPING` and context-switches to the CPU scheduler thread.
- **`wakeup(chan)`**: Scans the process table and marks all processes sleeping on matching `chan` as `RUNNABLE`.

#### **The Lost Wake-Up Problem & Condition Locks**

Because the condition being waited on involves shared data (e.g., checking if `pipe->buffer` is empty), it must be protected by a lock (the **condition lock**, e.g., `pipe->lock`).

- **The Hazard**: If a thread checks the condition, sees that it is false, and is interrupted _before_ calling `sleep()`, another thread on a second CPU could update the condition and call `wakeup()`. Since no thread is sleeping yet, the wakeup is missed ("lost"). When the first thread resumes, it calls `sleep()` and may wait forever.
- **The Solution**: `sleep()` requires the caller to hold the condition lock and pass it as an argument. `sleep()` atomically acquires the process's own `p->lock`, releases the condition lock, sets `p->state = SLEEPING`, and yields the CPU via `sched()`.

---

### **2. Detailed Section Breakdown**

#### **Section 9.2: Code: Sleep and Wakeup (`kernel/proc.c`)**

- **Atomic Lock Hand-Off**:
    1. `sleep()` acquires `p->lock` before releasing condition lock `lk`. Holding `p->lock` prevents a concurrent `wakeup()` (which must also acquire `p->lock`) from missing the process state transition.
    2. `sleep()` sets `p->chan = chan` and `p->state = SLEEPING`, then calls `sched()`.
    3. The CPU scheduler thread releases `p->lock` once on its own stack.
- **Spurious Wakeups & Loop Requirement**:
    - `wakeup(chan)` wakes **all** processes sleeping on `chan`.
    - If multiple processes are sleeping on the same channel (e.g., two readers on a pipe), all are marked `RUNNABLE`. The first process to run acquires the condition lock and reads the data. When subsequent processes run, they find the buffer empty again.
    - **Rule**: `sleep()` must **always** be called inside a `while` loop that re-tests the condition after waking up.

---

#### **Section 9.3: Code: Pipes (`kernel/pipe.c`)**

- Demonstrates producer/consumer synchronization in `pipewrite()` and `piperead()`.
- **Separate Wait Channels**:
    - `&pi->nread`: Writers sleep here when the buffer is full (`pi->nwrite == pi->nread + PIPESIZE`).
    - `&pi->nwrite`: Readers sleep here when the buffer is empty (`pi->nread == pi->nwrite`).
- **Flow**:
    - `pipewrite()` adds data; if full, it calls `wakeup(&pi->nwrite)` to alert readers and sleeps on `&pi->nread`.
    - `piperead()` reads data; calls `wakeup(&pi->nread)` to alert writers and returns.

---

#### **Section 9.4: Code: Wait, Exit, and Kill (`kernel/proc.c`)**

- **`kwait()`**:
    - Serves as the kernel implementation of `wait()`.
    - Acquires global `wait_lock` (the condition lock for parent/child state changes).
    - Scans `proc[]` table for child processes (`p->parent == myproc()`).
    - If a child is in `ZOMBIE` state: reclaims its resources and `proc` structure, copies its exit status (`xstate`), and returns its PID.
    - If children exist but none are zombies: calls `sleep(myproc(), &wait_lock)` to sleep until a child exits.
- **`kexit()`**:
    - Acquires `wait_lock` and `p->lock` (lock ordering: `wait_lock` first, then `p->lock`).
    - Reparents any child processes to the `init` process (PID 1).
    - Calls `wakeup(p->parent)` to wake a parent waiting in `kwait()`.
    - Sets state to `ZOMBIE` and calls `sched()`.
- **`kkill()`**:
    - Sets victim's `p->killed = 1`.
    - If the victim is `SLEEPING`, `kkill()` calls `wakeup()` to force it into `RUNNABLE` state so it can process `p->killed` and terminate.
    - **Interruptible vs. Uninterruptible Sleep**:
        - System calls that can be safely abandoned check `p->killed` in their `while` sleep loops (e.g., pipe reads).
        - Multi-step operations (e.g., `virtio` disk driver) do **not** check `p->killed` inside the sleep loop to prevent file system corruption, deferring termination until trap return.

---

#### **Section 9.5: Process Locking (`p->lock`)**

The per-process lock `p->lock` is the most complex lock in xv6. It protects:

1. Direct fields: `p->state`, `p->chan`, `p->killed`, `p->xstate`, and `p->pid`.
2. Allocation of `proc[]` array slots in `allocproc()`.
3. Process visibility during creation and destruction.
4. Race conditions between a parent's `kwait()` and a child's `kexit()` transition to `ZOMBIE`.
5. Context-switching and scheduler state hand-offs across CPUs.

_(Note: `p->parent` is protected by `wait_lock`, not `p->lock`.)_

---

#### **Section 9.6: Real World Comparison**

- **Lost Wakeup Prevention**:
    - Original Unix: Disabled interrupts (uniprocessor only).
    - FreeBSD: `msleep` takes an explicit lock argument (like xv6).
    - Linux: Uses per-condition wait queues (`wait_queue_head_t`) with internal locks instead of scanning all processes on a global channel.
- **Signal vs. Broadcast**:
    - xv6's `wakeup()` is a **broadcast** (wakes all waiters).
    - Modern condition variables provide `signal` (wakes 1 waiter) and `broadcast` (wakes all) to avoid _thundering herd_ performance penalties.

---

### **3. Figures and Tables in Chapter 9**

If you are documenting or referencing Chapter 9, note the following figures, tables, and code locations:

1. **Figure 9.1: Overlapping locks to avoid lost wake-up**
    
    - **Location**: Page 83 of the xv6 text (Section 9.2).
    - **Description**: A diagram tracing the overlapping lock acquisition sequence between `piperead()` holding `pipe->lock`, calling `sleep()`, acquiring `p->lock`, releasing `pipe->lock`, setting `p->state = SLEEPING`, and entering the scheduler thread (which finally releases `p->lock`).
    - **Note**: This is the **only figure or table** in Chapter 9.
2. **Key Code References (`xv6-riscv` source files)**:
    
    - `sleep()` & `wakeup()`: `kernel/proc.c`
    - `piperead()` & `pipewrite()`: `kernel/pipe.c`
    - `kwait()`, `kexit()`, `kkill()`: `kernel/proc.c`
    - Condition Locks: `wait_lock` (global parent/child lock) and `p->lock` (per-process lock).

---

# Chapter 10: File System

Here is a detailed breakdown of **Chapter 10: File System** from the _xv6-riscv_ book.

---

### **1. Overview & Core Challenges**

The primary purpose of a file system is to organize and store data persistently across reboots while supporting sharing among users and applications. The xv6 file system provides Unix-like files, directories, and pathnames, storing its data on a `virtio` disk.

To achieve this, the file system addresses four fundamental challenges:

1. **On-Disk Data Structures**: Representing directory trees, mapping files to content blocks, and tracking free/allocated blocks.
2. **Crash Recovery**: Ensuring file system consistency if power fails midway through a multi-block update.
3. **Concurrency Control**: Coordinating concurrent process accesses to maintain invariants.
4. **Performance & Caching**: Maintaining an in-memory cache of popular blocks because disk accesses are orders of magnitude slower than main memory.

---

### **2. The 7-Layer File System Architecture**

The xv6 file system is organized into seven distinct layers:

1. **Disk Layer**: Reads and writes 512-byte sectors/blocks on a `virtio` hard drive.
2. **Buffer Cache Layer**: Caches disk blocks in RAM and synchronizes concurrent access using per-buffer sleep-locks.
3. **Logging Layer**: Wraps multi-block updates into atomic transactions to ensure crash consistency.
4. **Inode Layer**: Provides individual files, each identified by a unique i-number and containing metadata and data block pointers.
5. **Directory Layer**: Implements directories as special inodes containing sequences of directory entries (`struct dirent`).
6. **Pathname Layer**: Provides hierarchical pathnames (e.g., `/usr/rtm/xv6/fs.c`) and resolves them recursively.
7. **File Descriptor Layer**: Abstracts files, pipes, and devices behind a uniform Unix file descriptor interface.

---

### **3. Detailed Section Breakdown**

#### **3.1 On-Disk Layout & Block Allocator**

- **Disk Partitioning**: The disk is divided into 1024-byte blocks:
    - **Block 0**: Unused (holds the boot sector).
    - **Block 1**: **Superblock** — Contains metadata about file system size, number of data blocks, inodes, and log blocks.
    - **Blocks 2..**: **Log blocks**.
    - **Next Blocks**: **Inode area** (multiple on-disk inodes per block).
    - **Next Blocks**: **Bitmap blocks** (tracking free vs. allocated data blocks).
    - **Remaining Blocks**: **Data blocks**.
- **Block Allocator (`balloc` / `bfree`)**:
    - Tracks free blocks using a free bitmap (0 = free, 1 = in use).
    - `balloc()` scans bitmap blocks looking for a zero bit, sets it to 1, and returns the allocated block number.
    - `bfree()` locates the corresponding bitmap block and clears the bit.
    - Race conditions are prevented because the buffer cache guarantees exclusive access to any single bitmap block.

---

#### **3.2 Buffer Cache Layer (`kernel/bio.c`)**

- **Two Responsibilities**:
    1. Synchronize block accesses so only one copy of a disk block exists in memory and only one thread modifies it at a time.
    2. Cache popular blocks to avoid slow disk re-reads.
- **Data Structure**: A doubly-linked list of `struct buf` entries initialized during `binit()`.
- **Operations**:
    - **`bread(dev, sector)`**: Calls `bget()` to acquire a buffer, reading the data from disk via `virtio_disk_rw()` if not already valid (`b->valid == 0`).
    - **`bwrite(b)`**: Writes the modified buffer out to disk.
    - **`brelse(b)`**: Releases the buffer's sleep-lock and moves it to the head of the doubly-linked list so the **Least Recently Used (LRU)** buffer stays at the tail for recycling.
- **Synchronization**: `bcache.lock` protects cache lookups/allocations, while each `struct buf` contains a per-buffer sleep-lock ensuring exclusive thread access.

---

#### **3.3 Logging Layer & Crash Recovery (`kernel/log.c`)**

- **Goal**: Guarantees atomic updates across crashes (all logged writes appear on disk after recovery, or none do).
- **Log Structure**: Fixed location on disk containing a **header block** followed by logged block slots. The header stores a count of logged blocks and an array of destination sector numbers.
- **System Call Usage Pattern**:
    
    ```
    begin_op();
    ...
    bp = bread(...);
    bp->data[...] = ...;
    log_write(bp);
    ...
    end_op();
    ```
    
- **Transaction Lifecycle**:
    1. **`begin_op()`**: Reserves log space (`log.outstanding`) and blocks if a commit is currently running.
    2. **`log_write(bp)`**: Records the block’s destination sector, pins the buffer in memory, and absorbs repeated writes to the same block within a transaction (**absorption**).
    3. **`end_op()`**: Decrements `log.outstanding`. When the last active system call finishes, it calls `commit()`.
    4. **Commit Sequence**:
        - Writes modified blocks from RAM to the disk log (`write_log()`).
        - Writes the log header block to disk (**the commit point**).
        - Installs writes from the log into their true disk locations (`install_trans()`).
        - Writes the log header with a count of 0 to unreserve the log.
- **Recovery (`recover_from_log`)**: Called during boot (`fsinit`) before any user process runs. If the header count > 0, it replays all logged writes to their target disk locations and clears the log.

---

#### **3.4 Inode Layer (`kernel/fs.c`)**

- **Dual Meaning**:
    - **On-Disk Inode (`struct dinode`)**: Contains file type, link count (`nlink`), size, and block pointers.
    - **In-Memory Inode (`struct inode`)**: Contains a copy of `struct dinode` plus kernel state (`ref` pointer count, sleep-lock `lock`, device number, inode number).
- **Block Pointers & Multi-Level Index** (Figure 10.3):
    - `addrs[0..11]`: **12 Direct Blocks** (12 × 1024 bytes = 12 KB).
    - `addrs`: **1 Indirect Block Pointer** pointing to a disk block containing 256 data block addresses (256 × 1024 bytes = 256 KB).
    - **Max File Size**: \(12 + 256 = 268\) blocks (268 KB).
- **Mapping (`bmap`)**: Maps a logical file block number `bn` to an actual disk block sector, allocating new blocks via `balloc()` if needed.
- **Inode Operations**:
    - `ialloc()`: Allocates a new on-disk inode.
    - `iget()`: Fetches/returns an in-memory inode entry without locking it, incrementing `ip->ref`.
    - `ilock()` / `iunlock()`: Acquires/releases the inode's sleep-lock and reads data from disk if necessary.
    - `iput()`: Decrements `ip->ref`. If `ref == 0` and `nlink == 0`, it frees the file's data blocks (`itrunc`) and frees the inode.

---

#### **3.5 Directory, Pathname, & File Descriptor Layers**

- **Directory Layer**:
    - Directories are files with type `T_DIR` containing directory entries (`struct dirent`).
    - Each `struct dirent` holds an inode number and a filename up to `DIRSIZ` (14) characters.
    - `dirlookup()` searches a directory for a name, returning a pointer to the matching locked/unlocked inode.
    - `dirlink()` writes a new `struct dirent` entry into a directory.
- **Pathname Layer (`namei` / `nameiparent`)**:
    - Parses pathnames recursively starting from the root directory `/` (if path starts with `/`) or the process's current working directory `cwd`.
    - Uses `dirlookup()` to evaluate each component.
- **File Descriptor Layer (`file.c` & `sysfile.c`)**:
    - Abstracts Unix resources via `struct file` (wrapping an inode or pipe along with an offset).
    - `ftable`: Global table of open files managed via `filealloc()`, `filedup()`, and `fileclose()`.
    - System calls like `sys_link()` and `sys_unlink()` manipulate inode `nlink` counts and directory entries inside transaction blocks.

---

### **4. Figures and Tables to Reference in Chapter 10**

Here are the specific figures and structural references in Chapter 10:

1. **Figure 10.1: Layers of the xv6 file system**
    
    - **Location**: Page 90 of the xv6 text.
    - **Description**: A stacked architectural diagram showing the 7 abstraction layers of the file system: \[\text{File Descriptor} \rightarrow \text{Pathname} \rightarrow \text{Directory} \rightarrow \text{Inode} \rightarrow \text{Logging} \rightarrow \text{Buffer Cache} \rightarrow \text{Disk}\]
2. **Figure 10.2: Structure of the xv6 file system**
    
    - **Location**: Page 90 of the xv6 text.
    - **Description**: A disk block layout diagram illustrating how physical disk blocks are partitioned:
        - Block 0: `boot`
        - Block 1: `super` (Superblock)
        - Blocks 2..: `log`
        - Following: `inodes`
        - Following: `bit map`
        - Remaining: `data` blocks
3. **Figure 10.3: The representation of a file on disk**
    
    - **Location**: Page 98 of the xv6 text.
    - **Description**: Diagrams the `struct dinode` block pointer hierarchy:
        - Header fields: `type`, `major`, `minor`, `nlink`, `size`.
        - `address 1` through `address 12`: Point directly to data blocks.
        - `indirect`: Points to an indirect block containing 256 block addresses (`address 1` .. `address 256`), which point to data blocks.

---

💡 **Next Step**: Would you like to proceed to **Chapter 11: Concurrency Revisited** to examine complex locking patterns, lock-free idioms, and deadlock avoidance techniques in detail?


%%

