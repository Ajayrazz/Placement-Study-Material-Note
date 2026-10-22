# Operating Systems (OS) — The SDE-1 & SWE Intern Interview Handbook

> **Target Audience:** SWE Interns & SDE-1 Candidates  
> **Objective:** Master OS architecture, concurrency, memory management, low-level mechanics, real-world analogies, and exact interview talking points.

---

## Table of Contents
1. [OS Fundamentals & Architecture](#1-os-fundamentals--architecture)
   - [Dual-Mode Operation (Kernel vs. User Mode)](#dual-mode-operation-kernel-vs-user-mode)
   - [System Calls vs. Library Calls](#system-calls-vs-library-calls)
   - [Interrupts, Traps, and Exceptions](#interrupts-traps-and-exceptions)
2. [Process Management](#2-process-management)
   - [Process Anatomy & Memory Layout](#process-anatomy--memory-layout)
   - [Process Control Block (PCB)](#process-control-block-pcb)
   - [Process States & Lifecycle](#process-states--lifecycle)
   - [Process Creation: `fork()`, `exec()`, and Copy-On-Write (COW)](#process-creation-fork-exec-and-copy-on-write-cow)
   - [Zombie vs. Orphan Processes](#zombie-vs-orphan-processes)
3. [Threads & Concurrency](#3-threads--concurrency)
   - [Process vs. Thread (Deep Comparison)](#process-vs-thread-deep-comparison)
   - [User-Level Threads (ULT) vs. Kernel-Level Threads (KLT)](#user-level-threads-ult-vs-kernel-level-threads-klt)
   - [Context Switching Mechanics & Overheads](#context-switching-mechanics--overheads)
4. [CPU Scheduling](#4-cpu-scheduling)
   - [Preemptive vs. Non-Preemptive](#preemptive-vs-non-preemptive)
   - [Core Algorithms (FCFS, SJF/SRTF, Round Robin, Priority, MLFQ)](#core-algorithms)
   - [The Convoy Effect & Starvation (Aging)](#the-convoy-effect--starvation-aging)
5. [Process Synchronization (High Priority)](#5-process-synchronization-high-priority)
   - [Race Conditions & Critical Section Problem](#race-conditions--critical-section-problem)
   - [Hardware Primitives (Test-and-Set, Compare-and-Swap)](#hardware-primitives)
   - [Mutex vs. Semaphore (Binary vs. Counting)](#mutex-vs-semaphore)
   - [Spinlock vs. Sleeping Lock](#spinlock-vs-sleeping-lock)
   - [Classic Problems (Producer-Consumer, Readers-Writers, Dining Philosophers)](#classic-synchronization-problems)
6. [Deadlocks](#6-deadlocks)
   - [The 4 Coffman Conditions](#the-4-coffman-conditions)
   - [Deadlock Strategies (Prevention, Avoidance, Detection, Ignorance)](#deadlock-handling-strategies)
   - [Banker's Algorithm & Safe State](#bankers-algorithm)
   - [Deadlock vs. Livelock vs. Starvation](#deadlock-vs-livelock-vs-starvation)
7. [Memory Management & Virtual Memory (Crucial)](#7-memory-management--virtual-memory-crucial)
   - [Logical vs. Physical Address Space & MMU](#logical-vs-physical-address-space--mmu)
   - [Fragmentation (Internal vs. External)](#fragmentation)
   - [Paging Architecture & Multi-Level Page Tables](#paging-architecture)
   - [Translation Lookaside Buffer (TLB)](#translation-lookaside-buffer-tlb)
   - [Demand Paging & Page Fault Handling Sequence](#demand-paging--page-fault-handling-sequence)
   - [Thrashing & Working Set Model](#thrashing--working-set-model)
   - [Page Replacement Algorithms (FIFO, Belady's Anomaly, LRU, Clock)](#page-replacement-algorithms)
8. [File Systems & Storage](#8-file-systems--storage)
   - [Linux Inodes Architecture](#linux-inodes-architecture)
   - [Hard Links vs. Soft (Symbolic) Links](#hard-links-vs-soft-symbolic-links)
   - [Disk Scheduling Algorithms (SCAN, C-SCAN, LOOK)](#disk-scheduling-algorithms)
9. [Inter-Process Communication (IPC)](#9-inter-process-communication-ipc)
10. [Top 20 SDE-1 / Intern Interview Questions & Model Answers](#10-top-20-sde-1--intern-interview-questions--model-answers)

---

## 1. OS Fundamentals & Architecture

### What is an Operating System?
An Operating System is a system software that acts as an intermediary between the computer hardware and application programs. It has two main functions:
1. **Resource Manager:** Arbitrates CPU time, memory space, file storage, and I/O devices fairly and efficiently among competing applications.
2. **Extended Machine (Abstraction Provider):** Hides bare-metal hardware complexities, exposing clean abstractions (files, processes, sockets) via system calls.

> **Real-World Analogy: Airport Air Traffic Control (ATC)**  
> Think of computer hardware as runways, terminals, and fuel pumps. The planes are user applications. The ATC is the Operating System. No plane can land, take off, or taxi whenever it wants—it must request permission. ATC ensures planes don't collide (concurrency control), schedules runway time (CPU scheduling), and assigns gates (memory management).

---

### Dual-Mode Operation (Kernel vs. User Mode)
To prevent rogue applications from halting the machine or reading another program's private memory, modern CPUs enforce hardware-level privilege separation:

```
+--------------------------------------------------------------+
|                     USER SPACE (Ring 3)                      |
|   Web Browser, Code Editor, Video Game, JVM Runtime          |
+--------------------------------------------------------------+
               |                                ^
  System Call  | (Trap / Syscall Instruction)   | Return to User
  (Mode Bit 1) v                                | (Mode Bit 0)
+--------------------------------------------------------------+
|                    KERNEL SPACE (Ring 0)                     |
|   Process Scheduler, Virtual Memory Manager, Device Drivers  |
|   Direct Hardware Access (CPU, RAM, Disks, Network NIC)      |
+--------------------------------------------------------------+
```

| Dimension | User Mode (Ring 3) | Kernel Mode (Ring 0 / Supervisor) |
| :--- | :--- | :--- |
| **Privilege Level** | Restricted. Cannot execute privileged CPU instructions. | Unrestricted. Full access to all hardware and memory. |
| **Direct Hardware Access** | Disallowed. Must invoke system calls. | Allowed. Direct I/O ports and device registers access. |
| **Crash Impact** | Process terminates (e.g., Segfault); OS keeps running. | System crashes (Kernel Panic / Blue Screen of Death). |
| **Hardware Bit** | Mode bit is `1`. | Mode bit is `0`. |

---

### System Calls vs. Library Calls

```
Application Code:  printf("Hello");        // Standard C Library call
                         |
Library Level:     write(1, "Hello", 5);   // Invokes system call wrapper
                         |
Hardware Boundary: TRAP / SYSCALL (INT 0x80 / SYSENTER)
                         | Mode bit flips: 1 -> 0
Kernel Level:      sys_write() executed by kernel driver
                         | Mode bit flips: 0 -> 1
Application Code:  resumes execution
```

- **Library Call (e.g., `strcpy()`, `strlen()`, `abs()`):** Executes entirely in user mode. No context switch to kernel mode; ultra-fast.
- **System Call (e.g., `open()`, `read()`, `write()`, `fork()`):** Requires CPU privilege transition (User $\rightarrow$ Kernel $\rightarrow$ User). Involves saving user registers, switching stacks, checking permissions, executing kernel code, and restoring registers.

---

### Interrupts, Traps, and Exceptions

| Type | Nature | Trigger Source | Example |
| :--- | :--- | :--- | :--- |
| **Hardware Interrupt** | Asynchronous | External hardware device | Keyboard press, Network packet arrives, Timer tick |
| **Trap (Software Interrupt)** | Synchronous | User software explicitly requesting OS service | Invoking `read()` or `fork()` via `syscall` instruction |
| **Exception / Fault** | Synchronous | Software error or anomalous condition | Division by zero, Page fault, Segmentation fault |

---

## 2. Process Management

### Process Anatomy & Memory Layout
A **Process** is a program in execution. It is an active entity with its own dedicated virtual address space.

```
High Memory (0xFFFFFFFF)
+--------------------------------------------------+
|                   KERNEL SPACE                   |
|     (Mapped into every process; inaccessible)    |
+--------------------------------------------------+
|               USER STACK (grows down)            |
|   - Function frames, local variables, ret addrs  |
|                         |                        |
|                         v                        |
|                                                  |
|                         ^                        |
|                         |                        |
|               HEAP (grows up via brk/sbrk)       |
|   - Dynamically allocated memory (malloc, new)   |
+--------------------------------------------------+
|               BSS SEGMENT (Uninitialized)        |
|   - Global & static vars initialized to zero     |
+--------------------------------------------------+
|               DATA SEGMENT (Initialized)         |
|   - Global & static vars with explicit values    |
+--------------------------------------------------+
|               TEXT SEGMENT (Code)                |
|   - Read-only executable machine instructions    |
+--------------------------------------------------+
Low Memory (0x00000000)
```

---

### Process Control Block (PCB)
The PCB is the data structure maintained in kernel memory for every active process. When a context switch occurs, the OS saves the process state into its PCB.

**PCB Contents:**
1. **Process ID (PID):** Unique numeric identifier.
2. **Process State:** Ready, Running, Waiting, etc.
3. **Program Counter (PC):** Address of the next machine instruction to execute.
4. **CPU Registers:** Accumulators, index registers, stack pointers (`ESP`/`RSP`).
5. **CPU Scheduling Info:** Priority, scheduling queue pointers.
6. **Memory Management Info:** Page tables, segment tables, base and limit registers.
7. **Accounting Info:** CPU time used, clock ticks elapsed.
8. **I/O Status Info:** List of open file descriptors (`0: stdin`, `1: stdout`, `2: stderr`), open sockets.

---

### Process States & Lifecycle

```
           Admit                 Dispatch (Scheduler)
 [ NEW ] ---------> [ READY ] ------------------------> [ RUNNING ]
                      ^                                      |
                      |          I/O Complete or Event       |
                      +--------------------------------------+ (Timer Interrupt / Yield)
                      |
                      |
                  [ WAITING ] <------------------------------+
                  (BLOCKED)      I/O Request / Wait Event
                      |
                      | Process finishes execution
                      v
                [ TERMINATED ] (Zombie until reaped)
```

1. **New:** The process is being created.
2. **Ready:** The process is loaded in main memory and waiting to be allocated CPU time.
3. **Running:** Instructions are currently being executed by the CPU core.
4. **Waiting (Blocked):** Process cannot run until an external event occurs (e.g., waiting for disk read, network packet, lock).
5. **Terminated:** Process has finished execution; its resources are released, but its exit code remains until read by its parent.

---

### Process Creation: `fork()`, `exec()`, and Copy-On-Write (COW)

#### `fork()`
Creates an exact clone of the calling process:
- Returns `0` to the child process.
- Returns the **Child's PID** to the parent process.
- Returns `-1` if creation fails.

#### Copy-On-Write (COW) — The Optimization Interviewers Love
Historically, `fork()` duplicated the entire physical memory of the parent into the child—a massive waste of CPU cycles and RAM.
* **Modern OS approach (COW):** Parent and child initially share the **exact same physical memory frames**, but mark the page table entries as **read-only**.
* If either parent or child attempts to **write** to a page, a CPU fault triggers. The kernel intercepts this, allocates a **new physical page frame**, copies the data, updates the writing process's page table with write permissions, and resumes execution.

#### `exec()` family (`execve`, `execlp`, etc.)
Replaces the current process image, text, data, heap, and stack with a brand-new executable program. PID remains the same.

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        perror("Fork failed");
        return 1;
    } else if (pid == 0) {
        // Child Process
        printf("Child Process: PID=%d, Parent PID=%d\n", getpid(), getppid());
        char *args[] = {"ls", "-l", NULL};
        execvp("ls", args); // Overwrites child process with 'ls'
    } else {
        // Parent Process
        printf("Parent Process: PID=%d, waiting for child %d\n", getpid(), pid);
        wait(NULL); // Reap child to prevent zombie
        printf("Child finished. Parent exiting.\n");
    }
    return 0;
}
```

---

### Zombie vs. Orphan Processes

```
+--------------------------------------------------------------------------+
|                       ZOMBIE vs. ORPHAN PROCESS                          |
+--------------------------------------------------------------------------+
| Dimension       | Zombie Process               | Orphan Process          |
|-----------------|------------------------------|-------------------------|
| Status of Child | Finished execution (Dead)    | Still running (Alive)   |
| Status of Parent| Alive, but hasn't called     | Terminated/Crashed      |
|                 | wait() to read exit status   | before child finished   |
| Resource Usage  | Minimal: Holds PID and PCB   | Normal process memory   |
|                 | entry in process table       | (CPU, RAM, Stack, Heap) |
| How to Fix      | Parent must call wait()/     | Adopted automatically   |
|                 | waitpid(). If parent dies,   | by init / systemd       |
|                 | init reaps it.               | (PID 1), which reaps it.|
| Command to View | ps aux | grep 'Z'            | ps -ef                  |
+--------------------------------------------------------------------------+
```

---

## 3. Threads & Concurrency

### Process vs. Thread (Deep Comparison)
A **Thread** is the smallest unit of CPU execution within a process (often called a *Lightweight Process*).

```
PROCESS 1 (Heavyweight Isolation)
+-----------------------------------------------------------+
| Code (Text) | Data (Globals) | Heap (Dynamic Memory)       |
+-----------------------------------------------------------+
| Thread 1: Registers, PC, Stack                            |
| Thread 2: Registers, PC, Stack                            |
| Thread 3: Registers, PC, Stack                            |
+-----------------------------------------------------------+
```

| Feature | Process | Thread |
| :--- | :--- | :--- |
| **Definition** | An independent program in execution. | A lightweight path of execution inside a process. |
| **Address Space** | Isolated. Does not share memory with other processes. | Shared. All threads of a process share Text, Data, Heap, and File Descriptors. |
| **Private State** | PCB, Full Address Space, Open Files. | Thread ID, Program Counter, Register Set, Stack Memory. |
| **Creation Cost** | High (memory allocation, page table creation). | Low (shares existing address space). |
| **Context Switch** | Heavy (TLB flush, cache invalidation, page table switch). | Fast (same page table, minimal cache churn). |
| **Inter-communication**| Requires IPC (Pipes, Sockets, Shared Memory). | Direct read/write to shared heap/globals (needs synchronization!). |
| **Crash Safety** | If one process crashes, others are unaffected. | If one thread segfaults, the **entire process crashes**. |

---

### User-Level Threads (ULT) vs. Kernel-Level Threads (KLT)

```
User-Level Threads (Many-to-One):        Kernel-Level Threads (One-to-One):
    [ T1 ]  [ T2 ]  [ T3 ]                  [ T1 ]      [ T2 ]      [ T3 ]
       \      |      /                        |           |           |
    [ User Thread Library ]                   v           v           v
              |                             [ K1 ]      [ K2 ]      [ K3 ]
              v                             (Linux NPTL / Windows Threads)
        [ 1 Kernel Thread ]
```

- **User-Level Threads (ULT):** Managed completely by user-space library (e.g., green threads). The kernel sees only 1 process.
  * *Pros:* Lightning fast creation and context switching (no kernel traps).
  * *Fatal Con:* If one ULT performs a blocking I/O call (e.g., reading disk), the kernel blocks the **entire process**, pausing all other user threads. Cannot utilize multi-core parallelism.
- **Kernel-Level Threads (KLT):** Managed directly by OS kernel (e.g., Linux POSIX threads / pthreads).
  * *Pros:* True multi-core parallelism. If one thread blocks on I/O, other threads continue running on other cores.
  * *Con:* Thread operations require system calls into kernel mode.

---

### Context Switching Mechanics & Overheads
A **Context Switch** is the procedure of saving the state of the currently executing process/thread and loading the state of the next ready process/thread into CPU registers.

#### Step-by-Step Context Switch Sequence:
1. Timer interrupt or blocking syscall fires; CPU enters Kernel Mode.
2. Kernel saves current CPU registers, Program Counter, and Stack Pointer into the current process's **PCB**.
3. Scheduler algorithm selects the next process to run from the Ready Queue.
4. Kernel switches memory mappings by updating the CPU's **CR3 register** (Page Directory Base Register) to point to the new process's Page Table.
5. **TLB is flushed** (on architectures without Address Space IDs / ASIDs).
6. Kernel loads CPU registers from the new process's PCB and sets PC to its saved instruction.
7. Mode bit flips back to User Mode; new process begins running.

#### Why Context Switching is Expensive:
- **Direct Costs:** CPU cycles spent saving/restoring register sets and executing scheduler logic.
- **Indirect Costs (The real killer):** CPU hardware cache (L1/L2/L3) misses and TLB misses. The new process has a "cold cache," slowing down execution until working set data is re-cached.

---

## 4. CPU Scheduling

The CPU Scheduler decides which ready process receives CPU time when a running process terminates, blocks on I/O, or has its time slice expire.

### Preemptive vs. Non-Preemptive

| Criteria | Preemptive Scheduling | Non-Preemptive Scheduling |
| :--- | :--- | :--- |
| **Mechanism** | OS can forcibly interrupt a running process (e.g., timer interrupt or higher priority arrival). | Once CPU is allocated, process holds it until it voluntarily yields or blocks on I/O. |
| **Responsiveness** | High. Essential for modern interactive systems (Linux, Windows, macOS). | Low. A rogue loop can freeze the system. |
| **Overhead** | Higher due to frequent context switching. | Lower context switching overhead. |

---

### Core Algorithms

#### 1. First-Come, First-Served (FCFS) [Non-Preemptive]
- Jobs are executed in order of arrival.
- **Flaw: The Convoy Effect.** A massive CPU-bound process arrives first ($100\text{ms}$ burst); subsequent short I/O-bound processes ($2\text{ms}$ burst) are forced to wait, cratering CPU/device utilization.

#### 2. Shortest Job First (SJF) & Shortest Remaining Time First (SRTF)
- **SJF (Non-preemptive):** Runs the process with the smallest CPU burst time. Proven mathematically optimal for minimizing average waiting time.
- **SRTF (Preemptive version of SJF):** If a new process arrives with a remaining burst shorter than current process's remaining time, preempt current.
- **Flaw:** Impossible to know exact upcoming CPU burst length in advance (must estimate using exponential smoothing). Causes **starvation** for long jobs if short jobs keep arriving.

#### 3. Round Robin (RR) [Preemptive]
- Each process gets a fixed slice of CPU time called a **Time Quantum** ($q$, typically $10\text{ms} - 100\text{ms}$). If not finished, the process is preempted and sent to the tail of the Ready Queue.
- **Quantum Trade-Off:**
  * *If $q$ is extremely large:* RR degrades into FCFS (poor response time).
  * *If $q$ is extremely small:* Context switching overhead dominates, wasting CPU cycles on scheduling rather than useful work.
  * *Golden Rule:* Time quantum should be large relative to context switch time (context switch overhead $< 1\%$ of quantum).

#### 4. Priority Scheduling
- Each process is assigned an integer priority. CPU goes to highest priority.
- **Problem: Starvation (Indefinite Blocking).** Low-priority processes may never execute.
- **Solution: Aging.** Gradually increase the priority of processes that wait in the ready queue for a long time.

#### 5. Multilevel Feedback Queue (MLFQ)
- The industry standard (used in modern OS kernels).
- Uses multiple queues with different priority levels and different time quanta:
  * Top queues have short time quanta (favors interactive, I/O-bound jobs).
  * Bottom queues have long time quanta or run FCFS (for compute-heavy batch jobs).
  * Processes dynamically move between queues: if a process uses its entire time quantum, it is demoted to a lower queue; if it voluntarily yields on I/O, it stays at or is promoted to a higher queue.

---

## 5. Process Synchronization (High Priority)

### Race Conditions & Critical Section Problem
- **Race Condition:** A situation where multiple threads/processes access and manipulate shared data concurrently, and the final outcome depends on the non-deterministic order of execution.
- **Critical Section:** The code block that accesses shared resources (memory, file, socket).

```
Thread A: Read balance ($100) ----> Context Switch!
Thread B: Read balance ($100) ----> Deduct $50 ----> Write balance ($50)
Thread A: Resumes! Deducts $20 ---> Writes balance ($80)
Result: $50 withdrawal by Thread B is completely lost!
```

#### The 3 Mandatory Requirements for a Valid Solution:
1. **Mutual Exclusion:** If process $P_i$ is executing in its critical section, no other processes can be executing in their critical sections.
2. **Progress:** If no process is in its critical section and some wish to enter, only those processes not in their remainder section can participate in deciding who enters next, and this selection cannot be postponed indefinitely.
3. **Bounded Waiting:** There must be a limit on the number of times other processes are allowed to enter their critical sections after a process has requested entry, preventing starvation.

---

### Hardware Primitives
Software synchronization depends on hardware-enforced atomic instructions:
- **Test-and-Set (TAS):** Atomically writes 1 to a memory location and returns its previous value.
- **Compare-and-Swap (CAS):** Atomically compares contents of memory location with an expected value, and modifies it to a new value only if they match. Forms the basis of lock-free data structures and Java's `java.util.concurrent.atomic`.

---

### Mutex vs. Semaphore

```
+--------------------------------------------------------------------------+
|                          MUTEX vs. SEMAPHORE                             |
+--------------------------------------------------------------------------+
| Feature         | Mutex (Mutual Exclusion Object)| Semaphore             |
|-----------------|--------------------------------|-----------------------|
| Nature          | Locking Mechanism              | Signaling Mechanism   |
| Capacity        | Strictly binary (0 or 1)       | Integer counter       |
|                 | (Locked or Unlocked)           | (0 to N resources)    |
| Ownership       | **Has Ownership:** Only the    | **No Ownership:** Any |
|                 | thread that acquired the lock  | thread can signal /   |
|                 | can release it.                | post (V) a semaphore. |
| Primary Purpose | Mutual exclusion for critical  | Resource counting or  |
|                 | sections.                      | inter-thread sync.    |
| Types           | Recursive, Timed, Error-check  | Binary, Counting      |
+--------------------------------------------------------------------------+
```

---

### Spinlock vs. Sleeping Lock

- **Spinlock:** A thread busy-waits in a CPU loop (`while(!lock_acquired);`), continuously checking if the lock has become free.
  * *When to use:* Multicore systems where the lock is held for **ultra-short durations** (a few instructions). Avoids the heavy cost of a thread sleep/wake context switch.
  * *When NOT to use:* Single-core systems (the spinner starves the lock-holder!) or for long-running critical sections (wastes CPU cycles).
- **Sleeping Lock (Mutex / Semaphore):** If lock is unavailable, the thread voluntarily relinquishes CPU and enters the **WAITING / BLOCKED** state. The kernel wakes it up when the lock is freed.
  * *When to use:* Critical sections involving I/O, network calls, or long compute.

---

### Classic Synchronization Problems

#### 1. Producer-Consumer (Bounded Buffer)
- **Problem:** Producer puts items in buffer; Consumer takes them out. Producer must stop if buffer is full; Consumer must stop if buffer is empty.
- **Solution:** 3 Synchronization primitives:
  * `mutex`: Binary lock for protecting buffer insertion/deletion.
  * `emptySlots`: Counting semaphore initialized to buffer capacity $N$.
  * `filledSlots`: Counting semaphore initialized to $0$.

```c
// Producer
wait(emptySlots);
wait(mutex);
insert_item();
signal(mutex);
signal(filledSlots);

// Consumer
wait(filledSlots);
wait(mutex);
remove_item();
signal(mutex);
signal(emptySlots);
```

#### 2. Readers-Writers Problem
- Multiple readers can read simultaneously.
- Only one writer can write at a time (exclusive access). No readers allowed while writing.
- **Variants:** Reader-preference (writers may starve), Writer-preference (readers may starve).

#### 3. Dining Philosophers Problem
- 5 philosophers sit at a round table with 5 chopsticks. Need both left and right chopstick to eat.
- If every philosopher picks up their left chopstick simultaneously $\rightarrow$ **Circular Deadlock!**
- **Solutions:**
  1. Pick up chopsticks only if both are available (atomic pickup).
  2. Asymmetric solution: Odd philosophers pick Left then Right; Even philosophers pick Right then Left.
  3. Allow at most 4 philosophers at the table simultaneously.

---

## 6. Deadlocks

A **Deadlock** is a state where a set of processes are blocked because each process is holding a resource and waiting for another resource acquired by some other process in the cycle.

```
[ Process P1 ] ---- holds ----> ( Resource R1 )
      ^                                |
    waits for                      waits for
      |                                v
( Resource R2 ) <--- holds ---- [ Process P2 ]
```

### The 4 Coffman Conditions
All 4 conditions **MUST hold simultaneously** for a deadlock to occur:

1. **Mutual Exclusion:** At least one resource must be non-shareable (only one process can use it at a time).
2. **Hold and Wait:** A process must be holding at least one resource and actively waiting to acquire additional resources held by others.
3. **No Preemption:** Resources cannot be forcibly confiscated from a process; they can only be released voluntarily after completion.
4. **Circular Wait:** A closed chain of processes exists: $P_0$ waits for $P_1$, $P_1$ waits for $P_2$, $\dots$, $P_n$ waits for $P_0$.

---

### Deadlock Handling Strategies

```
                       DEADLOCK HANDLING STRATEGIES
                                    |
     +-------------------+----------+---------+-------------------+
     |                   |                    |                   |
1. Prevention       2. Avoidance         3. Detection        4. Ignorance
(Break 1 of the     (Banker's Alg;       & Recovery          (Ostrich Algorithm;
4 conditions)       stay in Safe State)  (Detect cycle; kill) do nothing)
```

1. **Deadlock Prevention:** Design protocols that guarantee at least one of the 4 Coffman conditions can never hold.
   * *Break Circular Wait (Most Common in Practice):* Impose a strict global numerical ordering on all resource types ($R_1 < R_2 < R_3$). A process can only request resources in strictly ascending numerical order.
   * *Break Hold and Wait:* Require processes to request all resources at once before beginning execution.
2. **Deadlock Avoidance (Banker's Algorithm):** OS dynamically analyzes every resource request. If granting the request leaves the system in an **Unsafe State** (a state where a sequence of completions is not guaranteed), the request is denied or delayed.
3. **Deadlock Detection & Recovery:** Allow deadlocks to happen. Periodically run cycle-detection algorithms on the Resource Allocation Graph.
   * *Recovery:* Terminate processes (kill all or kill one-by-one) or preempt resources.
4. **Deadlock Ignorance (The Ostrich Algorithm):** Pretend deadlocks never occur. Used by Linux, Windows, and macOS because the runtime overhead of prevention/avoidance exceeds the rare cost of rebooting/killing a frozen process.

---

### Deadlock vs. Livelock vs. Starvation

| Dimension | Deadlock | Livelock | Starvation |
| :--- | :--- | :--- | :--- |
| **Process State** | Blocked / Sleeping (Waiting on an event). | Active / Running (Executing code). | Ready / Blocked. |
| **CPU Usage** | 0% CPU consumed while deadlocked. | 100% CPU consumed (busy looping). | Waiting for schedule or resource. |
| **Analogy** | Two cars meet on a 1-lane bridge; neither backs up; both turn off engines. | Two polite people meet in a hallway; both repeatedly step to the same side to yield. | A low-priority customer at a bank counter who keeps getting cut in line by VIPs. |

---

## 7. Memory Management & Virtual Memory (Crucial)

### Logical vs. Physical Address Space & MMU
- **Logical (Virtual) Address:** Generated by the CPU while executing code. Allows every process to believe it owns a contiguous block of memory starting from address `0x0`.
- **Physical Address:** Actual hardware RAM location.
- **Memory Management Unit (MMU):** Hardware chip between CPU and RAM that translates Virtual Addresses to Physical Addresses on every memory access.

---

### Fragmentation

- **Internal Fragmentation:** Memory allocated to a process is slightly larger than the requested size. The unused portion inside the allocated block is wasted.
  * *Cause:* Fixed-size page frames (e.g., process requests 5 KB; OS allocates two 4 KB pages = 8 KB; 3 KB internal fragment wasted).
- **External Fragmentation:** Total free memory space is sufficient to satisfy a request, but it is divided into small, non-contiguous holes scattered across RAM.
  * *Solution:* Non-contiguous allocation via **Paging**.

---

### Paging Architecture
Paging eliminates external fragmentation by breaking physical memory into fixed-size blocks called **Frames**, and logical memory into blocks of the same size called **Pages** (typically 4 KB).

```
Virtual Address (32-bit):
[ Page Number (p): 20 bits ] [ Page Offset (d): 12 bits ]
             |
             v (Look up in Page Table)
[ Frame Number (f): 20 bits ]
             |
             +--------------------> Physical Address: [ Frame (f) ] [ Offset (d) ]
```

#### Page Table Entry (PTE) Bits:
- **Frame Number:** Physical base address in RAM.
- **Valid/Invalid Bit (Present bit):** `1` if page is present in physical RAM; `0` if swapped out to disk (triggers Page Fault).
- **Dirty (Modified) Bit:** `1` if page was written to. Dictates whether page must be written back to disk on eviction.
- **Referenced (Access) Bit:** Used by page replacement algorithms (Clock/LRU).
- **Protection Bits:** Read, Write, Execute (`r-w-x`).

---

### Translation Lookaside Buffer (TLB)
Because page tables reside in main memory, every memory access requires **two memory accesses**:
1. Access page table to fetch frame number.
2. Access actual data at physical address.

**The Solution: TLB.** A high-speed, associative hardware cache inside the CPU that caches recent virtual-to-physical address translations.
- **Effective Memory Access Time (EMAT):**
  $$\text{EMAT} = \text{Hit Ratio} \times (t_{\text{TLB}} + t_{\text{RAM}}) + (1 - \text{Hit Ratio}) \times (t_{\text{TLB}} + 2 \times t_{\text{RAM}})$$

---

### Demand Paging & Page Fault Handling Sequence

```
1. CPU generates virtual address.
2. MMU checks TLB -> Miss.
3. MMU checks Page Table -> Valid bit is 0! (PAGE FAULT TRAP)
4. CPU drops into Kernel Mode.
5. OS locates desired page in Swap Space on secondary storage (SSD/HDD).
6. OS finds a free physical frame in RAM (or evicts an existing page using LRU).
7. OS issues disk I/O read into the target frame (Current thread is put to SLEEP).
8. Disk controller signals I/O interrupt when read completes.
9. OS updates Page Table entry: sets frame number and sets Valid Bit = 1.
10. CPU restarts the EXACT instruction that caused the fault.
```

---

### Thrashing & Working Set Model
- **Thrashing:** A state where the system spends significantly more time swapping pages in and out of disk than executing user instructions.
  * *Cause:* Degree of multiprogramming is too high. The sum of all processes' **Working Sets** (pages currently in active use) exceeds total physical RAM.
  * *Fix:* Reduce degree of multiprogramming (suspend/swap out some processes).

---

### Page Replacement Algorithms

#### 1. First-In, First-Out (FIFO)
- Replaces the oldest page loaded in memory.
- **Belady's Anomaly:** *A counter-intuitive phenomenon where increasing the number of physical page frames results in an INCREASE in the number of page faults!* (Does not occur in stack-based algorithms like LRU).

#### 2. Least Recently Used (LRU)
- Replaces the page that has not been used for the longest period of time.
- Approximates the optimal algorithm well, but is expensive to implement in pure hardware (requires timestamp registers or list shifting on every memory reference).

#### 3. Clock Algorithm (Second Chance)
- Practical approximation of LRU used in Linux.
- Pages arranged in a circular buffer with a clock hand. Each page has a reference bit (`0` or `1`).
- When looking for a victim:
  * If reference bit is `0`: Select this page as victim.
  * If reference bit is `1`: Clear bit to `0`, advance hand, and check next.

---

## 8. File Systems & Storage

### Linux Inodes Architecture
In Unix/Linux, everything is a file, and every file is represented by an **Inode (Index Node)**.

```
INODE STRUCTURE:
+------------------------------------------------------+
| File Metadata: Permissions (rwxr-xr-x), Owner, Size  |
| Timestamps: atime, mtime, ctime                      |
+------------------------------------------------------+
| 12 Direct Pointers         -----> Direct Data Blocks |
| 1 Single Indirect Pointer  -----> Block of Pointers  |
| 1 Double Indirect Pointer  -----> Pointers to Blocks |
| 1 Triple Indirect Pointer  -----> Nested Pointers    |
+------------------------------------------------------+
```
*Note: The **File Name is NOT stored inside the Inode**; file names are stored inside directory files mapped to Inode numbers!*

---

### Hard Links vs. Soft (Symbolic) Links

| Dimension | Hard Link (`ln file link`) | Soft / Symbolic Link (`ln -s file link`) |
| :--- | :--- | :--- |
| **Inode Number** | Shares the **exact same Inode** as original file. | Gets a **brand new Inode** with file type marked as symlink. |
| **Link Contents** | Direct reference to the original data blocks. | A tiny pointer file containing the path string of the target. |
| **Original File Deleted** | Data is intact! Inode reference count decrements by 1. Data remains until ref count = 0. | Broken Link ("Dangling link"). Cannot access data. |
| **Cross-Filesystem** | Disallowed (Inode numbers are only unique within a single filesystem). | Allowed (points to an arbitrary path string). |
| **Directory Linking** | Disallowed (prevents directory graph cycles).| Allowed. |

---

## 9. Inter-Process Communication (IPC)

Processes have isolated address spaces. To exchange data, they must use OS-provided IPC mechanisms:

1. **Pipes (Anonymous):** Unidirectional data channel for related processes (parent-child). Data is FIFO byte stream.
2. **Named Pipes (FIFOs):** Unidirectional or bidirectional; exists as a filesystem node, allowing unrelated processes to communicate.
3. **Shared Memory:** Fastest IPC method. OS maps a common physical memory region into the address spaces of multiple processes. Once mapped, communication requires **zero syscalls** (processes read/write RAM directly; requires mutexes to synchronize).
4. **Message Queues:** Kernel-managed linked list of messages with priority and type tagging.
5. **Unix Domain Sockets & Network Sockets:** Bidirectional stream/datagram channels. Works across different machines over TCP/IP or locally on the same host.
6. **Signals:** Asynchronous software notifications sent to a process (e.g., `SIGINT` [Ctrl+C], `SIGKILL` [Kill 9], `SIGSEGV`).

---

## 10. Top 20 SDE-1 / Intern Interview Questions & Model Answers

### Q1: What happens during a Context Switch? Why is it considered an expensive operation?
**Answer:** A context switch saves the CPU register context (PC, SP, general registers) of the currently running process into its PCB and loads the saved context of a new process from its PCB, updating the MMU page table pointer (CR3 register). It is expensive due to direct CPU scheduling overhead and indirect performance penalties from cache invalidation (L1/L2/L3 cache misses) and TLB flushing.

---

### Q2: What is the difference between a process and a thread?
**Answer:** A process is an independent program in execution with its own isolated virtual address space, file descriptor table, and memory segments (heap, stack, data, text). A thread is a lightweight execution unit inside a process; all threads of a process share the same text, data, heap, and open files, but maintain their own private Program Counter, register set, and stack.

---

### Q3: Why does `fork()` not copy memory immediately in modern OSes?
**Answer:** Modern operating systems use **Copy-On-Write (COW)**. Calling `fork()` merely duplicates the parent's page table and marks all memory pages as read-only. Physical memory is only copied when either process attempts to write to a page, which triggers a minor page fault where the kernel allocates a new frame. This optimizes performance, especially when `fork()` is immediately followed by `exec()`.

---

### Q4: What is a Zombie process, how is it created, and how do you eliminate it?
**Answer:** A Zombie process is a terminated process whose execution has finished, but its entry remains in the Process Table because its parent has not yet read its exit status via `wait()` or `waitpid()`. To eliminate zombies, the parent must reap them with `wait()`. If the parent has frozen or ignored it, killing the parent causes `init` / `systemd` (PID 1) to adopt the zombie and immediately reap it.

---

### Q5: What is the difference between a Mutex and a Binary Semaphore?
**Answer:** A Mutex is a locking mechanism with **strict ownership**—only the specific thread that acquired the mutex can release it. A Binary Semaphore is a **signaling mechanism** with no concept of ownership—any thread can post/signal a semaphore to wake up another waiting thread.

---

### Q6: What are the 4 necessary conditions for a Deadlock?
**Answer:** The Coffman conditions: (1) Mutual Exclusion, (2) Hold and Wait, (3) No Preemption, and (4) Circular Wait. All 4 must be present simultaneously for a deadlock to exist.

---

### Q7: How does a multi-level page table save memory compared to a single-level page table?
**Answer:** In a 32-bit or 64-bit architecture, a single-level page table would require a massive, contiguous block of RAM allocated upfront for the entire address space, even if most of it is unused. Multi-level paging organizes the table into a tree hierarchy; non-contiguous sub-tables are allocated **on-demand** only for virtual address ranges that the process is actually using.

---

### Q8: What is Belady's Anomaly? Which page replacement algorithm exhibits it?
**Answer:** Belady's Anomaly is the phenomenon where allocating more physical page frames to a process leads to an *increase* in the number of page faults. It occurs in the **FIFO** (First-In, First-Out) page replacement algorithm because FIFO is not a stack-based algorithm.

---

### Q9: What is Thrashing, and how does the OS detect and resolve it?
**Answer:** Thrashing occurs when physical RAM is oversubscribed, causing processes to spend more time swapping pages between disk and memory than executing actual instructions (CPU utilization plummets). The OS detects it when page fault rates spike while CPU utilization drops. It resolves thrashing by reducing the degree of multiprogramming (suspending processes and swapping them out completely).

---

### Q10: What is the difference between a Hard Link and a Soft (Symbolic) Link?
**Answer:** A Hard Link is an additional directory entry pointing to the **same underlying Inode**; deleting the original filename does not destroy the file data until all hard links are deleted (reference count drops to zero). A Soft Link is a distinct, independent file with its own Inode containing a text path string pointing to the target; deleting the target leaves the soft link broken ("dangling").

---

### Q11: Explain the difference between user mode and kernel mode. Why is this separation necessary?
**Answer:** The separation protects system integrity. User mode (Ring 3) executes unprivileged code without direct access to hardware or raw physical memory. Kernel mode (Ring 0) executes supervisor code with unrestricted hardware access. Without this distinction, a single buggy application or malicious pointer could crash the entire operating system or spy on other processes.

---

### Q12: What is the Translation Lookaside Buffer (TLB)? What happens to the TLB during a context switch?
**Answer:** The TLB is an on-chip associative hardware cache that stores recent virtual-to-physical address translations. During a process context switch, the virtual address space changes, requiring the OS to flush the TLB (or invalidate entries without matching Address Space IDs [ASIDs]), causing a transient performance dip as the new process repopulates it.

---

### Q13: Can a deadlock occur with only a single process?
**Answer:** No. By definition, a deadlock requires a circular wait condition between multiple processes or a process waiting for a resource that only another action can produce. A single thread deadlocking on its own non-recursive mutex is a self-deadlock bug, but classical OS deadlock involves a set of processes.

---

### Q14: What is the difference between Starvation and Deadlock?
**Answer:** In a Deadlock, two or more processes are permanently blocked waiting on resources held by each other; none can make progress, and CPU usage is 0%. In Starvation, resources are continuously granted to higher-priority processes, so a low-priority process waits indefinitely, even though the system as a whole continues to do active work.

---

### Q15: Why is Round Robin scheduling sensitive to time quantum size?
**Answer:** If the time quantum is too large, Round Robin degenerates into FCFS, hurting response times for interactive jobs. If the time quantum is too small, context switching overhead consumes a significant fraction of CPU time, degrading total system throughput.

---

### Q16: What is a Spinlock, and when would an engineer choose it over a Mutex?
**Answer:** A spinlock executes a tight CPU polling loop (`while(!lock)`) until the lock is released. An engineer chooses a spinlock on multi-core systems when the critical section is extremely short (e.g., updating a counter in a device driver), because the cost of spinning for a few nanoseconds is lower than the cost of two context switches (putting the thread to sleep and waking it up).

---

### Q17: What is the difference between a trap, an interrupt, and an exception?
**Answer:** An interrupt is an **asynchronous hardware signal** generated by external devices (e.g., keyboard, timer, NIC). A trap is a **synchronous software-initiated invocation** of the kernel via a CPU instruction (used for system calls). An exception is a **synchronous hardware-detected fault or error** caused by software execution (e.g., division by zero, segfault, page fault).

---

### Q18: What is Inode in Unix/Linux? Does it store the file name?
**Answer:** An Inode (Index Node) is a filesystem data structure that stores all metadata about a file (permissions, owner, size, timestamps, data block pointers). **It does NOT store the file name.** The file name is stored in a directory entry (dentry), which maps the name string to an Inode number.

---

### Q19: What is the difference between shared memory and message passing IPC?
**Answer:** Shared memory maps a common physical memory region into the address spaces of participating processes; data exchange occurs at memory bus speeds without system calls, but requires developer synchronization (locks). Message passing uses kernel system calls (`send()`, `recv()`) to exchange packets; it is slower due to kernel copying, but safer and suitable for distributed systems.

---

### Q20: Explain the complete sequence of events during a Page Fault.
**Answer:**
1. CPU references a virtual address whose Page Table entry has Present Bit = 0.
2. Hardware triggers a Page Fault Exception, trapping into kernel mode.
3. OS checks if the memory access was valid (if invalid $\rightarrow$ Segmentation Fault).
4. OS finds a free physical frame in RAM (or evicts an existing page using a page replacement algorithm).
5. OS schedules disk I/O to read the page from swap space/storage into the target frame.
6. The faulting process is moved to the **Waiting/Blocked** state.
7. Disk completes read and sends an I/O interrupt.
8. OS updates the process's Page Table (sets frame number, sets Valid Bit = 1).
9. Process moves to the **Ready** queue.
10. When scheduled, the OS restarts the exact instruction that triggered the page fault.