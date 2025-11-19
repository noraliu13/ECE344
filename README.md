# Table of Contents

**Kernels**
- [Lecture 1: Operating Systems](#lecture-1-operating-systems)
- [Lecture 2: Kernels](#lecture-2-kernels)
- [Lecture 3: Libraries](#lecture-3-libraries)

**Processes**
- [Lecture 4: Process Creation](#lecture-4-process-creation)
- [Lecture 5: Process Management](#lecture-5-process-management)
- [Lecture 6: Basic IPC](#lecture-6-basic-ipc)
- [Lecture 7: Process Practice](#lecture-7-process-practice)
- [Lecture 8: Subprocess](#lecture-8-subprocess)

**Scheduling**
- [Lecture 9: Basic Scheduling](#lecture-9-basic-scheduling)
- [Lecture 10: Advanced Scheduling](#lecture-10-advanced-scheduling)
- [Lecture 11: Virtual Memory](#lecture-11-virtual-memory)
- [Lecture 12: Page Tables](#lecture-12-page-tables)
- [Lecture 13: Page Table Implementation](#lecture-13-page-table-implementation)

### Midterm Break

**Threads**
- [Lecture 16: Threads](#lecture-16-threads)
- [Lecture 17: Thread Implementation](#lecture-17-thread-implementation)

**Locks**
- [Lecture 21: Locks](#lecture-21-locks)
- [Lecture 22: Locks Implementation](#lecture-22-locks-implementation)
- [Lecture 23: Semaphores](#lecture-23-semaphores)


# Lecture 1: Operating Systems

## 📌 Core OS Themes  

### 1. Virtualization  
Making **one resource look like many**.  
- Example: One CPU → many processes appear to run “at once.”  
- Example: One memory → each process sees its own private memory.  

### 2. Concurrency  
Multiple things happening at once (real or apparent).  
- Example: Two programs running on one CPU, switching back and forth.  

### 3. Persistence  
Information survives **power loss**.  
- OS manages disks & filesystems → ensures data is not lost when system reboots.  



### 💡 Computer Science Trick: **Indirection**  
- Problems can often be solved with another **layer of abstraction**.  
- OS rarely gives you hardware directly — instead, it gives you **handles**:  
  - Process → handle to CPU + memory  
  - File descriptor → handle to file  
  - Virtual address → handle to physical memory  



## 📌 First Abstraction: The Process  

### Program vs. Process  
- **Program** = file on disk (instructions + data).  
- **Process** = a running instance of a program.  



### Process Components  
Each process is given by the OS:  
- **Virtual Registers** → private copy of CPU registers.  
- **Virtual Memory** → private address space containing:  
  - **Stack** (local variables)  
  - **Heap** (dynamic memory)  
  - **Globals** (global variables + static data)  
  - **Code** (instructions being executed)  



## 🔍 Example from Lecture  

Two copies of the same program are run:  
- Both print **local** and **global** variables.  
- Observation:  
  - Local variables naturally independent.  
  - Global variables **also independent**, even if they appear at the same address.  

✅ Proof: Each process gets its **own virtual address space**.  



## 🖼 Diagram: Process Virtualization  

```text
          ┌──────────────────────────────┐
          │          Process A           │
          │ ┌─────────┐  ┌───────────┐  │
Virtual    │ │ Stack   │  │  Globals  │  │
Memory     │ ├─────────┤  ├───────────┤  │
           │ │ Heap    │  │  Code     │  │
          └─┴─────────┴──┴───────────┴──┘
               ↑  Private to Process A
               (address 0x1234 here may mean 
                something totally different 
                in Process B)

          ┌──────────────────────────────┐
          │          Process B           │
          │ ┌─────────┐  ┌───────────┐  │
Virtual    │ │ Stack   │  │  Globals  │  │
Memory     │ ├─────────┤  ├───────────┤  │
           │ │ Heap    │  │  Code     │  │
          └─┴─────────┴──┴───────────┴──┘

```

---
 
# Lecture 2: Kernels

## 🌐 Introduction
- Explore what happens when running **Hello World**.
- Learn about **kernels**, **system calls**, and how programs interact with the OS.
- Some of this is exam-relevant, some is just fun background.

## ⚙️ Assembly & Instruction Set Architectures (ISA)
- **Machine Code** = actual binary numbers CPU executes.  
- **Assembly** = human-readable version of machine code.  

### Major ISAs Today
- **x86-64** → desktops, servers, Windows laptops.  
- **ARM64 (AArch64)** → phones, tablets, Apple M-series Macs, Snapdragon laptops.  
- **RISC-V** → open-source ISA, popular in academia/research.  

📌 In this course:
- You **won’t write** assembly.
- You **may need to read** small snippets.

## 🖥️ Processes & File Descriptors
- **Process** = running program (with its own memory + registers).  
- **File Descriptor (FD)** = integer handle used for I/O.  
- FDs can point to:
  - File
  - Terminal
  - Network socket
  - Pipe

### Standard FDs
```text
0 → stdin   (input)
1 → stdout  (output)
2 → stderr  (errors/logs)
```

## 🔧 System Calls

- A **system call (syscall)** is a request from a user program to the OS kernel.  
- They **look like functions** in C but actually invoke kernel services.

### Minimal Hello World
Two syscalls are required:

1. **`write(fd, buffer, count)`**
   - `fd` = file descriptor (1 = stdout)  
   - `buffer` = pointer to characters  
   - `count` = length of string  
   - returns number of bytes written  

2. **`exit_group(status)`**
   - Ends the process  
   - Usually called when `main` returns  

### Example (low-level C-like pseudocode)
```c
void _start() {
    write(1, "Hello World\n", 12);
    exit_group(0);
}
```

## 📚 API vs ABI

- **API (Application Programming Interface):**  
  Describes functions at a high level (names, arguments, return values).  

- **ABI (Application Binary Interface):**  
  Defines low-level details of how functions are invoked (register usage, stack layout, calling conventions).  

➡️ System calls rely on the **ABI**, not just the API.  

## 🛠️ How Syscalls Work

- Syscalls are **not normal function calls**.  
- Instead, they use a **special CPU instruction** to transition into kernel mode.  

### On ARM64
- Syscall number → stored in register `X8`  
- Arguments → placed in `X0` … `X5`  
- Instruction `SVC` (Supervisor Call) → traps into the kernel  

### Example Flow
```text
X8 = syscall_number
X0 = arg0
X1 = arg1
...
SVC → trap into kernel
```
---

# Lecture 3: Libraries

## 🔧 System Calls Recap
- A **system call** is a function that runs in **kernel mode** instead of **user mode**.  
- They allow a program to request services from the kernel (e.g., access hardware).  
- Execution flow:  

## 🔧 System Call Flow
A system call transitions a program from **user mode** into **kernel mode** to request OS services.  

### Execution Flow
```text
Program (User Mode)
   ↓
System Call
   ↓
Switch to Kernel Mode
   ↓
Perform Operation
   ↓
Return to User Mode
```

## 🖥️ What Makes an Operating System?

- **Kernel**: The core of the OS that manages processes, memory, devices, and hardware access.  
- **Libraries**: Provide reusable functions and programming interfaces.  
- **Utilities**: User-facing tools and programs.  

### Formula
```text
Kernel + Libraries + Utilities = Operating System
```

### 🌍 Examples of Operating Systems
- **Apple platforms** (iOS, macOS, iPadOS, etc.) share the **same kernel**, but differ in libraries and user applications.  
- **Android vs Debian**: both rely on the **Linux kernel**, but feel very different because they use different libraries and utilities.  
- When people say *Linux*, they usually mean **GNU/Linux**:  
  - **Linux** → the kernel  
  - **GNU** → the C standard library + essential utilities  

 

## ⚙️ Compilation and Linking
- Each `.c` source file is compiled into an **object file** (`.o`) containing machine code.  
- The **linker** combines multiple `.o` files into a single executable.  

### Example
```text
main.c → main.o
util.c → util.o
foo.c  → foo.o
bar.c  → bar.o
     -
Linker → final executable
```

## 📚 Libraries

A **library** is a collection of precompiled object files that can be shared and reused across multiple programs.  
They help avoid rewriting common functionality and make programs more modular.  

 

### 🔒 Static Libraries (`.a`)
- Library code is **copied directly** into the executable during linking.  
- Produces a **self-contained** binary (no external dependencies at runtime).  

**Drawbacks**:  
- Larger executable size  
- Updating the library requires **relinking** every program  

 

### 🔗 Dynamic Libraries
- Linux: `.so`  
- macOS: `.dylib`  
- Windows: `.dll`  
- Code is **not embedded** into the executable. Instead, the program stores a **reference** to the library.  
- At runtime, the **dynamic linker** loads the required library.  

**Advantages**:  
- Smaller executables  
- Easy to update shared libraries without recompilation

---

# Lecture 4: Process Creation

## 1. 🔄 What is a Process?
When we run a program, it becomes a **process**.  
A process contains:
- **Virtual registers** (program counter, stack pointer, etc.)  
- **Virtual memory** (Heap, Stack, Globals)  
- **Open file descriptors** (array of ints managed by the kernel)  

## 2. 🖥️ Kernel vs User Space
- **Kernel Mode**
  - Runs privileged instructions  
  - Manages processes, files, memory, scheduling  

- **User Mode**
  - Runs regular applications  
  - Can only interact with the kernel via **system calls**  

- **Libraries**
  - Provide higher-level functions  
  - Often call system calls on behalf of programs  

### Diagram
```text
Application (User Mode)
   |
   | function calls
   v
Library
   |
   | system calls
   v
Kernel (Kernel Mode)
```

## 3. 📂 File Descriptors

Every process maintains its own **table of file descriptors (FDs)**.  
These are small integer indexes that map to open files, devices, or sockets managed by the kernel.  

### Standard File Descriptors
```text
0 → stdin   (standard input)
1 → stdout  (standard output)
2 → stderr  (standard error)
```
--- 

# Lecture 5: Process Management

## 1. Introduction

- In the previous lecture, we learned **how to create a process**.
- Today: **managing processes**, understanding the **parent-child relationship**, and Linux-specific **process states**.

## 2. Linux Process States

- Linux has **five main states** for processes:

1. **Running (Runnable)**  
   - Process is currently running or ready to run.
   
2. **Interruptible Sleep (Sleeping)**  
   - Process has given up the CPU (e.g., waiting after a system call).  
   - Can be woken up if needed.

3. **Uninterruptible Sleep (Blocked)**  
   - Process is blocked and cannot run, typically waiting for I/O.

4. **Stopped**  
   - Process is explicitly paused. Can be resumed later.

5. **Zombie (Zed) State**  
   - Process has **terminated**, but its **exit status has not been acknowledged** by the parent.  
   - Exists in the process table but cannot run.


## 3. Unix Process Hierarchy

- When a computer boots:
  - Kernel runs a **single process** (PID 1, often `init` or `systemd`).
  - PID 1 creates all other processes either **directly or indirectly**.
- Parent-child relationship:
  - Strict in Unix.
  - Parent is responsible for its child’s termination acknowledgment.
  
### Example Process Tree

``` text
init/systemd (PID 1)
├─ journalD
├─ udev
├─ systemd-user
│ └─ gnome-shell
│ └─ Firefox
│ └─ Firefox tabs (each tab = separate process)
└─ terminal → shell → htop
```

## 2. Process IDs (PID)

- Every process gets a **unique PID** that does **not change** during its lifetime.
- Maximum PID in Linux: typically **32,000**, configurable.
- PID 0: reserved/invalid, used only as a return value in `fork()`.

## 3. Process States (Linux)

1. **Running (Runnable)**: Process is executing or ready to execute.
2. **Interruptible Sleep**: Process is blocked but can be awakened.
3. **Uninterruptible Sleep**: Process is blocked waiting for I/O; cannot run.
4. **Stopped**: Process explicitly paused.
5. **Zombie**: Process terminated but still has a PID until parent reads its exit status.

## 4. Zombies and Orphans

### 4.1 Zombie Processes
- **Definition**: Process terminated but **parent has not called `wait()`**.
- Still occupies:
  - Process ID
  - Process Control Block
- **Example**:
```c
if (fork() == 0) {
    sleep(2);
    exit(0);
} else {
    // Parent does not call wait()
    sleep(3); 
    // Child becomes a zombie after 2 seconds
}
```

## 4.2 Orphan Processes

- **Definition**: Occurs when a parent process exits while its child is still running.  
- **Reparenting**: Kernel automatically assigns the orphaned child to a new parent (default: PID 1, `init`/`systemd`).  
- **Behavior**: Orphaned processes continue to run normally and are eventually cleaned up by the system.

## 5. `wait()` System Call

**Purpose**: Allows a parent process to **acknowledge the termination of its child** and collect its exit status.

**Syntax**:
```c
pid_t wait(int *status);
```

## `wait()` System Call

**Parameters:**  
- `status` → pointer to an integer where the child's exit information will be stored.

**Return Value:**  
- PID of terminated child  
- `-1` on error  
- `0` for non-blocking calls

**Behavior:**  
- Blocks until **any child** terminates.

**Macros:**  
- `WIFEXITED(status)` → Returns true if the child exited normally  
- `WEXITSTATUS(status)` → Returns the child's exit code

**Example:**
```c
pid_t pid = fork();
if (pid == 0) { 
    // Child process
    sleep(2);
    exit(42);
} else {
    // Parent process
    int wstatus;
    pid_t cpid = wait(&wstatus);
    if (WIFEXITED(wstatus)) {
        printf("Child PID %d exited with status %d\n", cpid, WEXITSTATUS(wstatus));
    }
}
```

## Forking Multiple Processes

- Each `fork()` creates a **duplicate of the calling process**.  
- **Return values distinguish parent and child:**  
  - `0` → child process  
  - `>0` → parent process (PID of child)  
- Multiple forks can create **many processes**.  
- **Note:** Variables are copied; be careful with shared data.

## Process Behavior Examples

### Zombie Example
- Parent does **not call `wait()`**, child sleeps 2 seconds.  
- After the child exits, it becomes a **zombie** until the parent acknowledges termination.

### Orphan Example
- Parent sleeps 1 second, child sleeps 2 seconds.  
- Parent exits **before child**, making the child an **orphan**.  
- Kernel reassigns the orphaned child to **PID 1 (`systemd`)** for cleanup.

## Process Table Limits

- **Zombies and orphans consume PIDs**.  
- PID exhaustion can occur due to:  
  - **Infinite fork loops** → prevents new processes from being created  
  - **Zombie processes** → occupy PID slots until cleaned up

## Key Takeaways

- `fork()` creates new processes; return values distinguish parent and child.  
- Each process has a **unique PID** and **independent address space**.  
- **Zombies**: terminated but **unacknowledged by parent**.  
- **Orphans**: parent died, **reparented to PID 1**.  
- Always use `wait()` to **properly clean up child processes**.

---

# Lecture 6: Basic IPC

## Introduction to IPC
- IPC (Interprocess Communication) is necessary because processes are independent.
- Even printing "Hello World" involves IPC (process writes to the terminal).
- IPC is just transferring bytes between processes.
- Reading and writing files is a form of IPC:
  - One process writes to a file.
  - Another process reads from the file.
- Processes on different machines can communicate via IPC (e.g., internet).

## Basic Read/Write Example

```c
#define BUFFER_SIZE 4096

char buffer[BUFFER_SIZE];
int bytes_read;

while ((bytes_read = read(0, buffer, sizeof(buffer))) > 0) {
    int bytes_written = write(1, buffer, bytes_read);
    assert(bytes_read == bytes_written);
}

if (bytes_read == -1) {
    perror("read error");
}
```

## Reading and Writing

- Reads from **standard input** (`fd = 0`) and writes to **standard output** (`fd = 1`).
- `read()` is **blocking**: waits until input is available.
- End of file is signaled by `read()` returning `0`.
- Errors are indicated by `read()` returning `-1`.



## Standard File Descriptors

- `0` → standard input (stdin)  
- `1` → standard output (stdout)  
- `2` → standard error (stderr)  

### Notes:

- File descriptors can be closed and reassigned:  
  Closing `fd 0` and opening a file assigns it to `fd 0`.
- Shell can redirect standard file descriptors before running a program:  
  ```bash
  ./program < file.txt
  ```
## Pipes

- Pipes connect the **stdout** of one process to the **stdin** of another.  
- Example:
  ```bash
  echo "Hello" | ./read_write_example
  ```
## Signals

- Signals are a form of **IPC** that interrupts a process.
- Default signal handlers:
  - Ignore the signal
  - Terminate the process
- Example: Pressing **Ctrl+C** sends `SIGINT` (default: terminate)

### Common Signals

| Signal   | Number | Description                     |
|-|--||
| SIGINT   | 2      | Keyboard interrupt              |
| SIGKILL  | 9      | Force terminate                 |
| SIGTERM  | 15     | Polite terminate                |
| SIGSEGV  | 11     | Memory access violation         |



## Handling Signals

- Register custom signal handlers with `sigaction`.

```c
void handle_signal(int signal_num) {
    printf("Received signal %d\n", signal_num);
}

struct sigaction sa;
sa.sa_handler = handle_signal;
sigaction(SIGINT, &sa, NULL);
sigaction(SIGTERM, &sa, NULL);
```

- Interrupted system calls return `-1` and set `errno` to `EINTR`.
- Can handle by **retrying the system call**.



## Killing Processes

- `kill <pid>` → sends `SIGTERM` by default  
- `kill -9 <pid>` → sends `SIGKILL` (cannot be ignored)  
- Only the process owner or root can kill processes.  
- Root can kill any process, but **PID 1 (systemd)** is protected.



## Non-blocking Wait

- `waitpid(pid, &status, WNOHANG)` → returns immediately if child has not exited.

### Example

```c
while (waitpid(-1, &status, WNOHANG) == 0) {
    sleep(1); // reduce wasted CPU cycles
}
```

## Tradeoff

- **Longer sleep** → slower reaction  
- **Shorter sleep** → higher CPU usage



## Signal Handling with Children

- The kernel sends `SIGCHLD` when a child process terminates.  
- Parent can register a handler:

```c
void sigchld_handler(int signal_num) {
    int status;
    wait(&status);
}
```

- Ensures child processes don’t become **zombies**.
- Signal handlers can be nested; careful **resource management** is needed (e.g., closing files).

## Summary

- IPC enables communication between independent processes.
- Standard file descriptors provide easy read/write access.
- Pipes allow chaining processes.
- Signals allow asynchronous notifications.
- Signal handlers can handle or ignore signals.
- Non-blocking waits and signal-driven waits provide flexibility in process management.
- Proper cleanup is essential to avoid **zombies** and **resource leaks**.

---

# Lecture 7: Process Practice

## 1. Overview
- Today’s focus: process creation, zombies/orphans, signals, and interprocess communication (IPC) via pipes.
- Relevant for **Lab 2**: common mistakes can cause hours of debugging.
- System call layer: we are using, not implementing, OS internals.

## 2. Processes and `init`
- `init` process (from Unix):
  - Launches shell (`sh`) via `fork()` and `exec()`.
  - Uses an infinite loop to wait for child processes.
  - Cleans up zombies/orphans.
- Older OS: **uni-programming** - only one process at a time.
- Modern OS: **multi-programming**, parallel/concurrent execution using CPU cores.
- **Scheduler**:
  - Pauses current process, saves its state (registers, memory).
  - Chooses next process to run.
  - Loads its state and executes.

### Context Switching
- Mechanism to switch between processes.
- Saves current registers, loads next process registers.
- Overhead exists; optimized by OS (skip saving unused registers).

### Multitasking Styles
1. **Cooperative**: process must yield CPU voluntarily.
2. **Preemptive (True multitasking)**: OS can take CPU control, using:
   - Time slices
   - Interrupts

## 3. Pipes
- System call: `pipe(int fds[2])`
  - Returns `0` on success, `-1` on failure.
  - `fds[0]` → read end
  - `fds[1]` → write end
- Pipe: **one-way communication channel** between processes.
- Managed by kernel, accessed only via file descriptors.

### Key Points
- Parent and child processes **share file descriptors** after `fork()`.
- Important: **close unused ends** to avoid blocking reads/writes.

## 4. Example: Parent → Child Communication

### Child
```c
close(fds[1]); // Close write end
char buffer[496];
int bytes_read = read(fds[0], buffer, sizeof(buffer));
if (bytes_read > 0) {
    printf("%.*s\n", bytes_read, buffer);
}
close(fds[0]);
```

## Parent 
```c
close(fds[0]);          // Close read end
char *msg = "Howdy child";
write(fds[1], msg, strlen(msg));
close(fds[1]);
```

# Pipe Behavior Notes

## Pipe Behavior
- `read()` is **blocking**: waits for data unless all write ends are closed.
- Closing **unused file descriptors** ensures `read()` returns `0` when no more data is possible.
- **Standard file descriptors**:
  - `0` → stdin
  - `1` → stdout
  - `2` → stderr
- Closing `stdout` (`1`) disables `printf()` output.

## Important Tips
- **Always close file descriptors** as soon as they are no longer needed to prevent blocking or resource leaks.
- **Memory management**:
  - Allocated **before `fork()`** → exists in both parent and child; free in both.
  - Allocated **after `fork()`** → exists only in that process; free there.
- **Pipes must be created before `fork()`** to share between parent and child.
- **Multiple reads advance the pipe buffer**; data is **not repeated** unless explicitly repositioned.

## Common Pitfalls
- Forgetting to close **write end in child** → `read()` blocks forever.
- Forgetting to close **read end in parent** → `write()` may succeed but no reader exists.
- Creating a pipe **after `fork()`** → parent and child have independent pipes → no communication.
- Using **multiple writers/readers** → execution order affects output (race conditions).

## Fun / Advanced Notes
- `&` in the shell → runs processes in the background.
- Recursive pipe forks (`while true; fork`) → can crash the system.
- Multiple processes reading/writing → race conditions possible.
- `printf()` may fail if `stdout` is closed.

---

# Lecture 8: Subprocess

## Lecture Overview
- Topic: Completing Lab 2 – sending and receiving data from a process.
- Goals:
  1. Create a new process that launches a command-line argument.
  2. Send the string `"testing\n"` to that process.
  3. Receive and read any data printed to its standard output.

 

## APIs Covered
### `execve` / `execvp`
- `execve`: Starts running another program, replacing the current process.
- `execvp`: Convenient wrapper over `execve`.
  - Finds program in `PATH`, no need for full path.
  - Pass C strings directly; terminate with `NULL`.
  - If successful, does **not return**; on failure, returns `-1` and sets `errno`.

### `dup` / `dup2`
- `dup(oldfd)` → returns new FD pointing to the same resource.
- `dup2(oldfd, newfd)` → makes `newfd` point to same resource as `oldfd`.
  - Closes `newfd` first if already in use.
  - Useful to manipulate standard file descriptors (`stdin`, `stdout`, `stderr`) before `exec`.

 

## File Descriptors
- Standard FDs:
  - `0` → stdin
  - `1` → stdout
  - `2` → stderr
- Closing `stdout` disables `printf()` output.
- Pipes are **one-way communication channels**; need one pipe per direction.

 

## Pipe Setup & Forking
1. **Create pipe before fork** to share between parent and child.
2. **Fork**:
   - Parent: `fork()` returns child PID.
   - Child: `fork()` returns `0`.
3. Each process initially has the same FDs open (0, 1, 2, plus pipe FDs).

### Reading from a Pipe
- `read()` is **blocking**: waits until all write ends are closed and data is consumed.
- Must close unused pipe ends to prevent blocking.

### Sending Data via Pipe
- Write data to the pipe using `write(pipe_fd, buffer, size)`.
- Read from the pipe with `read(pipe_fd, buffer, size)`.

 

## Example: Capturing `uname` Output
- Parent creates a pipe (`out_pipe`) and forks.
- Child:
  - Redirects `stdout` to the write end of `out_pipe` via `dup2`.
  - Closes unused pipe FDs.
  - Executes `uname` via `execvp`.
- Parent:
  - Reads from the read end of `out_pipe`.
  - Prints output (e.g., `"Linux"`).

**Important**: Always close unused FDs to avoid blocking or resource leaks.

 

## Bidirectional Communication
- Use two pipes:
  1. `out_pipe`: child writes, parent reads.
  2. `in_pipe`: parent writes, child reads.
- Redirect FDs using `dup2`:
  - Child `stdout` → `out_pipe` write end.
  - Child `stdin` → `in_pipe` read end.
- Parent writes data to `in_pipe` and reads from `out_pipe`.

 

## Example: Sending `"testing\n"` to a child running `cat`
1. Parent writes `"testing\n"` to `in_pipe`.
2. Child reads from `stdin` (redirected to `in_pipe`) and writes to `stdout` (redirected to `out_pipe`).
3. Parent reads from `out_pipe` and sees `"testing\n"`.

## Best Practices
- **Close file descriptors** as soon as you are done using them:
  - Parent: close read end of `in_pipe` after writing.
  - Parent: close write end of `out_pipe` if not used.
  - Child: close unused ends of pipes before `exec`.
- **Wait for child process** using `wait(pid, &status, 0)` to prevent zombies/orphans.
- Use `assert` to check child exit status:
  ```c
  int wstatus;
  wait(pid, &wstatus, 0);
  assert(WIFEXITED(wstatus));
  assert(WEXITSTATUS(wstatus) == 0);
  ```

## Common Pitfalls
- Not closing the **write end** in the parent or **read end** in the child → can cause reads/writes to block.  
- Leaving pipes open → child may hang waiting for EOF (e.g., with `cat`).  
- Overwriting `stderr` unnecessarily → keep it available for error messages.  

## Lab 2 Steps Summary
1. Create two pipes: `in_pipe` and `out_pipe`.  
2. Fork a new process.  
3. **Child process**:  
   - Redirect `stdin` and `stdout` using `dup2`.  
   - Close any unused pipe file descriptors.  
   - Execute the target program with `execvp`.  
4. **Parent process**:  
   - Write data to `in_pipe`.  
   - Read output from `out_pipe`.  
   - Close any used file descriptors.  
   - Wait for the child to finish.  

## Key Takeaways
- Proper management of file descriptors is essential for inter-process communication.  
- Pipes provide a straightforward way for data transfer between parent and child.  
- `dup2` enables redirection of standard I/O to pipes.  
- Always wait for child termination to avoid creating zombies or orphaned processes.  


---

# Lecture 9: Basic Scheduling 

## Overview
CPU scheduling determines **which process runs when**.  
This topic follows process management (creation, termination, adoption, etc.) and focuses on CPU resource sharing.

## Types of Resources

### Preemptable Resources
- Can be taken away and reassigned.
- Example: CPU.
- Shared via **scheduling**.

### Non-Preemptable Resources
- Cannot be taken without consent.
- Examples: Disk, memory.
- Shared via **allocation/deallocation**.

## Dispatcher vs Scheduler

- **Dispatcher**: Performs the actual context switch (saves and restores process state).
- **Scheduler**: Decides **which process runs next**.

Schedulers run when:
- A process terminates.
- A process blocks.
- (Preemptive) The OS interrupts and selects another process.

## Scheduling Evaluation Metrics

- Minimize **waiting time**  
- Minimize **response time**  
- Maximize **CPU utilization**  
- Maximize **throughput**  
- Ensure **fairness**

> Note: Fairness can conflict with other metrics (e.g., minimizing waiting time).

## Scheduling Algorithms

### 1. First Come First Serve (FCFS)
- Non-preemptive.
- Processes are executed in order of arrival (FIFO).
- Simple to implement.
- Can cause **long waiting times** for short jobs behind long ones (**convoy effect**).

### 2. Shortest Job First (SJF)
- Non-preemptive.
- Picks the process with the shortest CPU burst next.
- **Optimal** for minimizing average waiting time.
- Requires knowledge of burst times → **impractical**.
- Can lead to **starvation** (long jobs never run).

### 3. Shortest Remaining Time First (SRTF)
- Preemptive version of SJF.
- Preempts running process if a new one has shorter remaining time.
- Optimal for average waiting time.
- Still **impractical** and prone to **starvation**.

### 4. Round Robin (RR)
- **Preemptive and fair.**
- Each process gets a fixed **time quantum**.
- If unfinished, it moves to the back of the ready queue.
- Context switches occur between processes.

**Tradeoffs:**
- **Short quantum** → more context switches (higher overhead).
- **Long quantum** → less fairness (behaves like FCFS).

## Key Takeaways

- **FCFS:** Simple but unfair to short jobs.  
- **SJF / SRTF:** Optimal but impractical; risk of starvation.  
- **Round Robin:** Fair and widely used.  
- **Starvation:** Common in SJF and SRT

---

# Lecture 10: Advanced Scheduling 

## 1. Review: Basic Scheduling
**Scheduling** determines the order in which processes run.

Examples:
- **Round Robin (RR)**
- **First-Come, First-Serve (FCFS)**

## 2. Priorities

### Priority Scheduling
- Each process is assigned a **priority**.
- Higher-priority processes run before lower-priority ones.
- Processes of equal priority can use **Round Robin** for fairness.
- Can be **preemptive** or **non-preemptive**.

**Linux convention:**
- Lower number = higher priority.
- Range: `-20` (highest) → `19` (lowest).

### Starvation
- A low-priority process might never run if high-priority ones keep arriving.
- **Solution:** *Priority aging* — gradually increase a waiting process’s priority over time.

## 3. Priority Inversion
Occurs when a **high-priority process depends** on a **low-priority** one holding a resource.

### Example
- Low-priority process holds a lock.
- High-priority process needs that lock → it must wait.
- Medium-priority processes can still run and delay completion.

### Solution: Priority Inheritance
- The low-priority process **temporarily inherits** the higher priority.
- Prevents unnecessary blocking.
- Can chain through multiple dependent processes.

## 4. Foreground vs Background Processes
- **Foreground:** User-interactive; needs *low response time*.
- **Background:** Non-interactive; optimize for *throughput*.

Possible to use **different scheduling algorithms**:
- Foreground → Round Robin
- Background → FCFS

Can mix with **multi-level queues**:
- Foreground queue
- Background queue  
→ Each queue can use different algorithms or static priorities.

## 5. Multiprocessor Scheduling

### Symmetric Multiprocessing (SMP)
- Multiple CPUs (cores) sharing the same memory.
- Each CPU can run any process.
- Each core typically has its own cache.

### 5.1 Global Scheduler (Single Queue)
- One global ready queue for all CPUs.

**Advantages:**
- High CPU utilization.
- Fairness across all processes.

**Disadvantages:**
- Not scalable (requires global locking).
- Poor cache locality — process might move between CPUs, losing cached data.

### 5.2 Per-CPU Scheduler
- Each CPU maintains its own **local ready queue**.

**Advantages:**
- Scalable (no shared lock).
- Better cache locality.

**Disadvantages:**
- Possible load imbalance (some CPUs idle, others overloaded).

### 5.3 Work Stealing (Hybrid)
- Idle CPUs “steal” tasks from busy CPUs.
- Helps maintain fairness and CPU utilization.

## 6. Processor Affinity
- A process’s **preference to stay** on the same CPU core.
- Improves **cache performance** and **reduces context switching cost**.
- Can be manually set using OS tools (e.g., `taskset` in Linux, Task Manager in Windows).

## 7. Gang Scheduling
- Schedule related processes (e.g., threads of the same program) **simultaneously** across multiple CPUs.
- Common in **High-Performance Computing (HPC)**.
- Ensures synchronized progress for tightly-coupled tasks.

## 8. Real-Time Scheduling

### Real-Time Characteristics
Processes must meet timing constraints.

**Two main types:**
1. **Hard Real-Time:**  
   - Deadlines *must* be met (e.g., autopilot, embedded systems).  
   - Requires strict timing guarantees.

2. **Soft Real-Time:**  
   - Deadlines are best-effort (e.g., video playback, audio streaming).  
   - No absolute guarantee, just higher priority.

**Linux:**
- Supports **only soft real-time**.
- Uses high-priority scheduling for time-sensitive tasks.


## 9. Scheduling Algorithms in Linux

| Type | Algorithm | Description |
|------|------------|-------------|
| Real-time | **SCHED_FIFO** | First-Come, First-Serve |
| Real-time | **SCHED_RR** | Round Robin |
| Normal | **SCHED_NORMAL** | Default for user processes |

### Priority Ranges
- **Real-time:** `0–99` (higher = higher priority)
- **Normal:** `-20 → 19` (niceness values)
- Linux internal range: `-1 → 139`

`nice` command:  
- Lower niceness = higher CPU priority.  
- "Niceness" = willingness to yield CPU time.


## 10. Reading Priorities in `htop`

| Process | PRI | NI | Description |
|----------|-----|----|-------------|
| systemd | 20 | 0 | Normal |
| auditd | 16 | -4 | Elevated priority |
| pipewire (audio) | 9 | -11 | High priority |
| Kernel migration thread | RT | — | Real-time (-100 equivalent) |


## 11. IFS – Ideal Fair Scheduling

### Concept
IFS (Ideal Fair Scheduling) defines the **theoretical goal** of fair scheduling:
> “If there are *N* runnable processes, each should receive **1/N** of the CPU time.”

In other words:
- Every process gets an **equal share** of CPU time.
- A perfectly fair scheduler would distribute CPU time exactly evenly.

IFS is **not directly implementable** (it’s idealized), but it serves as a **reference model** for how real schedulers should behave.

### Key Idea
- Track how much CPU time each process has received so far.
- Always run the process with the **smallest accumulated runtime**.
- Ensures fairness and prevents starvation.


## 12. CFS – Completely Fair Scheduler

### Motivation
- Designed to **approximate IFS** on real systems.
- Replaced the older **O(1)** scheduler in Linux.
- Balances fairness and performance, especially on multi-core systems.

### How It Works (Simplified)
- Each process has a **virtual runtime (vruntime)** that tracks how much CPU time it has used.
- The process with the **lowest vruntime** runs next.
- Uses a **red-black tree** to efficiently select the process with the smallest vruntime.
- Adjusts for priorities: higher-priority tasks accumulate vruntime more slowly.

### Properties
- Approximates ideal fairness (IFS).
- Prevents starvation.
- Scales well for multi-core systems.
- Adapts smoothly to CPU-bound and I/O-bound workloads.

## ✅ Summary

| Concept | Description |
|----------|-------------|
| **Priority Scheduling** | Higher-priority processes preempt lower ones. |
| **Priority Inversion** | Solved with priority inheritance. |
| **Multiprocessor Scheduling** | Global vs Per-CPU queues; work stealing for balance. |
| **Processor Affinity** | Keeps processes on the same CPU for cache efficiency. |
| **Gang Scheduling** | Synchronize parallel tasks. |
| **Real-Time Scheduling** | Hard vs Soft deadlines. |
| **IFS (Ideal Fair Scheduling)** | Theoretical model of perfect fairness. |
| **CFS (Completely Fair Scheduler)** | Linux’s practical implementation of IFS. |


---

# Lecture 11: Virtual Memory

## Virtual Memory Concept

Each process has its own **virtual memory space**, which must:

- Allow multiple processes to coexist.
- Keep processes unaware of each other’s memory unless explicitly shared.
- Be performant (close to using physical memory directly).
- Minimize fragmentation (wasted space).

Memory is **byte-addressable**:

- Each address indexes a single byte.
- Pointers are essentially indexes into this array.


## Segmentation (Historical)

- Segments divide virtual memory into **coarse-grain sections** (code, data, stack, heap).  
- Each segment has:
  - Base address (starting point in physical memory)
  - Limit (size)
  - Permissions (read, write, execute)
- Virtual address structure:  `[Segment Selector | Offset]`
- MMU checks bounds and permissions. Out-of-bounds access → **segmentation fault**.  
- Modern OSes rarely use segmentation because:
- Variable size management complexity
- Fragmentation issues
- Linux disables hardware segmentation by default.

## Paging

To improve efficiency, memory is divided into **fixed-size blocks** called **pages**.

- **Page Size**: Typically 4,096 bytes (4 KB) on x86 systems.
- **Physical Memory Block**: Also called a **frame**.
- MMU translates **virtual pages → physical frames**.
- Benefits:
- Translate blocks instead of individual bytes
- Offset within page remains the same

## Page Table

- **Page Table**: Maps virtual page numbers (VPN) → physical page numbers (PPN).  
- **Virtual Address Structure**: `[VPN | Offset]`

- 
### Page Table Entry (64-bit)

- **Valid bit**: Is the translation valid?
- **Physical Page Number (PPN)**: Maps virtual page to physical page
- **Permissions**: Readable, writable, executable
- **Other flags**: Accessed, dirty, global, supervisor-reserved bits

## Translation Example

### Given

- Page size: 4,096 bytes (12-bit offset)
- Virtual address: `0x0AB0`
- Page table:

| Virtual Page | Physical Page |
|--------------|---------------|
| 0            | 1             |
| 1            | 4             |
| 2            | 3             |
| 3            | 7             |

### Steps

1. Extract **offset**: last 3 hex digits → `0xAB0`
2. Extract **VPN**: remaining digits → `0x0`
3. Look up page table: VPN `0` → PPN `1`
4. Physical address = PPN + offset → `0x1AB0`

Virtual address `0x0001` → VPN 0, offset 0x001 → Physical address `0x1001`

## Quick Notes

- **Offset never changes**; it represents which byte within the page is being accessed.  
- Page tables can get large if virtual address space is large.  
- Typical 64-bit systems do not use the full 64-bit virtual address space.  
  - Example: use 39-bit virtual addresses → 512 GB per process.

## Summary

- Virtual memory allows **process isolation** and **efficient memory usage**.  
- Segmentation is mostly historical; paging is the standard.  
- Page tables map virtual addresses → physical addresses using fixed-size pages.  
- Understanding virtual → physical translation is critical for debugging memory access and understanding OS behavior.


# Lecture 12: Page Tables


Page tables are used to translate blocks of memory at a time. Previously, our major issue was that **page table size was huge**.

We have a virtual address, and we split it into:

- **Offset**: 12 bits (for 4 KB pages)
- **VPN (Virtual Page Number)**: 27 bits (to select the virtual page)

The page table has \(2^{27}\) entries. Each **page table entry (PTE)**:

- Has a **valid bit**
- Stores the **PPN (Physical Page Number)**
- Is 8 bytes in size

The offset remains 12 bits.

### Page Table Size Calculation

Each entry = 8 bytes  
Number of entries = \(2^{27}\)  

\[
2^{27} \times 8 \text{ bytes} = 2^{30} \text{ bytes} = 1 \text{ GB}
\]

- Each process has its **own page table**
- Example: 16 processes → 16 GB for page tables, leaving almost no RAM for programs.

---

## Reducing Page Table Size

To make page tables smaller:

- Fit **one page table per page** in memory
- Page size = `2^12` bytes
- Entry size = `8` bytes
- Number of entries per page table = `2^12 / 8 = 2^9 = 512`

## Multi-Level Page Tables

To translate large addresses without giant tables:

- Split VPN into multiple levels
- Example: 27-bit VPN → 3 levels (9 bits per level)

**Translation process:**

1. L2 page table gives **index for L1 page table**
2. L1 page table gives **index for L0 page table**
3. L0 page table gives the **PPN** for the virtual page

**Memory required per single virtual address:**

- 3 page tables × 4 KB per table = 12 KB
- Much smaller than the 1 GB for a single-level page table

### Entry Structure

- Each **PTE** has a **valid bit** and a **PPN**
- L2/L1 entries point to the next level
- L0 entries point to the actual physical page

## Memory Allocation for Page Tables

- **Memory allocator** uses a **free list** of pages
- Allocate/deallocate in units of pages
- Page tables are just normal pages in physical memory

## Example Translation (Two-Level Page Table)

- Virtual address split:

  - 12-bit offset
  - 9-bit L0 index
  - 9-bit L1 index

- Steps:

  1. Extract offset (12 bits)
  2. Use L1 index to find L0 page table
  3. Use L0 index to find physical page number
  4. Combine with offset → final physical address

- If page table entries point incorrectly (e.g., cyclic reference), this could lead to **segmentation faults** or wrong translations.


## Advantages of Multi-Level Page Tables

- **Significant memory savings**: only allocate tables for used portions of virtual memory
- **Flexibility**: supports large virtual address spaces without allocating a huge single-level table
- **Drawbacks**: each memory access requires multiple table lookups (4 accesses in a 3-level system)


## Summary

- Page tables map **VPN → PPN**
- Multi-level page tables reduce memory usage drastically
- Each page table level is an array of PTEs fitting in a page
- Translation involves walking the page table levels to reach the PPN
- Free list allocator simplifies memory management


# Lecture 13: Page Table Implementation

## Page Tables and Multi-Level Translation

- Start with a root page table for a process.
- Root table tells which L2 page table to use.
- L2 table index selects an entry using **9 bits**.
- Entry points to the next level (L1 page table).
- L1 table points to **L0 page table** in memory.
- L0 table gives the final physical page number (PPN).

### Example

- Maximum one L1 page table in a simple example.
- Full L1 can point to **512 different L1 tables**.
- Each L1 table can also point to 512 different L0 tables.
- All entries combined may need **1 GB** if fully populated.

**Key point:** Multi-level page tables save memory compared to a single giant table.


## Alignment

- **Alignment** ensures memory starts at multiples of page size (e.g., 4 KB).
- Reduces storage: only store **PPN**, offset is implied.
- Check 8-byte alignment: Address divisible by 8 (bottom hex digit 0 or 8).
- Physical pages always start at offset 0 and end at offset all-ones.


## Minimum and Maximum Page Tables

### Scenario

- Process uses **512 virtual pages**.
- Single page table can hold 512 entries per page.

### Minimum Page Tables

- 1 L2, 1 L1, 1 L0 → total **3 page tables**.

### Maximum Page Tables

- Worst case: L2 full → 512 L1 tables, each L1 only points to 1 L0 table.
- Total = **1 + 512 + 512 = 1025 page tables**.

**Contiguous virtual memory** reduces the number of page tables needed.


## Number of Levels Needed

- **Virtual address bits:** 32-bit
- **Page size:** 4 KB → 12-bit offset
- **PTE size:** 4 bytes → 2^2
- Page table entries per page = 2^12 / 2^2 = 2^10 = 1024 entries

### Formula
Levels = `ceil((Virtual bits - Offset bits) / Index bits per level)`


- Example: 32-bit VA → 20 bits for VPN
- Index bits = 10  
- Levels = ceil(20 / 10) = 2 → L0 and L1 indexes

- 64-bit VA (39 bits) → 3 levels
- Extra memory accesses due to more levels.

## Translation Lookaside Buffer (TLB)

- **TLB** is a cache for VPN → PPN mappings.
- Avoids repeated multi-level page table traversal.
- TLB hit → fast access  
- TLB miss → go through page table translation
- TLB only stores a limited number of entries
- Typically **per-process**, flushed on context switch if no hardware PID support.

### Effective Access Time

`TLB_Hit_Time` = `TLB_Search + Mem`

`TLB_Miss_Time` = `TLB_Search + 2 * Mem`

`EAT` = `α * T_hit + (1 - α) * T_miss`

- Example: 80% hit, TLB hit = 10 ns, memory = 100 ns  
- EAT = 0.8*10 + 0.2*210 = 130 ns  
- 30% slower than raw memory


## Memory Access Patterns

- Accessing contiguous memory improves performance:
  - More likely same page → fewer TLB misses.
- Example `test_tlb`:
  - Allocate 4 KB, stride 4 bytes → 1 miss, 123 hits
  - Allocate 512 MB, stride 4 KB → every access a miss → 30x slower


## Heap and Stack Management

- **Heap**: Managed by `malloc`/`sbrk`/`mmap`.
  - Kernel maps pages from free list.
  - Shrinking heap is limited to contiguous pages at the end.
- **Stack**:
  - Fixed size + guard page
  - Guard page triggers stack overflow error
  - Overflow past guard → segfault


## Kernel and Fixed Virtual Addresses

- Kernel may map certain virtual addresses for performance:
  - e.g., reading system time without system call
- Helps processes avoid expensive system calls for known resources.


**Key Takeaways:**

- Multi-level page tables save memory.
- Alignment simplifies PPN storage.
- TLBs improve performance significantly.
- Contiguous virtual memory reduces page tables and TLB misses.
- Stack overflow protection uses guard pages.
- Kernel can provide fixed virtual addresses for performance.

## Userspace Memory Allocation
- **`sbrk`** grows or shrinks the **heap** (not the stack).  
- When growing, it **grabs pages from the free list** and sets `PTE_V` (valid) and permissions.  
- In practice, `sbrk` is rarely used to shrink heaps — once allocated, pages stay claimed.  
- Modern memory allocators typically use **`mmap`** instead, to get large virtual memory blocks efficiently.

## Fixed Virtual Addresses
- The **kernel can map fixed virtual addresses** into a process, allowing user programs to read kernel data directly.  
- This avoids system calls — e.g., `clock_gettime()` just reads from a mapped address instead of making a syscall.

## Page Faults & Virtual Memory Management
- **Page faults** are exceptions triggered when:
  - The MMU cannot find a valid translation, or  
  - Permission checks fail.  
- They let the OS handle memory dynamically — enabling:
  - **Lazy allocation**
  - **Copy-on-write**
  - **Swapping pages to disk**

## Page Tables & Address Translation
- The **MMU (Memory Management Unit)** translates virtual to physical addresses using **page tables**, which can be:
  - **Single large tables** (wasteful for 32-bit systems)
  - **Multi-level tables** (space-efficient for sparse memory)
  - **TLB-backed** (Translation Lookaside Buffer) to speed up access

---

# Lecture 16: Threads

## Concurrency vs Parallelism

These terms are synonyms in English, but **not in computer science**.

### Concurrency
- Switching between two or more tasks.
- Tasks **make progress** but **not necessarily simultaneously**.
- Example: You work on task A, get interrupted, work on task B, and switch back and forth.

### Parallelism
- Running two or more tasks **at the exact same time**.
- Requires independent execution units (e.g., multiple CPUs).

### Key Difference
- **Concurrency** = Making progress on multiple things.
- **Parallelism** = Doing multiple things *at the same instant*.

If concurrency happens **fast enough**, it can appear like parallelism.

## Real-Life Example: Dinner Table Scenario

Imagine sitting at a dinner table. You can:
- Eat 🍽️
- Drink 🥤
- Talk 💬
- Gesture 🤌

Assume:
- You’re polite (not a “savage”).
- You have one mouth.
- Once you start eating, you **don’t stop** until you finish.

Let’s check combinations:

| Tasks | Parallel? | Concurrent? | Reason |
|-------|------------|-------------|--------|
| Talk & Drink | ❌ | ✅ | One mouth; can alternate |
| Eat & Drink | ❌ | ❌ | You eat continuously |
| Eat & Gesture | ✅ | ❌ | One task uses hands, other uses mouth |
| Gesture & Drink | ✅ | ✅ | Independent body parts |

So, concurrency ≠ parallelism.  
If you alternate *very* fast, it might *look* parallel.

## Threads: The Basics

Threads are like **lightweight processes** that **share memory**.

### Comparison to Processes
- Each thread has its own:
  - **Registers**
  - **Program counter**
  - **Stack**
- Threads within a process share:
  - **Heap memory**
  - **Global variables**
  - **File descriptors**
  - **Code and data segments**

### Why Use Threads?
- Faster and **less expensive to create** than processes.
- Ideal for **concurrent tasks** like handling multiple web requests.
- A single CPU can switch threads quickly, giving the illusion of parallelism.

## Memory and Address Space

All threads in a process share the **same address space**.

- **Stack** → Local to each thread.
- **Heap & Global variables** → Shared between threads.

Therefore:
- Stack variables are **thread-safe**.
- Heap/global variables may **change unexpectedly** due to other threads.

## Threads vs Processes Summary

| Property | Process | Thread |
|-----------|----------|--------|
| Address Space | Independent | Shared |
| Heap/Data | Separate | Shared |
| Stack | Own | Own |
| Registers | Own | Own |
| Creation Cost | High | Low |
| Context Switch | Expensive | Fast |
| Exit Cleanup | Removes all | Removes only its stack |
| Relationship | Independent | Within a process |

If a **process dies**, all its **threads** die as well.

## POSIX Threads (pthreads)

We use **POSIX Threads (pthreads)** on Unix, macOS, and Linux.

Include the header:
```c
#include <pthread.h>
```

Compile with 
```
gcc file.c -pthread
```

# Creating a Thread

Equivalent of `fork()` for threads:

```c
pthread_create(pthread_t *thread, 
               const pthread_attr_t *attr,
               void *(*start_routine)(void *),
               void *arg);
```
## Parameters

- **thread** — Pointer to a thread ID (similar to a process ID).  
- **attr** — Thread attributes (can be `NULL`).  
- **start_routine** — Function the thread will run.  
- **arg** — Argument passed to the function.

### Return Value

- `0` → Success  
- Error number → Failure  

## Example: Creating a Thread

```c
#include <pthread.h>
#include <stdio.h>

void *run(void *arg) {
    printf("In run\n");
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_create(&thread, NULL, run, NULL);
    printf("In main\n");
    return 0;
}
```

### Expected Output
```
In main
In run
```

### Problem: Missing Output

In practice, you might only see:
```
In main
```

### Why?

Because `main()` returned before the thread printed `"In run"`.  
When the **main thread exits**, the **entire process terminates**, killing all threads.


## Fixing It: `pthread_join`

We can wait for the thread to finish using **join**:

```c
pthread_join(pthread_t thread, void **retval);
```

- Equivalent to wait() for processes.
- Blocks until the thread terminates.
- retval stores the thread’s return value (or can be NULL).

### Example 

```c
#include <pthread.h>
#include <stdio.h>

void *run(void *arg) {
    printf("In run\n");
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_create(&thread, NULL, run, NULL);
    pthread_join(thread, NULL);
    printf("In main\n");
    return 0;
}
```

### Output
```
In main
In run
```

> **Note:**  
> The order may vary due to thread scheduling.

## Thread Exit

To end a thread early:
```c
pthread_exit(void *retval);
```
- Works like `exit()` for processes.
- The value passed to `pthread_exit()` becomes available to another thread via `pthread_join()`.
- Returning from the thread’s start function is equivalent to:
  ```c
  pthread_exit(NULL);
  ```


# Detached Threads

When you create a thread with `pthread_create()`:

## Joinable vs Detached

1. **Joinable threads** (default)  
   - The thread keeps some internal memory (its “thread control block”) even after it finishes.  
   - You **must** call `pthread_join()` to clean up that memory and optionally get the thread’s return value.  
   - If you don’t call `pthread_join()`, the memory stays allocated → **memory leak**.

2. **Detached threads**  
   - They clean up their own resources automatically after finishing.  
   - You **cannot** call `pthread_join()` on them.  
   - Useful when you don’t care about the thread’s return value and want it to run independently.

## 2. Analogy

Think of threads like guests at a party:

| Type      | What you do after the guest leaves | Memory/resources |
|-----------|----------------------------------|----------------|
| Joinable  | You check in and see how they did | You clean up after them |
| Detached  | You don’t care, they leave quietly | They clean up themselves |

## Example

```c
#include <pthread.h>
#include <stdio.h>

void* run(void* arg) {
    printf("Thread running!\n");
    return NULL;
}

int main() {
    pthread_t thread;
    // Create a detached thread
    pthread_create(&thread, NULL, run, NULL);
    pthread_detach(thread); // make it detached

    printf("Main finished!\n");
    return 0;
}
```

### Output (order may vary):

```
Main finished!
Thread running!
```
- The thread runs in the background.
- You don’t join it; it cleans itself up.


# Lecture 17: Thread Implementation 

## Threading Approaches

There are two main approaches to implementing threads:

### 1. User Threads
- Implemented completely in **user space**.
- Kernel is **not aware** of user threads.
- The process starts with a single thread. User threads enable **concurrency**, switching execution as needed.
- **Advantages**:
  - Fast creation and destruction (no system calls).
  - Portable (works on Windows, macOS, Linux, etc.).
- **Drawbacks**:
  - No parallelism (kernel schedules only the process, not individual threads).
  - If one thread performs a blocking system call, the **entire process blocks**.

### 2. Kernel Threads
- Implemented in **kernel space**.
- Created via **system calls** (slower than function calls).
- Kernel manages everything (stack allocation, registers, scheduling).
- Can achieve **true parallelism** on multi-core systems.
- **Advantages**:
  - Other threads can continue if one blocks.
  - Enables parallel execution on multiple cores.
- **Drawbacks**:
  - Slower creation.
  - Less control over scheduling.

## Thread Libraries and Mapping

Threading libraries manage the mapping between user threads and kernel threads:

| Model        | Description                                                                 |
|--------------|-----------------------------------------------------------------------------|
| **Many-to-One**  | Multiple user threads mapped to a single kernel thread (pure user threads, fast, portable, cooperative). |
| **One-to-One**   | Each user thread maps to a single kernel thread (e.g., `pthread_create`, supports parallelism).       |
| **Many-to-Many** | Multiple user threads mapped to fewer kernel threads, dynamic mapping (complex, like Java Virtual Threads). |

- **Many-to-One**: Pure user space, cooperative threads.
- **One-to-One**: Kernel threads, allows parallelism.
- **Many-to-Many**: Hybrid approach, complex mapping, can be unpredictable.

## Thread Support & Control

- Threads require a **Thread Control Block (TCB)**, similar to a **Process Control Block (PCB)**.
- **User Threads** need a runtime system for **scheduling**.
- **Kernel Threads** rely on kernel scheduling.
- Thread states: **Ready**, **Running**, **Blocked**.

## Signals in Multi-Threaded Processes

- If a signal is sent to a process with multiple threads:
  - Linux picks **one thread at random** to handle the signal.
- This can lead to non-deterministic behavior.
- **Recommendation**: Use **thread pools** for short-lived tasks to maintain performance and avoid creating thousands of threads.

## Example: Race Condition with Threads

```c
#include <pthread.h>
#include <stdio.h>

#define NUM_THREADS 8

int counter = 0; // Global variable

void* increment(void* arg) {
    for(int i = 0; i < 10000; i++) {
        counter++; // Race condition!
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];

    for(int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, increment, NULL);
    }

    for(int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("Final counter value: %d\n", counter);
    return 0;
}
```

**Expected Result:**  
`80,000 (8 * 10,000)`

**Actual Result:**  
Can vary due to **race conditions**.

**Why?**  
`counter++` is **not atomic**.  
The operation involves **read → increment → write**, which can **interleave between threads**, leading to inconsistent results.


# Fixing Race Conditions

## Option 1: Join Immediately
- Run threads **sequentially** to ensure safe increments.
- **Drawback:** No parallelism, defeating the purpose of multi-threading.

## Option 2: Thread-local Counters
- Each thread increments a **local counter**.
- Combine results **after all threads finish**.

```c
#define NUM_THREADS 8
static int counter = 0;

void* increment(void* arg) {
    int* local = malloc(sizeof(int));
    *local = 0;
    for(int i = 0; i < 10000; i++) {
        (*local)++;
    }
    return local;
}

int main() {
    pthread_t threads[NUM_THREADS];
    // Create threads
    for(int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, increment, NULL);
    }

    // Join threads and combine results
    for(int i = 0; i < NUM_THREADS; i++) {
        int* ret;
        pthread_join(threads[i], (void**)&ret);
        counter += *ret;
        free(ret);
    }

    printf("Final counter value: %d\n", total);
}
```
> **Note:** pthread_join waits for a thread to finish before returning.

### Visualization
```
Thread 1: *local = 10000
Thread 2: *local = 10000
Thread 3: *local = 10000
Thread 4: *local = 10000
...
main thread waits for thread i -> adds *local to counter
```

- Threads run in parallel, but their increments don’t touch `counter`.
- Only main thread modifies `counter`, sequentially after each join.

---

# Lecture 21: Locks

## Key Concepts
### Data Race (Important Definition)
A **data race** occurs when:
1. Two threads **concurrently access the same memory**, and  
2. **At least one** of those accesses is a **write**.

A data race ⇒ **undefined behavior**, usually incorrect results.

## 🔍 Why Data Races Happen
### Example: `counter++`
Even though it looks atomic in C, `counter++` → **three separate operations**:

1. **Load** from memory  
2. **Increment** (register operation)  
3. **Store** back to memory

Two threads can interleave these operations and break correctness.

### GCC “Gimple” Representation
The compiler transforms `counter++` into something like:
```
D1 = *counter // load
D2 = D1 + 1 // increment
*counter = D2 // store
```

Each line is **atomic**, but the entire increment is **not**.

## Result of Interleaving
For two threads incrementing a zero:

- Only **one** ordering produces the correct result (`2`)
- Most interleavings produce the wrong result (`1`)
- With more threads → correctness becomes “miracle”-level unlikely

## Fixing Data Races: `pthread_mutex_t`
A **mutex** ensures **Mutual Exclusion**:  
→ Only **one** thread can execute the protected code at a time.

### Basic API
```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

// or dynamic init:
pthread_mutex_init(&lock, NULL);

// lock
pthread_mutex_lock(&lock);

// unlock
pthread_mutex_unlock(&lock);

// destroy (if dynamically created)
pthread_mutex_destroy(&lock);
```

### Example: Fixing the Race
```c
pthread_mutex_lock(&lock);
counter++;               // now safe
pthread_mutex_unlock(&lock);
```

## Mutex Behavior and Correctness

### Correctness With Locks
When a shared update (e.g., `counter++`) is wrapped in a mutex lock/unlock, every thread is forced to execute the critical section one at a time.  
This ensures every run returns the correct answer.


## Mutex Caveats

### Double Locking Causes Deadlock
A thread that locks the same mutex twice without unlocking will deadlock.  
All other threads attempting to acquire the same lock will also be blocked indefinitely.
```c
pthread_mutex_lock(&lock);
pthread_mutex_lock(&lock);   // blocks forever waiting on itself
```

## Mutex Properties (Correctness Requirements)

### 1. Mutual Exclusion
Only one thread may be inside the critical section at any time.

### 2. Liveness / Progress
If multiple threads attempt to acquire the lock, one of them must eventually succeed.

### 3. Bounded Waiting (No Starvation)
A thread attempting to acquire the lock must not wait forever; each thread should eventually make progress.

## Threads Do Not Automatically Make Code Faster

Using threads with a mutex around very small operations (e.g., `counter++`) often makes a program:

- **Correct**  
- **But slower than a single-threaded implementation**

**Reason:**  
All threads serialize on the same lock, effectively running one at a time.

**Guideline:**  
Use mutexes only where necessary.

---

# Lecture 22: Locks Implementation 

## Overview
This lecture discusses software and hardware implementations of locks, issues with basic spin locks, and advanced mutex and read-write lock implementations for fairness and performance.

## 1. Software Lock Basics
- Initial naive software locks **fail Mutual Exclusion**:
  - Two threads may pass through `lock()` simultaneously.
- Software locks require:
  - Atomic **loads and stores**.
  - Sequential instruction execution (though modern hardware may reorder).

### Classic Algorithms
- **Peterson's Algorithm**
- **Lamport's Bakery Algorithm**
  - Number-based "first-come, first-served" approach.
  - Not scalable for multiprocessor systems.

## 2. Hardware Support: Compare-and-Swap (CAS)
- CAS is an atomic instruction:  
  ```c
  int compare_and_swap(int* ptr, int old, int new);
  ```
- Returns the original value at ptr.
- Changes value to new only if current value equals old.

**Example Lock Implementation: Spin Lock**
```c
while (compare_and_swap(&lock, 0, 1) != 0) {
    // busy-wait
}
```
- Atomicity ensures only one thread acquires the lock.
- Other threads spin until lock is released.

## Problems with Basic Spin Locks
- **Busy-waiting wastes CPU.**
- **Inefficient on single-core systems.**
- Needs `yield()` to avoid spinning when the lock holder is blocked.

## Improving Spin Locks
- Add `yield()` when lock acquisition fails.
  - Reduces CPU waste.
- **Problem: Thundering Herd**
  - Multiple threads wake up simultaneously, all attempt the lock.
  - Only one succeeds → wasted attempts.
  - Unfair: no guaranteed order of lock acquisition.

## Fair Mutex Implementation
- **Wait queue approach**:
  1. Threads failing to acquire the lock add themselves to the queue and sleep.
  2. Unlock operation wakes **exactly one thread** from the queue.
- **Uses a spin lock (guard)** to protect the queue from data races.
- **Transfers lock to the next thread safely.**
- **Issues addressed**:
  - **Lost wakeup:** a thread sleeps and never wakes; guard ensures atomicity.
  - **Wrong thread acquires lock:** queue order enforces fairness.

## Fair Mutex Using Guard Variable

### Structure
```c
typedef struct {
    int lock;     // mutex state: 0 = unlocked, 1 = locked
    int guard;    // spin lock to protect shared state
    queue_t *q;   // wait queue for threads
} mutex_t;
```

## Lock Function
```c
void lock(mutex_t *m) {
    // Acquire guard spin lock
    while (compare_and_swap(m->guard, 0, 1)); // lock(&m->guard);

    if (m->lock == 0) {
        // Lock is free → acquire it
        m->lock = 1;
        m->guard = 0; // release guard unlock(&m->guard);
    } else {
        // Lock is held → enqueue thread and sleep
        enqueue(m->q, self);
        m->guard = 0; // release guard before sleeping unlock(&m->guard);
        thread_sleep(); // lock will be transferred on wakeup
    }
}
```

## Unlock Function

```c
void unlock(mutex_t *m) {
    // Acquire guard spin lock
    while (compare_and_swap(m->guard, 0, 1)); // lock(&m->guard);

    if (queue_empty(m->q)) {
        // No waiting threads → release lock
        m->lock = 0;
    } else {
        // Transfer lock directly to next thread
        thread_wakeup(dequeue(m->q));
    }

    m->guard = 0; // release guard unlock(&m->guard);
}
```

## Step-by-Step Example: T1 acquires lock first, T2 tries to acquire

## Initial State

| Thread | Action           | Lock | Guard | Queue |
|--------|-----------------|------|-------|-------|
| T1     | About to lock    | 0    | 0     | empty |
| T2     | Wants lock       | 0    | 0     | empty |


## Step 1: T1 Calls `lock(&m)`

1. T1 acquires `guard` → spin lock succeeds (`guard = 1`).
2. Checks `lock` → it is 0, so T1 sets `lock = 1` (acquires mutex).
3. Releases guard (`guard = 0`).

| Thread | Action          | Lock | Guard | Queue |
|--------|----------------|------|-------|-------|
| T1     | Holds mutex     | 1    | 0     | empty |
| T2     | Waiting to lock | 1    | 0     | empty |


## Step 2: T2 Calls `lock(&m)`

1. T2 acquires `guard` → spin lock succeeds (`guard = 1`).
2. Checks `lock` → it is 1 (held by T1).
3. Adds itself to the queue (`m->queue` = T2).
4. Releases guard (`guard = 0`).
5. Calls `thread_sleep()` → T2 is now blocked.

| Thread | Action          | Lock | Guard | Queue |
|--------|----------------|------|-------|-------|
| T1     | Holds mutex     | 1    | 0     | T2    |
| T2     | Sleeping        | 1    | 0     | T2    |

## Step 3: T1 Calls `unlock(&m)`

1. T1 acquires guard → succeeds (`guard = 1`).
2. Checks `queue_empty()` → false, T2 is in queue.
3. Calls `thread_wakeup(dequeue(m->queue))` → wakes T2 **and transfers lock**.
4. Releases guard (`guard = 0`).

| Thread | Action       | Lock | Guard | Queue |
|--------|-------------|------|-------|-------|
| T1     | Finished    | 1 (transferred to T2) | 0 | empty |
| T2     | Woken up    | 1    | 0     | empty |


## Step 4: T2 Resumes

1. T2 resumes from `thread_sleep()`.
2. Already owns mutex (lock transferred by T1).
3. Proceeds into the critical section.

| Thread | Action          | Lock | Guard | Queue |
|--------|----------------|------|-------|-------|
| T2     | In critical section | 1 | 0 | empty |


## Key Takeaways

- **Guard spin lock** protects `lock` and `queue` from data races.
- Prevents **lost wakeup**: T2 adds itself to queue before T1 tries to wake.
- Ensures **fair transfer**: unlock hands off mutex to the first waiting thread.
- Avoids busy waiting for the main lock.
- Subtle edge case: If the woken thread is not yet asleep when unlock tries `thread_wakeup()`, retry is needed.  

## Reader-Writer Lock (RWLock) and the Guard

A **reader-writer lock (RWLock)** allows:

- Multiple **readers** to access a shared resource simultaneously.
- Only **one writer** at a time, excluding readers.

To manage this safely, we use two locks:

1. **`lock`** – Protects access to the actual shared resource.
2. **`guard`** – Protects updates to the reader count (`nreader`).

## The Reader Problem

When multiple threads are reading:

- Each reader must update `nreader` safely.
- The **first reader** acquires the main `lock` to block writers.
- The **last reader** releases the main `lock` to allow writers.
- We need a separate `guard` to prevent **data races** on `nreader`.


## Why All Readers Unlock the Guard

- Each reader locks `guard` **only to increment or decrement `nreader`**.
- After updating `nreader` (and acquiring `lock` if first reader), the reader **unlocks `guard`**.
- This allows other threads to safely modify `nreader` while the current reader is still reading.
- **Resource access is still protected** by `lock` until the last reader leaves.

## What `unlock(&guard)` does

- Simply releases the spin lock protecting `nreader`.
- **Does not release the shared resource**.
- Ensures only one thread updates the reader count at a time.


## Analogy

- **`guard`** = clipboard to track how many people are in the room.
- **`lock`** = door to the room (blocks writers if readers are inside).

**Steps:**

1. Entering:
   - Grab the clipboard (`lock(&guard)`).
   - Write down your name (increment `nreader`).
   - If first reader, lock the door (`lock(&lock)`).
   - Put the clipboard back (`unlock(&guard)`) and start reading.

2. While reading:
   - Clipboard is free for others, but the door remains locked to block writers.

3. Leaving:
   - Grab the clipboard (`lock(&guard)`).
   - Remove your name (decrement `nreader`).
   - If last reader, unlock the door (`unlock(&lock)`).
   - Put the clipboard back (`unlock(&guard)`).


## Key Points

- `guard` ensures **safe updates to reader count**.
- `lock` ensures **exclusive access for writers** and **shared access for readers**.
- Unlocking `guard` does **not affect the reader's access** to the resource.

---

# Lecture 23: Semaphores

## Locks and Mutexes

**Purpose:** Prevent **data races**.  

- **Mutex/lock** ensures **mutual exclusion**: only one thread executes a critical section at a time.  
- This **prevents concurrent writes** to the same memory location.  

> **Important:** Locks **do not ensure ordering** between threads — just exclusive access.


### Example: Thread Ordering

We have two threads:  

```c
void* print_first(void* arg) {
    printf("This is first\n");
}

void* print_second(void* arg) {
    printf("I'm going second\n");
}
```
- Running this may print messages in any order.
- `join()` can enforce order, but it forces serial execution, limiting parallelism.

## Attempt Using Mutex
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

```c
void* print_second(void* arg) {
    pthread_mutex_lock(&mtx);
    pthread_mutex_lock(&mtx);
    printf("I'm going second\n");
    pthread_mutex_unlock(&mtx);
}
void* print_first(void* arg) {
    printf("This is first\n");
    pthread_mutex_unlock(&mtx);
}
```

## Mutex Ordering Limitations

- Works sometimes, but **undefined behavior** occurs if:
  - Unlocking an unlocked mutex.
  - Unlocking a mutex locked by another thread.
- **Conclusion:** Mutexes alone are **not reliable for ordering**.

## Semaphores

- **Purpose:** Used for signaling and ordering between threads.
- Can also be shared across processes, but mostly used for threads in this course.

### Key Properties

- **Value ≥ 0** (initial value set during creation)
- **Operations (atomic):**
  - `wait()` → decrement; block if value = 0
  - `post()` → increment; unblock waiting threads

### Analogy

Semaphores are like **traffic lights**:  
- `wait()` → stop until light is green  
- `post()` → turn green for the next car

## Example: Ordering Two Threads

```c
sem_t sem;
sem_init(&sem, 0, 0); // initial value 0

void* print_first(void* arg) {
    printf("This is first\n");
    sem_post(&sem);
}

void* print_second(void* arg) {
    sem_wait(&sem);
    printf("I'm going second\n");
}
```
- Guarantees first prints before second, while allowing parallel execution after that.

## Adding a Third Thread

- To enforce order with three threads, **a second semaphore** is needed:

```c
sem_t sem2;
sem_init(&sem2, 0, 0);

void* print_third(void* arg) {
    sem_wait(&sem2);
    printf("I'm last\n");
}
```
- Place `sem_post(&sem2)` after `print_second()` to ensure the proper sequence.

## Mutex vs Semaphore

| Feature        | Mutex                        | Semaphore                     |
|----------------|------------------------------|-------------------------------|
| Use            | Protect shared data           | Enforce ordering or resources |
| Operations     | `lock()` / `unlock()`         | `wait()` / `post()`           |
| Initial Value  | Not relevant                 | Critical (≥0)                 |
| Behavior       | Exclusive access             | Counts available resources    |

**Rule of thumb:**
- Use **mutexes** for data races  
- Use **semaphores** for thread ordering  

## Thread Safety

- Functions may internally split into **multiple system calls**.  
- Example: `printf()` may execute multiple writes.  
- Without locks, outputs may **interleave between threads**.  
- **Solution:** Use mutexes to ensure **atomic execution** of non-thread-safe functions.

## Producer-Consumer Example

### Problem Setup
- Circular buffer shared between multiple producer and consumer threads.  
- Race conditions occur if:
  - Consumers read **empty slots**  
  - Producers write **filled slots**

### Solution: Semaphores

- **`filled_slots`** → counts slots with data  
  - Consumers `wait(filled_slots)`  
  - Producers `post(filled_slots)`

- **`empty_slots`** → counts available empty slots  
  - Producers `wait(empty_slots)`  
  - Consumers `post(empty_slots)`

### Initial Values
- `filled_slots = 0` → no data initially  
- `empty_slots = buffer_size` → all slots empty initially

### Producer Code

```c
wait(empty_slots);    // wait for empty slot
fill_slot();          // add data
post(filled_slots);   // signal slot filled
```
### Consumer Code

```c
wait(filled_slots);   // wait for filled slot
empty_slot();         // consume data
post(empty_slots);    // signal slot empty
```

### Analogy

**Parking lot analogy:**

- `empty_slots` → free parking spots  
- `filled_slots` → cars parked  

This ensures **orderly access without stopping concurrency**.

### Key Takeaways

- **Mutexes** → prevent data races, exclusive access  
- **Semaphores** → ordering and resource counting  
- Thread-safe functions are essential; otherwise, use mutexes  

**In producer-consumer problems:**  
- Use semaphores to prevent **overwriting** or **reading empty slots**
