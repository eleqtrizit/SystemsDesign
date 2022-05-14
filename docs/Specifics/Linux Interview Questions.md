**Are Linux commands case sensitive (easy red flag question)?**
Yes

**Location of system logs**
/var/log/messages

**Command to view boot logs?**
dmesg

**Can you legally edit the Linux kernel code?**
It is open source software under GPL2.  Anyone can edit and make changes.

**Where is swap space located and what is it used for?**
It is located in a dedicated swap partition.  It is used to extend the available memory to start new processes, by storing infrequently used pages there.

**What is the difference between memory buffers and cache?**
Buffers remember what's in directories, what file permissions are, and keep track of what memory is being written from or read to for a particular block device. The cache only contains the contents of the files themselves.

**What is the difference between hard and symbolic links?**
Hard Links
* Points to another file's inode directly (the physical location)
* Acts like an additional name to the same file
* Cannot be used across file systems.

Soft Links
- Points to another filename
- if original filename is deleted, softlink becomes broken
- Can be used across file names

**You see the load average of a system and the numbers are 0.5 1 2.  What does this mean?**
For the past minute, 0.5 processes were waiting for the cpu.  In the past minutes, an average of one process was waiting.  For the past 15, an average of 2 processes were waiting.

**What is the first process that is started by the kernel in Linux and what is its process id?**
The first process started by the kernel in Linux is “init” and its process id is 1.

**A large file is taking up all the space on a partition.  You delete it, yet the partition is still full.  Why?**
There is a process holding the file handle open.  It needs to be stopped before the space can be freed.

**What command can you use to show which process is holding the file open?**
lsof

**What types of files/directories start with '.'?**
Hidden

**What is the pwd command?**
Shows current path

**What are the three permissions under Linux?**
Read, Write, Execute

**How are the three permissions applied on the system?**
User, Group, Everyone/Other/All

**What command is used to set permissions?**
chmod

**What is the purpose of a sticky bit?**
Prevents anyone who is not the owner, owning group or root from removing or renaming a file

**What command do you use to change file/directory ownership?**
chown

Should you use telnet to administrate a Linux server?
No, it is not encrypted.

**Differences between Cron and Anacron?**
-   A Cron job can be scheduled by any normal user, while Anacron can be scheduled only by a superuser
-   Cron expects the system to be up and running, while Anacron doesn’t expect this all the time. In the case of Anacron, if a job is scheduled and the system is down at this time, it will execute the job as soon as the system is up and running.
-   Cron is ideal for servers, Anacron is ideal for both desktops and laptops.
-   Cron should be used when we want a job to be executed at a particular hour and minute, while Anacron should be used when the job can be executed at any time.

**What is a Zombie process?**
It's a dead or finished process that still has an entry in the process table.  It's parent hasn't read the exit status.

**What commands do you run to kill a process?**
ps and kill.  Maybe some sudo if you don't have the rights.

**Tell me about the /proc file system**
The /proc file system is a RAM based file system which maintains information about the current state of the running kernel including details on CPU, memory, partitioning, interrupts, I/O addresses, DMA channels, and running processes. This file system is represented by various files which do not actually store the information, they point to the information in the memory. The /proc file system is maintained automatically by the system

Explain key based authentication (more details the better)
- The generation of private/public air `ssh-keygen`
- Which key is transferred to the remote host (public) via  `ssh-copy-id` (or manually)
- The file where public keys are stored `authorized_keys`
- Proper permissions for authorized_keys `chmod 600 ~/.ssh/authorized_keys`

**What does lspci do?**
The lspci command displays information about PCI buses and the devices attached to your system

**What are the different types of modes in VI editor?**
* Command Mode - from Escape
* Insert/Edit Mode - from 'i'
* Execution/Replacement - from ':'

**Name the four Configuration Management Tools used in UNIX-like operating systems.**
-   Ansible
-   Chef
-   Puppet
-   CFEngine


Logging monitoring metric tracing