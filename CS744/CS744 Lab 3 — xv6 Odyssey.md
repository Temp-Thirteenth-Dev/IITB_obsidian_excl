# CS744 Lab 3 — xv6 Odyssey
## Final Lab-Exam Revision Notes

---

# 0. Big Picture

This lab has three layers:

```text
TASK 1 → USER PROGRAMS
    ↓
learn fork / exec / files / processes

TASK 2 → SYSTEM CALLS
    ↓
learn user → kernel syscall path
modify struct proc / file handling

TASK 3 → VIRTUAL MEMORY
    ↓
learn page tables
VA → PA translation
PTE flags
address-space size
```

The xv6 source tree is roughly:

```text
user/
    user programs + syscall declarations/wrappers

kernel/
    proc.c       → process lifecycle
    proc.h       → struct proc / PCB
    sysproc.c    → process-related syscall handlers
    sysfile.c    → file-related syscall handlers
    syscall.c    → syscall dispatch
    syscall.h    → syscall numbers
    vm.c         → page tables / virtual memory
    file.c       → open-file operations
    file.h       → struct file
    defs.h       → kernel function declarations
```

---

# 1. USER PROGRAMS — TASK 1

## 1A — `hello`

Goal:

```text
$ hello
Hello World!
```

File:

```text
user/hello.c
```

Typical structure:

```c
#include "kernel/types.h"
#include "user/user.h"

int
main(void)
{
    printf("Hello World!\n");
    exit(0);
}
```

### Important idea

User programs must be added to:

```make
UPROGS=\
    ...
    $U/_hello\
```

because xv6 builds user programs on the host and places them into the filesystem image.

The lab explicitly describes this static-build model.

---

# 1B — `clear`, `head`, `tail`

## `clear`

User-space program that clears the terminal.

Concept:

```text
escape sequence
      ↓
clear terminal
      ↓
move cursor to top-left
```

Typical:

```c
printf("\033[2J\033[H");
```

Expected:

```text
$ hello
Hello World!
$ clear
$
```

The lab requires `clear` as a user-space program.

---

## `head`

Usage:

```text
head <file> <N>
```

Print first N lines.

Core idea:

```text
open()
  ↓
read()
  ↓
count '\n'
  ↓
stop after N newlines
```

Important file APIs:

```c
open()
read()
close()
```

The lab specifically recommends looking at `cat.c`.

---

## `tail`

Usage:

```text
tail <file> <N>
```

Print last N lines.

Simple strategy:

```text
read whole file
     ↓
count newlines
     ↓
find beginning of last N lines
     ↓
print from there
```

### Viva

`head` and `tail` are **ordinary user processes**, not kernel commands.

---

# 1C — `cmd`

Usage:

```text
cmd <program> [args...]
```

Example:

```text
$ cmd echo hello world
hello world
```

### Process model

```text
                 cmd
                  |
                fork()
             /         \
            /           \
       parent           child
         |                |
      wait()            exec()
                          |
                          ↓
                     target program
```

The lab requires `fork()`, `exec()`, argument passing, and parent `wait()`.

### Critical distinction

```text
fork() → creates a new process

exec() → replaces the current process's program

wait() → parent waits for child
```

`exec()` does **not** create a process.

### Argument trick

For:

```text
cmd echo hello world
```

the child receives:

```text
argv[1] = "echo"
argv[2] = "hello"
argv[3] = "world"
```

Hence:

```c
exec(argv[1], &argv[1]);
```

The new program sees:

```text
argv[0] = "echo"
argv[1] = "hello"
argv[2] = "world"
```

---

# 1D — `cp`

Usage:

```text
cp source destination
```

Goal:

```text
source
  ↓
 read
  ↓
 buffer
  ↓
 write
  ↓
destination
```

Core loop:

```c
while ((n = read(src, buf, sizeof(buf))) > 0)
    write(dst, buf, n);
```

### `read()` return values

```text
> 0 → number of bytes read
  0 → EOF
< 0 → error
```

### Why `write(..., n)`?

Because the final `read()` may return fewer bytes than the buffer size.

Use:

```c
write(dst, buf, n);
```

not:

```c
write(dst, buf, sizeof(buf));
```

The lab explicitly requires `open`, `read`, `write`, and `close`.

---

# 1E — `mgrep`

Usage:

```text
mgrep <pattern> <file1> <file2> ... <fileN>
```

For **each file**, create **exactly one child**.

```text
                 parent
          /        |        \
         /         |         \
     worker 1   worker 2   worker 3
      file1       file2      file3
```

Each worker:

```text
open assigned file
      ↓
search pattern
      ↓
print matching lines
```

Output format:

```text
(Worker PID: 8) matching line...
```

Parent:

```text
fork all workers
     ↓
wait for workers
     ↓
exit
```

The lab requires one child per input file, independent searching, PID-tagged output, error handling, and waiting for workers.

### Interleaved output — VERY IMPORTANT

If all children run concurrently:

```text
child A → print
child B → print
child A → print
child C → print
```

their outputs can become interleaved.

Why doesn't:

```c
wait();
```

after creating all children fix it?

Because the children are **already running concurrently**.

The lab accepts a simple sequential solution:

```text
fork child1
wait child1

fork child2
wait child2

fork child3
wait child3
```

Alternatively, synchronization can be used while retaining concurrency.

---

# 2. SYSTEM CALLS — THE MOST IMPORTANT SECTION

