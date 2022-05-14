# File Systems

## List open file handles

-   lsof -p process-id
-   Or ls /proc/process-id/fd

## What is an inode?

An inode is a data structure in Unix that contains metadata about a file. Some of the items contained in an inode are:  
1) mode  
2) owner (UID, GID)  
3) size  
4) atime, ctime, mtime  
5) acl’s  
6) blocks list of where the data is

The filename is present in the parent directory’s inode structure.

### What is the difference between atime, mtime and ctime?

atime is the last time a file was accessed. For instance if you opened the file to view it.

mtime is the last time a file was modified. For instance you changed some text in the file, if it’s a text file.

ctime is the last time the inode contents were changed of a file, for instance the mode, or owner.

## What is the difference between a soft link and a hard link?

1) Hardlink shares the same inode number as the source link. Softlink has a different inode number. Example:

$ touch a
```
$ ln a b  
$ ls -i a b  
24 a 24 b  
$ ln -s a c  
$ ls -i a c  
24 a 25 c
```

2) In the data portion of the softlink is the name of the source file  
3) Hardlinks are only valid in the same filesystem, softlinks can be across filesystems

### When would you use a hardlink over a softlink?

A hardlink is useful when the source file is getting moved around, because renaming the source does not remove the hardlink connection. On the other hand, if you rename the source of a softlink, the softlink is broken. This is because hardlink’s share the same inode, and softlink uses the source filename in it’s data portion.

## Describe LVM and how it can be helpful

LVM stands for logical volume manager and it is a way of grouping disks into logical units. 

The basic unit of LVM is a PE or a physical extent. One disk may be divided into one or more PE’s. One or more PE’s are contained in a VG or a volume group. Or or more LV or logical volumes are created out of a VG. 

Example
- if we have a server with 2x1TB disk drives, we can create 4xPE’s on it, each one being 500GB. 
- On disk 1 let’s say we name the PE’s PE1 and PE3 and on disk 2 we name the PE’s PE2 and PE4. 
- We can then create VG0 out of PE1 and PE2, and VG1 out of PE3 and PE4. 
- After that we can create a LV called /root and another one called swap on VG0.

An advantage of using LVM is that we can create ‘software’ RAID, i.e., we can join multiple disks into one bigger disk. **We cannot select the RAID level with LVM**, for instance we cannot say that a VG is of RAID 5 type, however we are able to pick and chose the different PE’s we want in a VG. Also LVM allows for dynamically growing a disk.

## What is ‘md’ and how do you use it?

MD is Linux software RAID. RAID can be done either in hardware wherein there is a RAID controller that does RAID and presents a logical volume to the OS, or RAID can be done in software wherein the kernel has a RAID driver which takes one or more disks can does RAID across them. ‘MD’ refers to the software RAID component of Linux.


## If a filesystem is full, and you see a large file that is taking up a lot of space, how do make space on the filesystem?

1) If no process has the filehandle open, you can delete the file  
2) If a process has the filehandle open, it is better if you do not delete the file, instead you can ‘cp /dev/null’ on the file, which will reduce it’s size to 0.  
3) A filesystem has a reserve, you can reduce the size of this reserve to create more space using tunefs.

## What is the difference between character device and block device?

Block devices are generally buffered and are read/written to in fixed sizes, for instance hard drives, cd-roms. 

Characters devices read/writes are one character at a time, such as from a keyboard or a tty, and are not buffered.
