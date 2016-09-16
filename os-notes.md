# COP 4610 Notes
<hr />
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