## The syscall pipeline

Memorize this:

```text
USER PROGRAM
    |
    | getppid()
    ↓
user/user.h
    |
    ↓
user/usys.pl
    |
    | generated assembly
    | put syscall number in a register
    | ecall
    ↓
KERNEL
    |
    ↓
kernel/syscall.c
    |
    | dispatch using syscall number
    ↓
sys_*()
    |
    ↓
kernel operation
```

The lab describes these files and their roles explicitly.

---

## What each syscall file does

### `user/user.h`

Declarations visible to user programs:

```c
int getppid(void);
int square(int);
```

### `user/usys.pl`

Adds wrappers:

```perl
entry("getppid");
entry("square");
```

It generates `usys.S`; the wrapper places the syscall number in a register and executes `ecall`.

### `kernel/syscall.h`

Assigns unique syscall numbers:

```c
#define SYS_getppid ...
```

### `kernel/syscall.c`

Dispatch table:

```c
[SYS_getppid] sys_getppid,
```

and central dispatcher:

```c
syscall()
```

### `kernel/sysproc.c`

Actual process-related syscall handlers:

```c
sys_getppid()
sys_square()
...
```

### `kernel/proc.h`

Contains:

```c
struct proc
```

the Process Control Block.

### `kernel/proc.c`

Main process-management code:

```text
allocproc
fork
kfork
wait
exit
scheduler
...
```

### `kernel/defs.h`

Forward declarations for kernel functions shared across source files.

---

# 2A — `getppid()`

Interface:

```c
int getppid(void);
```

Goal:

```text
child → parent's PID
```

Kernel logic:

```c
struct proc *p = myproc();

return p->parent->pid;
```

Concept:

```text
myproc()
   ↓
current process
   ↓
p->parent
   ↓
parent process
   ↓
parent->pid
```

The lab specifically suggests using `getpid()` as the model.

### Viva

`myproc()` gives the process currently executing on the CPU.

---

# 2B — `square()`

Interface:

```c
int square(int num);
```

Kernel:

```c
uint64
sys_square(void)
{
    int x;

    if(argint(0, &x) < 0)
        return -1;

    return x * x;
}
```

### `argint()`

```c
argint(0, &x);
```

means:

> retrieve syscall argument #0 as an integer.

The lab explicitly points to `argint()` for this task.

---

# `argint`, `argaddr`, `argfd`

### `argint`

Use for integer arguments:

```c
argint(0, &n);
```

### `argaddr`

Use for pointer/address arguments:

```c
argaddr(1, &addr);
```

### `argfd`

Use for a file descriptor and corresponding `struct file`:

```c
argfd(0, 0, &f);
```

---

# 2C — `child_count`

Add to:

```c
struct proc
```

```c
int child_count;
```

### Meaning

Only **immediate children** count.

```text
          A
        /   \
       B     C
      /
     D
```

For A:

```text
child_count = 2
```

For B:

```text
child_count = 1
```

D is not A's immediate child.

### Initialize

New process:

```text
child_count = 0
```

### Increment

When a child is successfully created and associated with its parent:

```text
fork
 ↓
parent.child_count++
```

### Decrement

**Not when child exits.**

Instead:

```text
child exits
   ↓
 ZOMBIE
   ↓
wait()
   ↓
reaped
   ↓
parent.child_count--
```

The assignment explicitly emphasizes this distinction.

### Locking

Concurrent reaping can modify the same parent's counter.

Therefore protect the update using the appropriate process/wait locking. The assignment specifically warns about race conditions here.

---

## Syscalls

```c
int get_child_count(void);
int get_process_child_count(int pid);
```

Current process:

```c
return myproc()->child_count;
```

Other process:

```text
PID
 ↓
search proc[]
 ↓
matching struct proc
 ↓
child_count
```

Return:

```text
-1 → process doesn't exist
```

---

# 2D — `nfork()`

Interface:

```c
int nfork(int n, int *child_pids);
```

Required behavior:

```text
parent → n
children → 0
```

and:

```text
child_pids = [pid1, pid2, ...]
```

The assignment requires `argint`, `argaddr`, `kfork`, and `copyout`, and says not to modify `kfork()`.

### Important concepts

```c
argint(0, &n);
```

gets number of children.

```c
argaddr(1, &addr);
```

gets the userspace pointer.

`kfork()` creates the process.

`copyout()` copies kernel data to user memory.

### Why `copyout()`?

This is unsafe:

```c
child_pids[i] = pid;
```

if `child_pids` is a user pointer being accessed from kernel code.

Instead:

```text
kernel array
     |
 copyout()
     ↓
user memory
```

The lab explains that `copyout()` safely copies kernel memory to userspace.

### Child shouldn't continue parent's loop

This is one of the trickiest requirements.

The parent's logic is:

```text
create child1
create child2
create child3
```

Children must not start creating more children from inside the syscall.

Your supplied `kfork()` already sets:

```c
np->trapframe->a0 = 0;
```

so the child returns from the fork-like mechanism with a syscall result of `0`.

---

# 2E — Syscall counters

Add:

```c
int syscall_counts[...];
```

to `struct proc`.

Each process gets its own array.

Concept:

```text
PID 10:
  syscall 1 → 3
  syscall 7 → 1
  syscall 13 → 4

PID 11:
  syscall 1 → 1
  syscall 13 → 2
```

### Where to increment?

