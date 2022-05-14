# Linux OS Internals

mount fstab lvm lspci lspof top
how to find file system type
uefi


![[Pasted image 20220416071333.png]]

-   **Kernel:** It is considered a core or main part of Linux and is generally responsible for all major activities of OS such as process management, device management, etc.  
-   **System Library:** These are special functions or programs with the help of which application programs or system utilities can access features of the kernel without any requirement of code. It is simply used to implement the functionality of the OS. 
-   **System Utility:** These are utility programs that are responsible to perform specialized and individual-level tasks. They are considered more liable and allow users to manage the computer.  
-   **Hardware:** It is physical hardware that includes items such as a mouse, keyboard, display, CPU, etc. 
-   **Shell:** It is an environment in which we can run our commands, shell scripts, and programs. It is an interface between user and kernel that hides all complexities of functions of the kernel from the user. It is used to execute commands.

## What is Kernel? Explain its functions.

A kernel is considered the main component of Linux OS. It is simply a resource manager that acts as a bridge between hardware and software. Its main role is to manage hardware resources for users and is generally used to provide an interface for user-level interaction. A kernel is the first program that is loaded whenever a computer system starts. It is also referred to as low-level system software.

Its other main functions include: 

-   Memory Management
-   Process Management
-   Device Management
-   Storage Management
-   Manage access, and use of various peripherals that are connected to the computer.

## What are two types of Linux User Mode?
-   Command Line 
-   GUI

## What is LILO and Grub
**LILO** (Linux Loader) is basically a bootloader for Linux that is used to load Linux into memory and start the OS.  Only for Linux

**GRUB** 
- Supports all operating systems
- Allows selecting of specific Linux Kernel

## What is swap space?
Swap space, as the name suggests, is basically a space on a hard disk that is used when the amount of physical memory or RAM is full. It is considered a substitute for physical memory.

## Process States in Linux
![[Pasted image 20220416072303.png]]

-   **New/Ready:** In this state, a new process is created and is ready to run.
-   **Running:** In this state, the process is being executed.
-   **Blocked/Wait:** In this state, the process is waiting for input from the user and if doesn't have resources to run such as memory, file locks, input, then it can remain in a waiting or blocked state.
-   **Terminated/Completed:** In this state, the process has completed the execution or terminated by the OS.
-   **Zombie:** In this state, the process is terminated but information regarding the process still exists and is available in the process table.

## Linux Shell
Linux shell is a user interface present between user and kernel. It is used for executing commands and communication with Linux OS. Linux shell is basically a program used by users for executing commands. It accepts human-readable commands as input and converts them into kernel understandable language

![[Pasted image 20220416073649.png]]


## Maximum length for a filename
255 bytes

## Maximum length for a filename
Double memory size

## File Permissions in Linux
**Read (r):** It allows the user to open and read the file or list the directory. 
**Write (w):** It allows the user to open and modify the file. One can also add new files to the directory. 
**Execute (x):** It allows the user to execute or run the file.  One can also lookup a specific file within a directory.

## Automatically mount file systems
/etc/fstab

## LVM
LVM (Logical Volume Management) is basically a tool that provides logical volume management for the Linux kernel.   It is an abstraction layer and can do
- Allocate disks
- strip/mirror
- resizing (grow and reduce)

![[Pasted image 20220416074622.png]]

## “/proc” file system
Proc file system is a pseudo or virtual file system that provides an interface to the kernel data structure. It generally includes useful information about processes that are running currently. It can also be used to change some kernel parameters at runtime or during execution. It is also regarded as a control and information center for the kernel. All files under this directory are named virtual files.

## Daemons
Background processes with no controlling terminal.  Generally started at boot and die at shutdown.  Main purpose is to handle requests and forward them to some program for execution.

# Exec
The `exec` call is a way to basically **replace the entire current process with a new program**. It loads the program into the current process space and runs it from the entry point.

Shells typically do this whenever you try to run a program like `find` - the shell forks, then the child loads the `find` program into memory, setting up all command line arguments, standard I/O and so forth.

## Zombie Process
Zombie Process, also referred to as a defunct or dead process in Linux, is a process that has finished the execution, but its entry remains in the process table.  Happens when the parent hasn't read the child's status and reaped it.  Zombie Processes don't use any system resources are just dead entries in the process table.

## Difference between cron and anacron?
**Cron:** It is a program in Linux that is used to execute tasks at a scheduled time. It works effectively on machines that run continuously.   
  
**Anacron:** It is a program in Linux that is used to execute tasks at certain intervals. It works effectively on machines that are powered off in a day or week.
![[Pasted image 20220416080307.png]]

## Load average
From uptime and top:
`load average: 1.05, 0.70, 5.09`

This means:
- load average over the last 1 minute: 1.05
	- The computer was overloaded by 5% on average. On average, .05 processes were waiting for the CPU. (1.05)
- load average over the last 5 minutes: 0.70
	- The CPU idled for 30% of the time. (0.70)
- load average over the last 15 minutes: 5.09
	- The computer was overloaded by 409% on average. On average, 4.09 processes were waiting for the CPU. (5.09)

## NODE and Process Id
**INODE:** It is a unique name given to each file by OS. Each inode has a unique inode number within a file system. It stores various information about files in Linux such as ownership, file size, file type, access mode, number of links, etc.    
  
**Process Id (Identifier):** It is a unique Id given to each process. It is simply used to uniquely identify an active process throughout the system until the process terminates.

## First process that is started by the kernel in Linux and what is its process id
The first process started by the kernel in Linux is “init” and its process id is 1.


## Is there any relation between the modprobe.conf file and network devices?

Yes, this file assigns a kernel module to each network device.


### Difference between ext2 and ext3 file systems?
-   The ext3 file system is an enhanced version of the ext2 file system.
-   The most important difference between ext2 and ext3 is that ext3 supports journaling.
	- Reduces need for file check in all but the most dire circumstances.


## Explain Linux Boot Sequence.

**Answer:** There are six levels of a Linux Boot Sequence. These are as follows:

**BIOS:** Full form of BIOS is Basic Input or Output System that performs integrity checks and it will search and load and then it will execute the bootloader.

**MBR:** MBR means Master Boot Record. MBR contains the information regarding GRUB and executes and loads this bootloader.

**GRUB:** GRUB means Grand Unified Bootloader. In case, many kernel images are installed on your system then you can select which one you want to execute.

**Kernel:** Root file system is mounted by Kernel and executes the /sbin/init program.

**Init:** Init checks the file **/etc/inittab** and decides the run level. There are seven-run levels available from 0-6. It will identify the default init level and will load the program.

**Runlevel programs:** As per your default settings for the run level, the system will execute the programs.


## Explain Interrupts
Interrupts means the processor is transferred temporarily to another program or function. When that program is completed, the processor will be given back to that program to complete the task.  Interrupts can come from other programs and devices.

## What is page frame?
A page frame is a block of RAM that is used for virtual memory. It has its page frame number. The size of a page frame may vary from system to system, and it is in the power of 2 in bytes. Also, it is the smallest length block of memory in which an operating system maps memory pages.

## “user virtual address” vs “kernel virtual address"
User programs are run in "user virtual address".  If a program is running in kernel mode it'll use "kernel virtual address".  If the kernel program needs user interaction, it must be translated to user space.

