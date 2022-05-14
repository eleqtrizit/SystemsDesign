# Linux Commands

## CLIs
### netstat
 Networking tool being used for troubleshooting and configuration and used to display all network connections on a system. It simply provides a way to check whether various aspects of TCP/IP are working and what connections are present.

Check all listening ports
`netstat -ntlp`
 
### ping
Check connection status between source and destination.  Useful for name resolution, too.

### du
Check size of a directory
`du -sh /var/log/*`

### fdisk
Create partitions

### wc
Count number of characters or lines in a file

### grep
Search for a string in file(s)

### env
Print list of environment variables.  Can also be used to launch programs in different enviroments without modifying the current one.

### pwd
Show path

### Add new user and set password
`useradd smith; passwd smith`

### Check memory
The command used mostly to check memory status in Linux is “free”. Other commands that can be used are given below: 
-   **“cat” command:** It can be used to show or display Linux memory information. 
	- `cat /proc/meminfo`
-   **“vmstat” command:** It can be used to report statistics of virtual memory. 
-   **“top” command:** It can be used to check the usage of memory. 
-   **“htop” command:** It can be used to find the memory load of each process.

#### vmstat
```
Procs
    r: The number of processes waiting for run time.
    b: The number of processes in uninterruptible sleep.
Memory
    swpd: the amount of virtual memory used.
    free: the amount of idle memory.
    buff: the amount of memory used as buffers.
    cache: the amount of memory used as cache.
    inact: the amount of inactive memory. (-a option)
    active: the amount of active memory. (-a option)
Swap
    si: Amount of memory swapped in from disk (/s).
    so: Amount of memory swapped to disk (/s).
IO
    bi: Blocks received from a block device (blocks/s).
    bo: Blocks sent to a block device (blocks/s).
System
    in: The number of interrupts per second, including the clock.
    cs: The number of context switches per second.
CPU
    These are percentages of total CPU time.
    us: Time spent running non-kernel code. (user time, including nice time)
    sy: Time spent running kernel code. (system time)
    id: Time spent idle. Prior to Linux 2.5.41, this includes IO-wait time.
    wa: Time spent waiting for IO. Prior to Linux 2.5.41, included in idle.
    st: Time stolen from a virtual machine. Prior to Linux 2.6.11, unknown.
```

##### Buffers vs Cache
Buffers are associated with a specific block device, and cover caching of filesystem metadata as well as tracking in-flight pages. The cache only contains parked file data. That is, the buffers remember what's in directories, what file permissions are, and keep track of what memory is being written from or read to for a particular block device. The cache only contains the contents of the files themselves.

### pipe
Basically a form of redirection that is used to send the output of one command to another command for further processing

### unmask
Unmask, also known as user file-creation mask, is a Linux command that allows you to set up ***default permissions*** for new files and folders that you create. In Linux OS, unmask command is used to set default file and folder permission. It is also used by other commands in Linux like mkdir, tee, touch, etc. that create files and directories.

### dmesg
View boot logs

### updatedb
The **updatedb** utility is part of **_mlocate_**. It examines the entire file system and accordingly creates the database for the **locate** command


### First Line of Bash Script
`!#/bin/bash`

### lspci
List devices and their drivers, physical locations

### lshw
List detailed information about hardware configurations as **root** user.  Some flags can be used to see details information about the networking hardware.

### lsmod
List kernel modules

### insmod (or modprobe)
Load one kernel module.  Can also use **modprobe** instead

### rsync
Transfer files either to or from a remote host

### export
Marks enviroment variables to be shared in forked processes.  Otherwise, variables only apply to current environment.

### set
Display, set or unset values of shell attributes and positional parameters.
Debug script: `set -x`
Set positional arguments
```
>set red blue green
>echo $1 $2 $3
red blue green

#unset 
>set --
```

Don't ignore when pipes fail
`set -eo pipefail`

### umask
unmask stands for user file creation mode. When the user creates any file, it has default file permissions. So unmask will specify few restrictions to the newly created file (it controls the file permissions).


### Reduce or shrink the size of the LVM partition?
Below are the logical steps to reduce the size of the LVM partition:
-   Unmount the file system using the **unmount** command
-   Use the **resize2fs** command as follows:

`resize2fs /dev/mapper/myvg-mylv 10G`

Then, use the **lvreduce** command as follows:
`lvreduce -L 10G /dev/mapper/myvg-mylv`

### repquota
*uncommon question unless you expect many user work spaces*
check the status of a user’s defined quota, along with the disk space and the number of files used.

## Different types of modes used in VI
![[Pasted image 20220416073730.png]]
* Command Mode - from Escape
* Insert/Edit Mode - from 'i'
* Execution/Replacement - from ':'

## Networking Specific
### ssh-copy-id
Install your public key in a remote machine's authorized_keys.

### ssh-keygen
Generate ssh keys used for authentication, password-less logins, and other things.

### ip
Identify network interfaces 
`ip a`

Add temporary address to device 
`sudo ip addr add 10.102.66.200/24 dev enp0s25`

Set link up or down
```
ip link set dev enp0s25 up
ip link set dev enp0s25 down
```

Add route to interface
```
ip route add default via 10.102.66.1
```

Show route
```
ip route show
```

### ethtool
Displays and changes Ethernet card settings such as auto-negotiation, port speed, duplex mode, and Wake-on-LAN.

### Restart Networking
```
sudo /etc/init.d/networking restart
# or
sudo systemctl restart networking
```

### Network Status
```
# sudo /etc/init.d/networking status
or
# sudo systemctl status networking
```

## Netplan
The way Ubuntu and some others do things

Apple netplan `netplan apply`