In:

```c
syscall()
```

inside:

```text
kernel/syscall.c
```

because **every syscall passes through the dispatcher**.

Concept:

```c
num = p->trapframe->a7;

p->syscall_counts[num]++;

p->trapframe->a0 = syscalls[num]();
```

### Why `a7`?

On RISC-V, the syscall number is placed in register:

```text
a7
```

The lab explicitly identifies `syscall()` as the function to augment.

### Syscalls

```c
int print_syscalls(void);
int print_process_syscalls(int pid);
```

`print_syscalls()` → current process.

`print_process_syscalls(pid)` → find that process and print its array.

---

# 2F — File descriptors, inode, offset

Syscalls:

```c
uint64 get_inode_num(int fd);
uint64 get_read_offset(int fd);
```

The lab requires `-1` for invalid FD, wrong FD type, or unreadable file.

## Most important data structure

```c
struct proc
{
    ...
    struct file *ofile[NOFILE];
}
```

So:

```text
process
   |
ofile[fd]
   ↓
struct file
   |
   +── ip → struct inode
   |
   +── off
```

### inode number

```c
f->ip->inum
```

### read offset

```c
f->off
```

---

# Fork + file descriptor sharing

After:

```c
fd = open(...);
fork();
```

both parent and child have an FD referring to the same underlying open-file object.

Concept:

```text
parent fd ─┐
           ↓
       struct file
        /      \
       ↓        ↓
   inode       off
       ↑
child fd ─┘
```

Therefore:

- same inode
- same underlying file offset

The supplied test demonstrates this by having the child read 5 bytes and then checking the parent's offset.

### Example

Initially:

```text
offset = 0
```

Child:

```c
read(fd, buf, 5);
```

Afterward:

```text
offset = 5
```

The parent sees the changed offset because the `struct file` is shared.

---

# Common 2F trap

Check:

```c
f->readable
```

Correct validation:

```c
!f->readable
```

means unreadable.

Not:

```c
f->readable
```

---

# 2G — `peek2()`

Interface:

```c
int peek2(int fd, char *user_addr, int num_bytes);
```

Normal read:

```text
read()
 ↓
read data
 ↓
advance f->off
```

Peek:

```text
peek2()
 ↓
read data
 ↓
DO NOT change f->off
```

The assignment requires repeated peeks from the same offset to return the same bytes.

### Your `fileread()` is the clue

For an inode it does:

```c
ilock(f->ip);

if ((r = readi(f->ip, 1, addr, f->off, n)) > 0)
    f->off += r;

iunlock(f->ip);
```

For `peek2`, use the same `readi()` idea:

```c
ilock(f->ip);

r = readi(f->ip, 1, addr, f->off, n);

iunlock(f->ip);
```

but **do not**:

```c
f->off += r;
```

### Return values

```text
-1 → invalid / wrong type / unreadable FD
-2 → EOF
positive → bytes copied
```

The test checks:

```text
peek 5 → same 5 bytes
peek 5 → same 5 bytes
peek 2 → same 2 bytes
read 2 → offset advances
peek 3 → starts from new offset
```



---

# 3. VIRTUAL MEMORY

---

# 3A — `pte_valid()`

Interface:

```c
int pte_valid(uint64 va);
```

Return:

```text
1 → VA is mapped to a valid physical page
0 → not mapped / invalid
```

The lab explicitly points to:

```text
walk()
ismapped()
PTE_V
```



Your existing `ismapped()`:

```c
int
ismapped(pagetable_t pagetable, uint64 va)
{
  pte_t *pte = walk(pagetable, va, 0);

  if (pte == 0)
    return 0;

  if (*pte & PTE_V)
    return 1;

  return 0;
}
```

So your kernel helper can simply be:

```c
int
kpte_valid(uint64 va)
{
    return ismapped(myproc()->pagetable, va);
}
```

### Why `walk(..., 0)`?

`0` means:

```text
don't allocate a missing page-table page
```

We're only checking.

---

# 3B — PTE FLAGS

Interface:

```c
void get_pteflags(uint64 va);
```

Look up the PTE and print:

```text
R → readable
W → writable
X → executable
U → user accessible
```

The lab specifically names:

```c
PTE_V
PTE_R
PTE_W
PTE_X
```

and requires R/W/X/U output.

Typical logic:

```c
pte_t *pte = walk(myproc()->pagetable, va, 0);

if(pte == 0 || !(*pte & PTE_V))
    return;

printf("VA: 0x%016lx -> R:%d W:%d X:%d U:%d\n",
       va,
       !!(*pte & PTE_R),
       !!(*pte & PTE_W),
       !!(*pte & PTE_X),
       !!(*pte & PTE_U));
```

### IMPORTANT

`get_pteflags()` is meant to **print** the flags.

It is not the same as:

```c
pte_valid()
```

which returns:

```text
1 / 0
```

---

# 3C — `va2pa()`

Interface:

```c
uint64 va2pa(uint64 va);
```

Goal:

```text
Virtual Address → Physical Address
```

The assignment suggests:

```text
walk()
walkaddr()
PTE_V
```



---

## The most important formula

```text
PA = physical page base + virtual page offset
```

Use:

```c
uint64 pa = walkaddr(myproc()->pagetable, va);

if(pa == 0)
    return 0;

return pa + (va % PGSIZE);
```

### Why?

`walkaddr()` gives the physical page base.

