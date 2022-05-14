# Linux/Unix Processes


## Define the boot process of a Linux system

- Once you power a system on, the first thing that happens is the BIOS loads and performs POST or a power on self test, to ensure that the components needed for a boot are ok. 
	- For instance if the CPU is defective, the system will give an error that POST has failed. (BIOS stands for Basic Input/Output system)  
- After POST the BIOS looks at the MBR or master book record and executes the boot loader. 
	- In case of a Linux system that might be GRUB or Grand Unified BootLoader. GRUB’s job is to give you the choice of loading a Linux kernel or other OS that you may be running  
- Once you ask GRUB to load a kernel, usually an initial ramdisk kernel is loaded, which is a small kernel that understands filesystem. 
	- This will in turn mount the filesystem and will start the Linux kernel from the filesystem  
- The kernel will then start init, which is the very first process, usually having PID 1. 
- Init will look at /etc/inittab and will switch to the default run-level which on Linux servers tends to be 3.  
- There are different run level scripts in /etc/rc.d/rc[0-6].d/ which are then executed based on the runlevel the system needs to be in.  


### How do you make changes to kernel parameters that you want to persist across boot?
/etc/sysctl.conf contains kernel parameters that can be modified. You can also use the sysctl command to make changes at runtime.

## What is the difference between a process and a thread?

A thread is a lightweight process. Each process has a separate stack, text, data and heap. Threads have their own stack, but share text, data and heap with the process. Text is the actual program itself, data is the input to the program and heap is the memory which stores files, locks, sockets.

## What is a zombie process?

A zombie process is a one which has completed execution, however it’s entry is still in the process table to allow the parent to read the child’s exit status. The reason the process is a zombie is because it is “dead” but not yet “reaped” by it’s parent. Parent processes normally issue the wait system call to read the child’s exit status whereupon the zombie is removed. The kill command does not work on zombie process. When a child dies the parent receives a SIGCHLD signal. Zombie processes do not take up system resources, except for the tiny amount of space they use up when appearing in the process id table.

## How do you end up with zombie processes?

Zombie processes are created when the parent does not reap the child. This can happen due to parent not executing the wait() system call after forking.

## How to daemonize a process

1.  The fork() call is used to create a separate process.
2.  The setsid() call is used to detach the process from the parent (normally a shell).
3.  The file mask should be reset. The reason for this is because we want to create new files with the mask that is needed for the child process.
4.  The current directory should be changed to something benign. We may not want the child to be in the same pwd as the parent.
5.  The standard files (stdin,stdout and stderr) need to be reopened.


## Describe ways of process inter-communication
1.  POSIX mmap
	1. memory-mapped file containing the shared-memory object
2.  Message queues
3.  Semaphores
	1. Semaphore is used to protect any resources such as Global shared memory that needs to be accessed and updated by many processes simultaneously.
	2. Whenever a process needs to access the resource, it first needs to take permission from the semaphore. Semaphore give permission to access a resource if resource is free otherwise process has to wait.
4.  Shared memory
	1. The fastest way for processes to communicate is directly, through a shared segment of memory. A common memory area is added to the address space of sharing processes.
5.  Anonymous pipes
	1. Anonymous pipes is Unidirectional.
	2. One process to another
	3. Local only
	4. No ownership
6.  Named pipes
	1. Named Pipe can be used one-way or two-way(Duplex) Communication.
	2. Can be used between processes or over the network
	3. Multiple processes can use the same pipe
	4. Security and ownership included
7.  Unix domain sockets
	1. Data is exchanged between programs directly in the operating system’s kernel via files on the host filesystem. To send or receive data using domain sockets, programs read and write to their shared socket file, bypassing network based sockets and protocols entirely.
		1. `SOCK_STREAM` (compare to [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol "Transmission Control Protocol")) – for a stream-oriented socket
		2. `SOCK_DGRAM` (compare to [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol "User Datagram Protocol")) – for a datagram-oriented socket that preserves message boundaries (as on most UNIX implementations, UNIX domain datagram sockets are always reliable and don't reorder datagrams
		3. `SOCK_SEQPACKET` (compare to [SCTP](https://en.wikipedia.org/wiki/SCTP "SCTP")) – for a sequenced-packet socket that is connection-oriented, preserves message boundaries, and delivers messages in the order that they were sent
8. RPC
	1. Remote procedure call (RPC) is an Inter-process communication technology that allows a computer program to cause a subroutine or procedure to execute in another address space (commonly on another computer on a shared network) without the programmer explicitly coding the details for this remote interaction.
	2. Programmer doesn't have to account for the fact the function call is remote.

## Fork vs Exec
fork() creates a new process by duplicating the calling process, The new process, referred to as child, is an exact duplicate of the calling process, referred to as parent, except for the following :

1.  The child has its own unique process ID (PID)
2.  The child’s parent process ID is the same as the parent’s process ID.
3.  The child does not inherit its parent’s memory locks and semaphore adjustments.
4.  The child does not inherit outstanding asynchronous I/O operations from its parent nor does it inherit any asynchronous I/O contexts from its parent.

Because the two processes are now running exactly the same code, they can tell which is which by the return code of `fork` - the child gets 0, the parent gets the PID of the child.

The `exec` call is a way to basically **replace the entire current process with a new program**. It loads the program into the current process space and runs it from the entry point.

Shells typically do this whenever you try to run a program like `find` - the shell forks, then the child loads the `find` program into memory, setting up all command line arguments, standard I/O and so forth.

### Describe how processes executes in a Unix shell

Let’s take the example of /bin/ls. When you run ‘ls’ the shell searches in its path for an executable named ‘ls, when it finds it, the shell will forks off a copy of itself using the fork system call. If the fork succeeds, then in the child process the shell will run ‘exec /bin/ls’ which will replace the copy of the child shell with itself. Any parameters that that are passed to ‘ls’ are done so by exec.

## What are Unix Signals?

Signals are an inter process communication method. The default signal in Linux is SIG-TERM. SIG-KILL cannot be ignored and causes an application to be forcefully killed. Use the ‘kill’ command to send signals to a process. Another popular signal is the ‘HUP’ signal which is used to ‘reset’ or ‘hang up’ applications

The signals SIGKILL and SIGSTOP cannot be caught, blocked, or  ignored.

### When you send a HUP signal to a process, you notice that it has no impact, what could have happened?

During critical section execution, some processes can setup signal blocking. The system call to mask signals is ‘sigprocmask’. When the kernel raises a blocked signal, it is not delivered. Such signals are called pending. When a pending signal is unblocked, the kernel passes it off to the process to handle. It is possible that the process was masking SIGHUP.

## Process Models
What are the advantage of different server process models: threaded vs process vs pure async (eg. nodejs) vs hybrid (async + threadpool) 
- Async are non-blocking operations within a single thread.  eg. OS reads a file, and while waiting executes some math equation, then goes back when the file is ready.
- Threaded are concurrent processes (not in Python).
	- Share same memory space
- Processes do not share the same memory space and are separate
