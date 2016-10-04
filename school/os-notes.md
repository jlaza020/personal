# COP 4610 Notes
<hr />

## Table of Contents
[Chapter 2] (#chapter-2)  
[Chapter 4] (#chapter-4)  
[Chapter 5] (#chapter-5)  

## Chapter 2

**Types of OS services**:

Types that help the user:
  
+ UI
+ program execution
+ I/O operations
+ file-system manipulation
+ communications
+ error detection

Types that don't help the user:
  
+ resource allocation
+ accounting
+ protection and security

**OS Interfaces**

+ command-line / command interpreter
+ GUI

Operating systems provide an interface to the services made available by an 
operating system.  

Three common APIs:

+ Win32
+ POSIX API
+ Java API

A system-call interface in a programming language serves as the link to system 
calls made available by the OS - managed by run-time support library.  

Three methods to pass params to the operating system:

+ registers
+ in-memory block or table (and address of block passed as param in register)
+ stacks (params pushed by program and popped by OS)

**Types of System Calls**:

There are 6 types of system calls:

+ process control:
end, abort, load, execute, create processes, wait for time
event, signal event, allocate, free memory

+ file manipulation:
create, delete, open, close, read, write to file, get, set
file attributes

+ device manipulation:
request, release device, read, write, get, set device
attributes, logically attach, detach devices

+ information maintenance: 
get set time, date, system data, process, file,
device attributes

+ communications: 
create, delete communication connection, send receive messages
transfer status info, attach, detach remote devices

+ protection

Debugger can examine dump (state of memory written to disk) when a program
abnormally ends (aborts, crashes).  

In a batch system, job is terminated and next job is executed if a job aborts. 

Control cards indicate special recovery actions in case an error occurs and are
a batch-system concept. It is a command that  manages the execution of a
process.  

MS-DOS OS is a single-tasking system. FreeBSD is a multitasking system.  

Two models of IPC:

+ message-passing model
+ shared-memory model

Types of systems programs / utilities (convenient environment for program
development and execution):

+ file management
+ status info
+ file modification
+ programming-language support
+ program loading and execution
+ communications

Systems programs are different from application programs (IE, Word, SQL Server,
etc.)  

Mechanisms determine *how* to do something, while policies determine *what*
will be done.  

consolidation

simulation

process failure: core dump
kernel failure: crash dump

States of processes:

+ New
+ Running
+ Waiting
+ Ready
+ Terminated

Process Control Block (PCB) / Task Control Block:

+ Process state
+ Program counter
+ CPU registers
+ CPU-scheduling info
+ memory-management info
+ accounting info
+ I/O info
+ (thread info)

Once process is allocated the CPU and is executing, one of several events could
occur:

+ issue an I/O request and be placed in I/O queue
+ process creates a new subprocess and waits for subprocess termination
+ removed foricbly from CPU as result of interrupt, and put onto ready queue

Job scheduler vs CPU scheduler

## Chapter 4

Threads share:

+ code section
+ data section
+ open files
+ signals

Benefits of threads:

+ responsiveness
+ resource sharing
+ economy
+ scability

Challenges:

+ dividing activites
+ balance
+ data splitting
+ data dependency
+ testing and debugging

Models:

+ M:1
	+ management by thread library in user space (efficient)
	+ entire process will block if thread makes a blocking system call
	+ cannot run in parallel on multiprocessers
+ 1:1 
	+ more concurrency
	+ not efficient (generate a new kernel thread each time)
+ M:M
	+ can create as many threads as user wants
	+ kernel threads can run in parallel on multiprocessor
+ 2-level:
	+ 1:1 and M:M
	
Thread library gives programmer API for creating and managing threads.

Libraries:

+ POSIX Pthreads
	+ user and kernel
	+ specification of thread behavior, not implementation
+ Win32
	+ kernel
+ Java
	+ Java programs

Parallelism implies concurrency. The converse is NOT true.


## Chapter 5

Process scheduling and thread scheduling are often used interchangeably.

Process execution consists of a cycle of CPU execution and I/O wait. Process execution begins with a CPU
burst, and then an I/O burst. The final CPU burst ends with a system request to terminate execution.

There is usually a large number of short CPU bursts and a small number of long CPU bursts. I/O-bound
program usually has many short CPU bursts. And CPU-bound program might have a few long CPU bursts.

The short-term scheduler (CPU scheduler) selects the process in the ready queue to be executed once the
CPU becomes idle. The schedule selects a process form the ones in memory that are ready to execute and
allocates CPU to that process.

A ready queue could be:

+ FIFO
+ Priority Queue
+ Tree
+ Unordered Linked List

Records in ready queues are usually PCBs of the processes.

CPU-scheduling decisions may take place when:

+ a process switches from running to waiting state (nonpreemptive / cooperative)
+ a process switches from running state to ready state (for example, when an interrupt occurs)
(preemptive)
+ a process switches from the waiting state to the ready state (completion of I/O) (preemptive)
+ a process terminates (nonpreemptive / cooperative)

Windows 95 and later utilize preemptive CPU-scheduling.

Preemptive scheduling incurs a cost associated with access to shared data.

The dispatcher is the module that gives control of the CPU to the process selected by the short-term
scheduler. This function involves:

+ switching context
+ switching to user mode
+ jumping to the proper location in the user program to restart the program

**dispatch latency**: time it takes for the dispatcher to stop one process and start another running

Many criteria is used for comparing CPU-scheduling algorithms:

+ CPU utilization: keep CPU as busy as possible
+ Throughput: number of processes completed per time unit
+ Turnaround time: how long it takes to fully execute a process to termination time
+ Waiting time: how long a process waits in ready queue (sum of periods spent waiting)
+ Response time: limited by speed of output device

Types of scheduling algorithms:

+ First-Come, First-Serve (FCFS)
	+ Pros
		+ easy to implement
	+ Cons
		+ long average waiting time
+ Shortest-Job-First (SJF): has preemptive and nonpreemptive versions
	+ Pros
		+ optimal b/c of minimum average waiting time
	+ Cons
		+ cannot be implemented at the level of short-term CPU scheduling
		+ no way to know the length of the next CPU burst (may be able to predict)
+ Priority Scheduling: equal-priority processes are scheduled in FCFS order. Can be preemptive or
nonpreemptive
	+ Pros
		+ aging solves starvation problem
	+ Cons
		+ starving
+ Round-Robin: time-slicing
	+ Pros
	+ Cons
		+ average waiting time is long
+ Multilevel queue scheduling algorithm
	+ distinction between foreground and background processes
	+ multiple queues
	+ each queue has its own scheduling algo. and algo. across queues.
	+ multilevel feedback queue scheduling algorithm: process moves in between queues
	
Quiz on chapters 1-7 on Oct. 20th:

+ Skip 1, 2.8, 2.9, 4.4.2, 4.4.3, 4.5.2, 4.5.4, 4.7, 5.9.1, 5.9.2, 5.9.3, 5.10
+ Know Ch. 6 Windows and Solaris
+ KNOW Chapter 4, Chapter 5, Chapter 6, Chapter 7
+ Problems: 6.1-6.9, 6.10, 6.12, 6.14, 6.16, 6.18, 6.21, 6.29, 6.31a