Suppose:

```text
VA = 0x4064
```

and:

```text
walkaddr() = 0x87F43000
```

Then:

```text
offset = 0x4064 % 0x1000
       = 0x64

PA = 0x87F43000 + 0x64
   = 0x87F43064
```

The supplied third test demonstrates exactly this relationship.

---

# VA → PA mental model

```text
Virtual Address
┌──────────────────────┬────────────┐
│ Virtual Page Number  │   Offset   │
└──────────────────────┴────────────┘
           │                │
           │ page table     │ unchanged
           ↓                │
┌──────────────────────┬────────────┐
│ Physical Page Number │   Offset   │
└──────────────────────┴────────────┘
          Physical Address
```

### RULE:

```text
VPN changes
offset stays the same
```

---

# Test cases for VA → PA

## Test 1

Two different allocated pages:

```text
VA1: 0x4000
VA2: 0x5000
```

→ different virtual pages  
→ different physical pages.

## Test 2

Same variable address in parent and child:

```text
same VA
different PA
```

because parent and child have separate page tables.

## Test 3

```text
VA1 = 0x4000
VA2 = 0x4064
```

same virtual page, different offsets.

So:

```text
PA1 = same physical page base + 0x000
PA2 = same physical page base + 0x064
```

The lab's supplied outputs demonstrate all three cases.

---

# 3D — `getvasize(pid)`

Interface:

```c
int getvasize(int pid);
```

Return:

```text
size of process virtual address space
```

The key field is:

```c
struct proc
{
    ...
    uint64 sz;
}
```

So:

```text
PID
 ↓
find struct proc
 ↓
p->sz
 ↓
return
```

The lab explicitly asks you to study `sys_sbrk()` and `struct proc` for this.

### `sbrk()` relationship

Before:

```text
sz = 16384
```

Then:

```c
sbrk(1024)
```

After:

```text
sz = 17408
```

because:

```text
16384 + 1024 = 17408
```

`sbrk()` returns the **old size** / start of newly allocated area.

---

# 4. IMPORTANT XV6 FUNCTIONS

## `myproc()`

Returns:

```text
currently running process
```

Think:

```text
CPU
 ↓
myproc()
 ↓
struct proc *
```

---

## `allocproc()`

Creates/initializes a process structure.

Used for:

```text
fork/kfork
```

Good place to initialize newly added PCB fields:

```text
child_count = 0
syscall_counts[] = 0
```

---

## `fork() / kfork()`

Creates a child process.

Basic flow:

```text
allocproc()
    ↓
copy memory
    ↓
copy trapframe
    ↓
duplicate file references
    ↓
parent relationship
    ↓
RUNNABLE
```

Your supplied `kfork()` specifically:

```c
np->parent = p;
p->child_count++;
```

and:

```c
np->trapframe->a0 = 0;
```

---

## `exec()`

Replaces current process image.

```text
old program
    ↓
exec()
    ↓
new program
```

PID remains associated with the process; it is not a new process.

---

## `wait()`

Parent waits for a child.

Important state:

```text
running child
    ↓
exit()
    ↓
ZOMBIE
    ↓
wait()
    ↓
reaped
```

This is why `child_count` decreases on **reaping**, not exit.

---

## `exit()`

Terminates the process.

Child generally becomes:

```text
ZOMBIE
```

until parent reaps it.

---

# 5. STRUCT PROC — KNOW THIS

You should recognize fields such as:

```c
struct proc {
    struct spinlock lock;
    enum procstate state;
    void *chan;
    int killed;
    int xstate;
    int pid;
    struct proc *parent;

    uint64 kstack;
    uint64 sz;
    pagetable_t pagetable;
    struct trapframe *trapframe;

    struct file *ofile[NOFILE];
    struct inode *cwd;

    char name[16];

    // your additions:
    int child_count;
    int syscall_counts[...];
};
```

Exact arrangement varies by source, but the concepts are what matter.

---

# 6. FILE STRUCTURE — KNOW THIS

```c
struct file {
    enum { FD_NONE, FD_PIPE, FD_INODE, FD_DEVICE } type;
    int ref;
    char readable;
    char writable;

    struct pipe *pipe;
    struct inode *ip;
    uint off;
    short major;
};
```

Important fields:

```text
type
readable
writable
ip
off
```

For inode-backed file:

```text
f->ip        → inode
f->ip->inum  → inode number
f->off       → read/write offset
```

---

# 7. PTE FLAGS — MEMORIZE

```text
PTE_V → Valid
PTE_R → Read
PTE_W → Write
PTE_X → Execute
PTE_U → User accessible
```

For exam questions:

```text
V = Is there a valid mapping?
R = Can read?
W = Can write?
X = Can execute?
U = Can user access it?
```

---

# 8. SYSTEM CALL ARGUMENTS — QUICK TABLE

| Situation | Use |
|---|---|
| integer | `argint()` |
| userspace pointer/address | `argaddr()` |
| string | `argstr()` |
| file descriptor | `argfd()` |
| kernel → userspace copy | `copyout()` |
| userspace → kernel copy | `copyin()` |

Remember:

```text
argaddr() gives an address
copyout() actually safely writes data there
```

The lab explicitly defines these helpers this way.

---

# 9. KERNEL ↔ USER MEMORY

Never casually dereference a userspace pointer from kernel code.

Bad idea:

```c
*user_ptr = value;
```

Instead:

