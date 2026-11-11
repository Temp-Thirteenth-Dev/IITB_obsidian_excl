https://ocw.mit.edu/courses/6-828-operating-system-engineering-fall-2012/pages/exams/
https://pdos.csail.mit.edu/6.1810/2025/quiz.html
PYQs sent by sir
PYQs in Mythli mam's course

---

# ChatGPT 
https://chatgpt.com/share/6aee2c6a-8e78-83ee-95e2-b45732a4656d

Oh, this helps a LOT. Your syllabus is broader than a typical Operating Systems course. I'd actually classify CS744 as a Computer Systems + OS + Networking + Distributed Systems course.

Looking at your lecture schedule, I'd expand the resources in six directions. Some of these are particularly valuable because they have actual examination papers with solutions, not just lecture notes.

# 1. Computer Systems and Architecture

Your lectures 0–2: Introduction to Computing Systems, architecture basics, CSAPP Chapter 1.

Excellent source of exam questions

## Carnegie Mellon University — 15-213: Introduction to Computer Systems

Links:

- [Past examinations and solutions](https://www.cs.cmu.edu/~213/exams.html?utm_source=chatgpt.com)
    
- [Programming assignments and labs](https://www.cs.cmu.edu/~213/labs.html?utm_source=chatgpt.com)
    
- [CS:APP textbook website](https://csapp.cs.cmu.edu/3e/?utm_source=chatgpt.com)
    

This is one of the best matches for the beginning of your syllabus.

The examination archive contains a practice exam and many previous midterms and finals, with model solutions.

The assignments include:

|Lab|Topic|
|---|---|
|Data Lab|Bit manipulation|
|Bomb Lab|Assembly and debugging|
|Attack Lab|Stack and buffer overflow|
|Cache Lab|Memory hierarchy and caching|
|Malloc Lab|Dynamic memory allocation|
|Shell Lab|Processes and Unix system calls|
|Proxy Lab|Networking and sockets|

CMU School of Computer Science+1

What I'd solve: Especially the old exam questions on assembly, stack discipline, memory hierarchy, signals, Unix I/O and concurrency.

This also complements your lecture on `malloc()`, `sbrk()` and `free()`.

# 2. Operating Systems — the main chunk

You already have the IIT Bombay xv6 problem sets from my previous recommendation. I'd add two more.

## Massachusetts Institute of Technology — 6.1810: Operating System Engineering

Very high priority

Links:

- [Course website](https://pdos.csail.mit.edu/6.S081/2025/?utm_source=chatgpt.com)
    
- [Lecture schedule and assignments](https://pdos.csail.mit.edu/6.S081/2025/schedule.html?utm_source=chatgpt.com)
    
- [Past midterms and finals with solutions](https://pdos.csail.mit.edu/6.1810/2025/quiz.html?utm_source=chatgpt.com)
    

This is particularly interesting because its syllabus overlaps with yours almost lecture by lecture.

|Your CS744 topic|MIT 6.1810 material|
|---|---|
|OS organization|OS organization|
|Page tables|Page tables|
|System call mechanism|System call entry/exit|
|Address space|Page faults, superpages|
|Copy-on-write|COW fork lab|
|Synchronization|Locking, coordination|
|Networking|Network driver lab|
|File systems|File system lab|
|Crash recovery|Logging and crash recovery|
|Containers and VMs|Dedicated lecture and homework|

It also has midterms and finals from multiple years, including 2024 and 2025, with solutions.

MIT CSAIL PDOS+1

One important caveat: The modern MIT xv6 used here is RISC-V-based. Your uploaded CS219 paper has x86-specific concepts such as EIP, ESP and CR3. Therefore, use MIT for OS concepts and question styles, but don't blindly transfer architecture-specific details.

## University of Washington — CSE 451: Operating Systems

Links:

- [Course website](https://courses.cs.washington.edu/courses/cse451/26sp/?utm_source=chatgpt.com)
    
- [Assignments and problem sets](https://courses.cs.washington.edu/courses/cse451/26sp/?utm_source=chatgpt.com)
    

Why this one?

It uses a teaching OS derived from xv6 and has programming labs, short-answer lab questions, individual problem sets and two examinations.

Its labs include protected system calls, multiprocessing, paged virtual memory and crash-resilient file systems.

This is an excellent source for questions requiring you to understand how the kernel actually works.

# 3. OSTEP — your best topic-wise practice resource

Must use

Operating Systems: Three Easy Pieces

- [Free textbook](https://pages.cs.wisc.edu/~remzi/OSTEP/?utm_source=chatgpt.com)
    
- [Homework and simulators](https://pages.cs.wisc.edu/~remzi/OSTEP/Homework/homework.html?utm_source=chatgpt.com)
    
- [GitHub repository](https://github.com/remzi-arpacidusseau/ostep-homework?utm_source=chatgpt.com)
    

You already have OSTEP in your syllabus, but its homework page is something you should know about.

It has simulators that generate new practice problems with different inputs. You can also generate answers after attempting them yourself.

Particularly relevant exercises:

|Your syllabus|OSTEP practice|
|---|---|
|Processes|Process API|
|Virtual memory|Paging, TLBs, multilevel paging|
|Concurrency|Threads, locks, condition variables|
|Synchronization|Semaphores|
|File systems|Filesystem implementation|
|Crash recovery|Journaling, fsck|

pages.cs.wisc.edu

My recommendation: Use this when you have understood a topic but want more numerical problems or different variations of the same concept.

# 4. Networking, sockets and HTTP

Your lectures 21–22: Socket abstraction, network processing, web applications, HTTP and RPC.

## University of Massachusetts Amherst — Computer Networking (Kurose & Ross)

Excellent practice generator

Links:

- [Interactive end-of-chapter problems](https://www-net.cs.umass.edu/kurose_ross/interactive/?utm_source=chatgpt.com)
    
- [Knowledge checks](https://www-net.cs.umass.edu/kurose_ross/knowledgechecks/?utm_source=chatgpt.com)
    
- [Socket programming assignments](https://www-net.cs.umass.edu/kurose_ross/programming.php?utm_source=chatgpt.com)
    
- [Wireshark labs](https://www-net.cs.umass.edu/kurose_ross/wireshark.php?utm_source=chatgpt.com)
    
- [Lecture slides](https://www-net.cs.umass.edu/kurose_ross/ppt.php?utm_source=chatgpt.com)
    

This is an excellent resource because the interactive exercises can generate new problem instances.

Some particularly relevant topics:

|Topic|Available practice|
|---|---|
|HTTP|HTTP GET, HTTP RESPONSE|
|Sockets|Socket programming|
|Application layer|DNS, SMTP, web applications|
|TCP/UDP|Sequence numbers, ACKs, retransmission|
|Networking|Packet forwarding, routing|
|Network programming|Client-server, web server, HTTP proxy|

The programming assignments include simple client-server applications, web servers, UDP pingers and HTTP proxy servers.

www-net.cs.umass.edu+2

My recommendation: Focus on Chapter 2, particularly HTTP and socket programming. Don't spend too much time on advanced routing topics unless your lectures cover them.

# 5. RPC and distributed systems

Your lectures 22 and 24: RPC, checkpointing, recovery and consistent global state.

This is a slightly different area from conventional OS.

## Carnegie Mellon University — 15-440: Distributed Systems

Links:

- [Course website](https://courseweb.sp.cs.cmu.edu/15-440/fall-2026/?utm_source=chatgpt.com)
    
- [Past midterms and finals with solutions](https://courseweb.sp.cs.cmu.edu/15-440/fall-2026/past-exams/?utm_source=chatgpt.com)
    
- [Course syllabus](https://courseweb.sp.cs.cmu.edu/15-440/fall-2026/syllabus/?utm_source=chatgpt.com)
    

This course covers concurrency, communication across networks, failures, distributed programming and system design.

The particularly useful part is its examination archive:

- Fall 2024 Midterm I with solutions.
    
- Fall 2022 and 2021 Midterm I solutions.
    
- Fall 2024 Midterm II with and without solutions.
    
- Older final examinations.
    
- Practice debugging problems.
    

courseweb.sp.cs.cmu.edu+1

I'd use this for RPC, client-server communication, concurrency and reasoning about failures.

## University of Illinois Urbana-Champaign — CS 425: Distributed Systems

Especially useful for checkpointing

Links:

- [Course website](https://courses.grainger.illinois.edu/CS425/fa2026/?utm_source=chatgpt.com)
    
- [Lecture notes and previous course material](https://courses.physics.illinois.edu/cs425/fa2025/lectures.html?utm_source=chatgpt.com)
    
- [Assignments, practice midterm and solutions](https://courses.physics.illinois.edu/cs425/fa2025/assignments.html?utm_source=chatgpt.com)
    

This is one of the most relevant resources for your lecture 24.

The course specifically covers:

- Time and ordering.
    
- Distributed snapshots.
    
- Global states.
    
- Consensus and agreement.
    
- Distributed algorithms.
    

Its Fall 2025 course has a dedicated lecture on snapshots, a practice midterm, homework assignments and solutions.

courses.physics.illinois.edu+1

For your checkpointing lecture, I would particularly study the Snapshots lecture (Lecture 13).

Another useful resource is University of Washington's distributed systems lecture notes:

[UW CS 425/525 — Clocks, Consistent Cuts, Snapshots and Global Predicate Evaluation](https://homes.cs.washington.edu/~arvind/cs425/lectureNotes/?utm_source=chatgpt.com)

These notes cover precisely the terminology in your syllabus.

Homes at UW+1

# 6. Containers, virtual machines and cgroups

Your lectures 23–27: Containerization, namespaces, cgroups, VMs and virtualization.

## MIT 6.1810 — Containers and Virtual Machines

You can reuse the MIT course I mentioned earlier, but there is one particular resource worth highlighting:

[Containers and Virtual Machines — MIT lecture schedule](https://pdos.csail.mit.edu/6.S081/2025/schedule.html?utm_source=chatgpt.com)

[MIT container homework](https://pdos.csail.mit.edu/6.S081/2025/homework/paper-blending.html?utm_source=chatgpt.com)

The homework asks students to read about Firecracker, gVisor and LXC, and reason about their isolation mechanisms.

This is useful for the conceptual comparison questions that could appear in your exam.

MIT CSAIL PDOS+1

## Linux documentation — cgroups

- [cgroups(7) — Linux manual](https://man7.org/linux/man-pages/man7/cgroups.7.html?utm_source=chatgpt.com)
    
- [Control Group v2 — Linux kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html?utm_source=chatgpt.com)
    

These explain process grouping, resource limits, resource accounting and the cgroup hierarchy.

Linux manual page+1

I'd use these primarily as supplementary reading after your professor's lecture slides.

# My final shortlist for your CS744 end-sem

There are quite a few resources here, so I wouldn't attempt every assignment from every university.

This is the order I'd personally follow:

| Priority | Resource                  | What to use it for                                    |
| -------- | ------------------------- | ----------------------------------------------------- |
| 1        | IIT Bombay CS744 material | Your professor's exact syllabus, terminology and labs |
| 2        | MIT 6.1810                | OS, xv6, VM, synchronization, filesystems, containers |
| 3        | CMU 15-213                | Computer systems, architecture, malloc, exams         |
| 4        | OSTEP homework            | Unlimited variations of OS problems                   |
| 5        | UMass Kurose & Ross       | HTTP, sockets, networking                             |
| 6        | UIUC CS425                | Snapshots and consistent global states                |
| 7        | CMU 15-440                | RPC, distributed systems and failure handling         |

## A useful distinction for your preparation

Your syllabus has two topics that might seem similar but deserve separate practice:

Filesystem crash consistency (lectures 18–20): What happens when metadata or data blocks are written to disk in an unexpected order? Practice OSTEP's journaling/fsck problems and MIT's logging homework.

Distributed checkpointing and recovery (lecture 24): How do multiple processes obtain a consistent global snapshot? Practice UIUC's snapshots lecture and UW's consistent-cut notes.

These are different problems, and I'd prepare them separately.

My biggest recommendation: Start with MIT 6.1810's past examinations, CMU 15-213's examination archive and OSTEP's homework generators. Together, they give you a substantial amount of additional practice without requiring you to follow three entire courses.

Also, don't overlook the assignments from your own CS744 course. They are likely to be the best indication of the depth and conventions expected by your instructor.

The goal should be to find questions that test the same underlying concepts as your PYQs, even when the question is framed differently.

---

# Gemini
https://share.gemini.google/4abY51DZOMZw

Ah, I see! This is for **CS744 (Design and Engineering of Computing Systems)** taught by Prof. Purushottam Kulkarni at IIT Bombay.

  

Looking closely at your syllabus schedule, the course isn't just a standard OS course. It's heavily based on four specific textbooks/paradigms:

  

1. **[ostep]** - _Operating Systems: Three Easy Pieces_
    
      
    
2. **[xv6book]** - The MIT xv6 textbook
    
      
    
3. **[csapp]** - _Computer Systems: A Programmer's Perspective_ (which covers the architecture basics, memory hierarchy, and systems programming)
    
      
    
4. **[dive]** - _Dive into Systems_
    
      
    

Since your syllabus directly maps to these books, the **absolute best** places to find relevant practice sets, assignments, and exam questions are the home universities of the authors of these books. Here are the specific links you need:

  

### 1. For [csapp] and Architecture/Systems Basics

This book is the gold standard for "Computing Systems" and is written by professors at **Carnegie Mellon University (CMU)**. The course is known as "15-213: Introduction to Computer Systems (ICS)".

  

- **CMU 15-213 Exams (The Holy Grail for CSAPP):** [http://www.cs.cmu.edu/~213/exams.html](http://www.cs.cmu.edu/~213/exams.html)
    
      
    - _Focus on:_ Their questions on memory hierarchy, virtual memory, malloc/free implementations, and signals.
        
          
        
- **CSAPP Student Site:** [http://csapp.cs.cmu.edu/3e/students.html](http://csapp.cs.cmu.edu/3e/students.html) (Contains practice problems and lab assignments).
    
      
    

### 2. For [ostep] and OS Abstractions

This book was written by Remzi and Andrea Arpaci-Dusseau at the **University of Wisconsin-Madison**. Their OS course (CS 537) maps 1:1 with the OSTEP readings in your syllabus.

  

- **UW-Madison CS 537 Old Exams:** [https://pages.cs.wisc.edu/~remzi/Classes/537/Spring2018/oldexams.html](https://www.google.com/search?q=https://pages.cs.wisc.edu/~remzi/Classes/537/Spring2018/oldexams.html) (You can change the year in the URL to find more recent ones, e.g., `Fall2023`).
    
      
    
- **OSTEP Homework Simulations:** At the end of every OSTEP chapter, there are simulation homeworks. You can download the scripts here: [https://github.com/remzi-arpacidusseau/ostep-homework](https://github.com/remzi-arpacidusseau/ostep-homework)
    
      
    

### 3. For xv6 Internals

As mentioned previously, xv6 was developed at **MIT**.

  

- **MIT 6.1810 (formerly 6.828):** [https://pdos.csail.mit.edu/6.1810/2023/schedule.html](https://pdos.csail.mit.edu/6.1810/2023/schedule.html)
    
      
    - _Focus on:_ The midterms and finals linked at the top of their schedule. These exams ask the exact kind of code-tracing questions (like the trapframe and page table questions) you shared in your `endsem-key.pdf`.
        
          
        

### 4. For Pthreads, Futex, and Concurrency

Your syllabus goes quite deep into concurrency (Condition variables, Futex, Reader-Writer). For this, **Stanford's CS110/CS111** is excellent.

  

- **Stanford CS111:** [https://cs111.stanford.edu/](https://cs111.stanford.edu/) (Look at the "Exams" tab for excellent concurrency and synchronization problems).
    
      
    

**Pro-Tip for your End-Sem:** Because your syllabus covers `malloc-sbrk-free`, `huge pages`, and `VFS`, make sure you heavily practice the **CSAPP malloc lab/exam questions** from CMU and the **file system crash-consistency problems** from the UW-Madison (OSTEP) old exams.