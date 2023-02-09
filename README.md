# Method to shrink virtual machine disk

## Problem Description
There are cases where it might be need to reduce the vm-disks because the lack of storage space for the server.  
For these cases the procedure below can be used to shrink the vm-disks, but keep in mind that you might need to have at least enough space left in the server storage to build a new smaller disk.

## WARNING
This procedure might cause data loss, please it is strongly recommended to perform a backup before start the procedure.  
Perform this procedure at your own risk.

## Procedure Environment
- Proxmox 7.3-4
- Zabbix installed at one Virtual Machine with 4 vCPU, 8Gb MEM, 120Gb Hard Disk.
- VMID = 102

> The procedure can be used for any other type of environment, please be aware to adapt the steps.

## Steps

> **NOTE** Change the size of the disk to fit your need

### 1. Create a new disk with smaller size via Proxmox GUI.  
- webGUI: Node -> Virtual Machine -> Hardware -> Add -> Hard Disk.  

**Check biggest virtual disk info**  
~~~ 
sudo qemu-img info /dev/pve/vm-102-disk-0
~~~
~~~
image: /dev/pve/vm-102-disk-0
file format: raw
virtual size: 120 GiB (128849018880 bytes)
disk size: 0 B
~~~
**Check the new smaller virtual disk info**
~~~
sudo qemu-img info /dev/pve/vm-102-disk-1
~~~
~~~
image: /dev/pve/vm-102-disk-1
file format: raw
virtual size: 62 GiB (66571993088 bytes)
disk size: 0 B
~~~

### 2. Insert in the virtual CD/DVD an iso image of Ubuntu Desktop or any other linux distro.  

**The distro should contains: dd, fdisk, parted, e2fsck, resize2fs command availabe**
- webGUI: Node -> Virtual Machine -> Hardware -> CD/DVD Drive -> Edit -> Select ISO image file.  
- Start the VM and select to boot via CD/DVD.  
- If asked click at **Try Ubuntu** 
- Open the terminal.

> GParted can be used to reduce and create the partitions, but for simplest this procedure is going to use terminal command parted. 

**Check the available disk:**
~~~
ls -l /dev/sd*
~~~
~~~
brw-rw---- 1 root disk 8,  0 fev  9 00:51 /dev/sda
brw-rw---- 1 root disk 8,  1 fev  9 00:51 /dev/sda1
brw-rw---- 1 root disk 8,  2 fev  9 00:51 /dev/sda2
brw-rw---- 1 root disk 8, 16 fev  9 00:51 /dev/sdb
~~~
> The new empty disk is **/dev/sdb**

### 3. Reduce the partition of the biggest disk.  

**Check disk info**
~~~
sudo parted /dev/sda print free
~~~
~~~
Model: ATA QEMU HARDDISK (scsi)
Disk /dev/sda: 129GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
        17.4kB  1049kB  1031kB  Free Space
 1      1049kB  2097kB  1049kB                     bios_grub
 2      2097kB  129GB   129GB   ext4
        129GB   129GB   1032kB  Free Space
~~~

**Reducing the biggest disk to 61G, should be less than the new disk!**  
Check file system for errors and (if possible) fix them
~~~
sudo e2fsck -f -y -v -C 0 /dev/sda2
~~~
~~~
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure                                           
Pass 3: Checking directory connectivity                                        
Pass 4: Checking reference counts
Pass 5: Checking group summary information                                     
                                                                               
      142753 inodes used (1.82%, out of 7864320)
         808 non-contiguous files (0.6%)
         133 non-contiguous directories (0.1%)
             # of inodes with ind/dind/tind blocks: 0/0/0
             Extent depth histogram: 134290/188
     3230728 blocks used (10.27%, out of 31456512)
           0 bad blocks
           2 large files

      118070 regular files
       16218 directories
           8 character device files
           0 block device files
           0 fifos
         146 links
        8445 symbolic links (8256 fast symbolic links)
           3 sockets
------------
      142890 files
~~~
shrink file system
~~~
sudo resize2fs -p /dev/sda2 61G
~~~
~~~
resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/sda2 to 16252928 (4k) blocks.
Begin pass 3 (max = 960)
Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
The filesystem on /dev/sda2 is now 16252928 (4k) blocks long.
~~~
Delete and recreate the partition the file system with the required amount.
~~~
$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sda: 120 GiB, 128849018880 bytes, 251658240 sectors
Disk model: QEMU HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 394868BC-2A77-4103-8E70-92E9218DBD7D

Device     Start       End   Sectors  Size Type
/dev/sda1   2048      4095      2048    1M BIOS boot
/dev/sda2   4096 251658206 251654111  120G Linux filesystem

Command (m for help): d
Partition number (1,2, default 2): 

Partition 2 has been deleted.

Command (m for help): n
Partition number (2-128, default 2): 
First sector (4096-251658206, default 4096): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4096-251658206, default 251658206): +61G

Created a new partition 2 of type 'Linux filesystem' and of size 62 GiB.
Partition #2 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: n

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
~~~

**Check disk info after resize**
~~~
sudo parted /dev/sda print free
~~~
~~~
Model: ATA QEMU HARDDISK (scsi)
Disk /dev/sda: 129GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  Flags
        17.4kB  1049kB  1031kB  Free Space
 1      1049kB  2097kB  1049kB                     bios_grub
 2      2097kB  66.6GB  66.6GB  ext4
        66.6GB  129GB   62.3GB  Free Space
~~~

### 4. Create the partitions at new disk.
~~~
$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x7676eeab.

Command (m for help): g
Created a new GPT disklabel (GUID: 9B268DB2-EED8-C54F-BDB8-30467C836073).

Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-130023390, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-130023390, default 130023390): +1M

Created a new partition 1 of type 'Linux filesystem' and of size 1 MiB.

Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): 4
Changed type of partition 'Linux filesystem' to 'BIOS boot'.

Command (m for help): n
Partition number (2-128, default 2): 
First sector (4096-130023390, default 4096): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4096-130023390, default 130023390): 

Created a new partition 2 of type 'Linux filesystem' and of size 62 GiB.

Command (m for help): t
Partition number (1,2, default 2): 
Partition type or alias (type L to list all): 20

Changed type of partition 'Linux filesystem' to 'Linux filesystem'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
~~~
Format the partition to ext4
~~~
sudo mkfs.ext4 /dev/sdb2
~~~

### 5. Use dd command to copy the partitions:  
Copy boot partition
~~~
sudo dd if=/dev/sda1 of=/dev/sdb1
~~~
Copy data partition
~~~
sudo dd if=/dev/sda2 of=/dev/sdb2
~~~
> This process might take long time. Use the cmd **kill -USR1 {dd pid}** to check the progress.

Copy **only** the MBR.
~~~
sudo dd if=/dev/sda of=./mbrsda.bkp bs=512 count=1  
sudo dd if=./mbrsda.bkp of=/dev/sdb bs=446 count=1
~~~

## References
https://pve.proxmox.com/pve-docs/pve-admin-guide.html  
https://pve.proxmox.com/wiki/Storage  
https://access.redhat.com/articles/11963333  
https://www.cyberciti.biz/faq/howto-copy-mbr  