```c
copyout(...)
```

Similarly, when reading data from userspace:

```c
copyin(...)
```

This matters especially in:

```text
nfork()
peek2()
other pointer-taking syscalls
```

---

# 10. LOCKING — LAB EXAM FAVORITE

Why locks?

Because multiple CPUs can access shared kernel structures concurrently.

Typical pattern:

```c
acquire(&p->lock);

/* access protected process data */

release(&p->lock);
```

For the child counter, concurrent children could be reaped at the same time, so:

```text
parent.child_count--
```

must be protected.

The assignment explicitly calls this out.

---

# 11. MOST IMPORTANT DIFFERENCES

## `fork` vs `exec`

```text
fork → creates process
exec → replaces program
```

## `exit` vs `wait`

```text
exit → child terminates / becomes zombie
wait → parent reaps zombie
```

## `argint` vs `argaddr`

```text
argint  → integer
argaddr → pointer/address
```

## `fd` vs `struct file`

```text
fd → integer handle in process
struct file → kernel open-file object
```

## `inode` vs `struct file`

```text
inode → actual filesystem object metadata
struct file → open instance + offset + access mode
```

## `pte_valid` vs `get_pteflags`

```text
pte_valid    → returns 1/0
get_pteflags → prints R/W/X/U
```

---

# 12. COMMON BUGS / TRAPS

### 1. Forgetting `UPROGS`

Program compiles nowhere useful inside xv6 unless added to:

```make
UPROGS
```

### 2. Forgetting syscall number

Every new syscall needs a unique:

```c
#define SYS_...
```

### 3. Forgetting syscall-table entry

Need:

```c
[SYS_xxx] sys_xxx,
```

### 4. Forgetting `usys.pl`

Without:

```perl
entry("xxx");
```

the user-side syscall wrapper isn't generated.

### 5. Wrong argument extractor

Pointer:

```c
argaddr()
```

not `argint()`.

### 6. Writing directly to userspace

Use:

```c
copyout()
```

### 7. Advancing offset in `peek2`

Do NOT:

```c
f->off += r;
```

### 8. Decreasing `child_count` during `exit`

Wrong.

Decrease on:

```text
wait() / reaping
```

### 9. Incrementing `child_count` twice

Your `kfork()` already does:

```c
p->child_count++;
```

so don't add another increment in the caller.

### 10. Confusing page base and complete physical address

`walkaddr()` gives the physical page base.

Need:

```text
PA = walkaddr() + offset
```

---

# 13. BUILD / DEBUG WORKFLOW

After kernel modifications:

```bash
make clean
make
make qemu
```

If compilation fails:

```text
read the FIRST real error
```

not the final cascade.

Useful commands:

```bash
grep -R "function_name" kernel user
grep -n "sys_read" kernel/sysfile.c
grep -n "walkaddr" kernel/vm.c
grep -n "kfork" kernel/proc.c
```

---

# 14. VIVA QUESTIONS — RAPID FIRE

### What is xv6?

A small teaching OS based on Unix V6 ideas, implemented for RISC-V.

### What is a PCB?

Process Control Block:

```text
struct proc
```

contains information needed to manage a process.

### What does `myproc()` return?

Pointer to the current process's `struct proc`.

### Where is syscall number placed?

RISC-V:

```text
a7
```

### Where is syscall return value?

RISC-V:

```text
a0
```

### What instruction enters the kernel for a syscall?

```text
ecall
```

### What maps syscall number to kernel function?

```text
syscalls[] dispatch table
```

### Why `wait()`?

To reap a terminated child and collect its exit status.

### Why does a child become ZOMBIE?

It has terminated but its parent has not yet reaped it.

### What does `exec()` do?

Replaces process's current program image.

### Does `exec()` create a process?

No.

### What does `fork()` return?

```text
parent → child PID
child  → 0
error  → -1
```

### Why do parent and child share file offset after fork?

Their FD entries refer to the same underlying open-file object.

### What does `PTE_V` mean?

The PTE contains a valid mapping.

### What does `walk()` do?

Traverses a page table to find the PTE corresponding to a VA.

### What does `walkaddr()` give?

Physical page base corresponding to a mapped VA.

### What stays unchanged in VA → PA translation?

The **page offset**.

### Why can the same VA have different PAs in parent and child?

Different page tables.

### What is `PGSIZE`?

Page size:

```text
4096 bytes
```

### What does `va % PGSIZE` give?

The offset within the page.

### What is `p->sz`?

Current process virtual memory size.

---

# 15. TASK-BY-TASK ONE-LINERS

```text
1A → write a user program
1B → implement simple user file/terminal commands
1C → fork + exec + wait
1D → open + read + write + close
1E → multiple workers using fork

2A → add syscall, no arguments
2B → syscall + argint
2C → modify struct proc + process lifecycle
2D → syscall + pointers + kfork + copyout
2E → syscall statistics in dispatcher
2F → FD → struct file → inode/offset
2G → read without changing offset

3A → PTE validity
3B → PTE permission flags
3C → VA → PA
3D → process virtual address-space size
```

---

# 16. SUPER-COMPACT MEMORY MAP

When the examiner gives you a new syscall:

```text
1. user/user.h
      ↓ declaration

2. kernel/syscall.h
      ↓ syscall number

3. user/usys.pl
      ↓ wrapper

4. kernel/syscall.c
      ↓ declaration + dispatch table

5. kernel/sysproc.c / sysfile.c
      ↓ syscall handler

6. kernel/proc.c / vm.c / file.c
      ↓ actual kernel operation

7. kernel/defs.h
      ↓ shared helper declaration if needed
```

---

# 17. FINAL NIGHT-BEFORE REVISION

Make sure you can explain these without looking at code:

```text
fork()
exec()
wait()
exit()
myproc()

argint()
argaddr()
argfd()
copyin()
copyout()

struct proc
struct file
struct inode

child_count
syscall_counts[]

walk()
ismapped()
walkaddr()

PTE_V
PTE_R
PTE_W
PTE_X
PTE_U

VA
VPN
page offset
PA
PPN

sbrk()
p->sz
```

And especially these four diagrams:

```text
SYSCALL

user
 ↓
wrapper
 ↓
ecall
 ↓
syscall()
 ↓
sys_foo()
```

```text
PROCESS

fork()
 ├── parent
 └── child
      ↓
    exit()
      ↓
   ZOMBIE
      ↓
    wait()
      ↓
   REAPED
```

```text
FILE

process
  ↓
ofile[fd]
  ↓
struct file
  ├── ip → inode
  └── off
```

```text
MEMORY

VA
 ↓
page table / PTE
 ↓
physical page
 +
same page offset
 ↓
PA
```

## The biggest conceptual takeaway

The whole lab is really teaching three relationships:

```text
PROCESS
struct proc
    ↓
parent / child / PID / size / files

SYSCALL
user
  ↓
ecall
  ↓
kernel dispatcher
  ↓
handler

MEMORY
VA
  ↓
page table
  ↓
PTE
  ↓
PA
```

Once those three pipelines are clear, most of the individual tasks are just variations of them.

---
---
Absolutely. For the lab exam, these are the **code snippets worth memorizing/recognizing** rather than memorizing every full program.

## 1. User program skeleton

Most Task 1 programs follow:

```c
#include "kernel/types.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
    // code
    exit(0);
}
```

For an argument-based program:

```c
if(argc < 2){
    fprintf(2, "Usage: ...\n");
    exit(1);
}
```

Build inclusion in `Makefile`:

```make
UPROGS=\
    $U/_hello\
    ...
```

---

# Task 1 — Important snippets

## 1A. `hello`

```c
printf("Hello World!\n");
exit(0);
```

---

## 1B. Reading a file

### Open

```c
int fd = open(filename, O_RDONLY);

if(fd < 0){
    fprintf(2, "cannot open %s\n", filename);
    exit(1);
}
```

### Read loop

```c
char buf[512];
int n;

while((n = read(fd, buf, sizeof(buf))) > 0){
    write(1, buf, n);
}

close(fd);
```

Remember:

```text
open → read → process → close
```

### `head` idea

```c
while((n = read(fd, &c, 1)) > 0){
    write(1, &c, 1);

    if(c == '\n'){
        lines++;
        if(lines == N)
            break;
    }
}
```

### `tail` idea

Maintain the last `N` lines, then print them.

The important exam point is usually **how you use `open/read/close`**, not the exact implementation.

---

# 1C. `cmd` — fork + exec + wait

This is **very important**.

```c
int pid = fork();

if(pid < 0){
    fprintf(2, "fork failed\n");
    exit(1);
}

if(pid == 0){
    exec(argv[1], &argv[1]);
    fprintf(2, "exec failed\n");
    exit(1);
}

wait(0);
exit(0);
```

### Mental model

```text
Parent
  |
 fork()
 /    \
child  parent
 |       |
exec     wait
 |
program
```

### `exec`

```c
exec(program, args);
```

`exec()` **replaces the current process image**.

---

# 1D. `cp` — file copying

Core loop:

```c
int fd1 = open(src, O_RDONLY);
int fd2 = open(dst, O_CREATE | O_WRONLY);

char buf[512];
int n;

while((n = read(fd1, buf, sizeof(buf))) > 0)
    write(fd2, buf, n);

close(fd1);
close(fd2);
```

The key pattern:

```text
read from source
      ↓
buffer
      ↓
write to destination
```

---

# 1E. `mgrep` — one child per file

Basic structure:

```c
for(int i = 2; i < argc; i++){

    int pid = fork();

    if(pid < 0){
        fprintf(2, "fork failed\n");
        exit(1);
    }

    if(pid == 0){
        // child handles argv[i]

        int fd = open(argv[i], O_RDONLY);

        if(fd < 0){
            fprintf(2, "cannot open %s\n", argv[i]);
            exit(1);
        }

        // grep/search file

        close(fd);
        exit(0);
    }
}
```

Then parent waits:

```c
for(int i = 2; i < argc; i++)
    wait(0);
```

### Sequential version

To avoid interleaved output:

```c
for(int i = 2; i < argc; i++){

    int pid = fork();

    if(pid == 0){
        // process file
        exit(0);
    }

    wait(0);
}
```

---

# Task 2 — System-call plumbing

This is **probably the highest-value thing to memorize**.

Whenever you add a syscall, think:

```text
user/user.h
      ↓
user/usys.pl
      ↓
kernel/syscall.h
      ↓
kernel/syscall.c
      ↓
kernel/sysproc.c / sysfile.c
      ↓
kernel implementation/helper
```

---

# 2A. `getppid`

## User declaration

```c
int getppid(void);
```

## Syscall number

```c
#define SYS_getppid  ...
```

## `usys.pl`

```perl
entry("getppid");
```

## Dispatcher

```c
[SYS_getppid] sys_getppid,
```

## Kernel implementation

```c
uint64
sys_getppid(void)
{
    struct proc *p = myproc();

    if(p->parent == 0)
        return -1;

    return p->parent->pid;
}
```

### Remember

```c
myproc()
```

means:

> Give me the currently running process.

---

# 2B. `square`

Argument extraction:

```c
uint64
sys_square(void)
{
    int x;

    if(argint(0, &x) < 0)
        return -1;

    return x * x;
}
```

### Important

```c
argint(0, &x)
```

means:

> Get syscall argument 0 as an integer.

---

# 2C. `child_count`

## In `struct proc`

```c
int child_count;
```

Initialize:

```c
p->child_count = 0;
```

Increment when a child is created:

```c
p->child_count++;
```

Decrement when child is **reaped**:

```text
exit ≠ decrement
wait/reap = decrement
```

### Current process

```c
struct proc *p = myproc();
int count = p->child_count;
```

### Searching process table

Typical pattern:

```c
struct proc *p;

for(p = proc; p < &proc[NPROC]; p++){
    acquire(&p->lock);

    if(p->pid == pid){
        int count = p->child_count;
        release(&p->lock);
        return count;
    }

    release(&p->lock);
}

return -1;
```

Very important viva point:

```text
proc[] is in proc.c
↓
helper in proc.c
↓
sysproc.c calls helper
```

---

# 2D. `nfork`

The key code pattern:

```c
uint64
sys_nfork(void)
{
    int n;
    uint64 addr;

    if(argint(0, &n) < 0)
        return -1;

    if(argaddr(1, &addr) < 0)
        return -1;

    if(n < 0 || n > NPROC)
        return -1;

    struct proc *p = myproc();

    int pids[NPROC];

    for(int i = 0; i < n; i++){
        int pid = kfork();

        if(pid < 0)
            return -1;

        pids[i] = pid;
    }

    if(copyout(p->pagetable,
               addr,
               (char *)pids,
               n * sizeof(int)) < 0)
        return -1;

    return n;
}
```

### Three extremely important functions here

```c
argint()
argaddr()
copyout()
```

Think:

```text
userspace argument
      ↓
argint / argaddr
      ↓
kernel
      ↓
copyout
      ↓
userspace array
```

---

# `kfork()` — key section

Your important code:

```c
if ((np = allocproc()) == 0)
    return -1;
```

Copy address space:

```c
if (uvmcopy(p->pagetable, np->pagetable, p->sz) < 0){
    freeproc(np);
    release(&np->lock);
    return -1;
}
```

Copy trapframe:

```c
*(np->trapframe) = *(p->trapframe);
np->trapframe->a0 = 0;
```

Why?

```text
parent fork() → child PID
child fork()  → 0
```

Set parent:

```c
np->parent = p;
```

Make child runnable:

```c
np->state = RUNNABLE;
```

---

# 2E. Syscall counters

## In `struct proc`

```c
int syscall_counts[MAX_SYSCALLS];
```

Initialize to zero when process is created.

## Central dispatcher

This is the **important snippet**:

```c
void
syscall(void)
{
    int num;
    struct proc *p = myproc();

    num = p->trapframe->a7;

    if(num > 0 && num < NELEM(syscalls)
                && syscalls[num]){

        p->syscall_counts[num]++;

        p->trapframe->a0 = syscalls[num]();

    } else {
        printf("%d %s: unknown sys call %d\n",
               p->pid, p->name, num);

        p->trapframe->a0 = -1;
    }
}
```

### Memorize this

```c
num = p->trapframe->a7;
```

`a7` contains the syscall number.

```c
p->trapframe->a0 = syscalls[num]();
```

`a0` gets the return value.

---

# 2F. File descriptor information

Given:

```c
struct file {
    enum { FD_NONE, FD_PIPE, FD_INODE, FD_DEVICE } type;
    int ref;
    char readable;
    char writable;
    struct pipe *pipe;
    struct inode *ip;
    uint off;
    short major;
};
```

## Get inode number

Conceptually:

```c
struct proc *p = myproc();
struct file *f = p->ofile[fd];

if(f == 0 ||
   f->type != FD_INODE ||
   !f->readable ||
   f->ip == 0)
    return -1;

return f->ip->inum;
```

## Get current offset

```c
return f->off;
```

### Most important relationship

```text
fd
 ↓
p->ofile[fd]
 ↓
struct file
 ├── ip
 ├── off
 ├── readable
 └── type
```

---

# 2G. `peek2`

This one is worth knowing very well.

## `filepeek()`

```c
int
filepeek(struct file *f, uint64 addr, int n)
{
    int r;

    if(f->readable == 0 || n < 0)
        return -1;

    if(f->type != FD_INODE)
        return -1;

    ilock(f->ip);

    r = readi(f->ip, 1, addr, f->off, n);

    iunlock(f->ip);

    if(r == 0)
        return -2;

    return r;
}
```

### Why doesn't the offset change?

Because:

```c
readi(..., f->off, n);
```

uses the current offset, but we **never do**:

```c
f->off += r;
```

So:

```text
read  → offset changes
peek2 → offset doesn't change
```

## Syscall wrapper

```c
uint64
sys_peek2(void)
{
    struct file *f;
    int n;
    uint64 p;

    argaddr(1, &p);
    argint(2, &n);

    if(argfd(0, 0, &f) < 0)
        return -1;

    return filepeek(f, p, n);
}
```

---

# Task 3 — Virtual memory

## 3A. `pte_valid`

Your `ismapped()` logic:

```c
int
ismapped(pagetable_t pagetable, uint64 va)
{
    pte_t *pte = walk(pagetable, va, 0);

    if(pte == 0)
        return 0;

    if(*pte & PTE_V)
        return 1;

    return 0;
}
```

Helper:

```c
int
kpte_valid(uint64 va)
{
    return ismapped(myproc()->pagetable, va);
}
```

### Core idea

```text
VA
 ↓
walk()
 ↓
PTE
 ↓
PTE_V ?
 ↓
1 / 0
```

---

# 3B. PTE flags

Look up the PTE:

```c
pte_t *pte = walk(myproc()->pagetable, va, 0);
```

Check flags:

```c
int r = (*pte & PTE_R) != 0;
int w = (*pte & PTE_W) != 0;
int x = (*pte & PTE_X) != 0;
int u = (*pte & PTE_U) != 0;
```

Print:

```c
printf("VA: %p -> R:%d W:%d X:%d U:%d\n",
       va, r, w, x, u);
```

### Know the meanings

```text
PTE_R → Read
PTE_W → Write
PTE_X → Execute
PTE_U → User accessible
PTE_V → Valid
```

---

# 3C. `va2pa`

This is a **very important VM formula**.

```c
uint64
kva2pa(uint64 va)
{
    uint64 pa = walkaddr(myproc()->pagetable, va);

    if(pa == 0)
        return 0;

    return pa + (va % PGSIZE);
}
```

### Understand this

`walkaddr()` gives the **physical page base**.

Suppose:

```text
VA = 0x1234
page base PA = 0x9000
```

Then:

```text
VA offset = 0x234

PA = 0x9000 + 0x234
   = 0x9234
```

Formula:

```text
PA = physical_page_base + page_offset
```

---

# 3D. `getvasize`

The key field is:

```c
p->sz
```

For current process:

```c
struct proc *p = myproc();
return p->sz;
```

For a specific PID:

```c
struct proc *p;

for(p = proc; p < &proc[NPROC]; p++){
    acquire(&p->lock);

    if(p->pid == pid){
        uint64 sz = p->sz;
        release(&p->lock);
        return sz;
    }

    release(&p->lock);
}

return -1;
```

---

# System-call argument functions — MEMORIZE

These appear everywhere.

### Integer

```c
int x;

if(argint(0, &x) < 0)
    return -1;
```

### Address

```c
uint64 addr;

if(argaddr(1, &addr) < 0)
    return -1;
```

### File descriptor

```c
struct file *f;

if(argfd(0, 0, &f) < 0)
    return -1;
```

### Copy kernel → user

```c
copyout(p->pagetable,
        user_addr,
        (char *)kernel_buffer,
        size);
```

### Copy user → kernel

```c
copyin(p->pagetable,
       (char *)kernel_buffer,
       user_addr,
       size);
```

---

# Most important xv6 functions

For the lab exam, recognize these instantly:

```text
fork()
exec()
wait()
exit()

kfork()
allocproc()
myproc()

argint()
argaddr()
argfd()

copyin()
copyout()

walk()
walkaddr()
uvmcopy()

open()
read()
write()
close()

fileread()
readi()

ilock()
iunlock()
```

---

# Most important structures

## `struct proc`

Think:

```text
process
 ├── pid
 ├── parent
 ├── state
 ├── pagetable
 ├── sz
 ├── trapframe
 ├── ofile[]
 ├── cwd
 └── child_count
```

## `struct file`

```text
file descriptor
 ├── type
 ├── ref
 ├── readable
 ├── writable
 ├── ip
 └── off
```

---

# The 15 snippets I'd memorize first

If you're extremely short on revision time, focus on these:

```c
struct proc *p = myproc();
```

```c
if(argint(0, &x) < 0)
    return -1;
```

```c
if(argaddr(1, &addr) < 0)
    return -1;
```

```c
if(argfd(0, 0, &f) < 0)
    return -1;
```

```c
p->trapframe->a7
```

```c
p->trapframe->a0
```

```c
copyout(p->pagetable, addr, buf, n)
```

```c
int pid = fork();
```

```c
if(pid == 0)
    exec(...);
else
    wait(0);
```

```c
int fd = open(file, O_RDONLY);
```

```c
while((n = read(fd, buf, sizeof(buf))) > 0)
    write(1, buf, n);
```

```c
pte_t *pte = walk(myproc()->pagetable, va, 0);
```

```c
*pte & PTE_V
```

```c
uint64 pa = walkaddr(myproc()->pagetable, va);
```

```c
return pa + (va % PGSIZE);
```

And the **single most important mental picture**:

```text
USER PROGRAM
    |
    | syscall()
    v
usys.S
    |
   ecall
    v
TRAP HANDLING
    |
    v
syscall()
    |
    | a7 = syscall number
    v
syscalls[][]
    |
    v
sys_*()
    |
    v
kernel helper / data structure
```

That pipeline, plus `fork/exec/wait`, `arg*`, `copyout`, `walk/walkaddr`, and `struct proc/file`, covers a huge fraction of the likely lab-viva questions.