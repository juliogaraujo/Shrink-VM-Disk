
> Under progress....

# Metod to shrink virtual machine disk

## Problem Description
There are cases where it might be need to reduce the vm-disks because the lack of storage space for the server
For these cases the procedure below can be used to shrink the vm-disks, but keep in mind that you might need to have 
at least enough space left in the server storage to build a new smaller disk.

## WARNING
This procedure might cause data loss, please it is strongly recommended to perform a backup before start the procedure.  
Perform this procedure at your own risk.

## Procedure Environment
- Proxmox 7.3-4
- Zabbix installed at one Virtual Machine with 4 vCPU, 8Gb MEM, 120Gb Hard Disk.

> The procedure can be used for any other type of environmen, please be aware to adapt the steps.

## Steps

1. Create a new disk with smaller size via Proxmox GUI.  

 

2. Insert in the virtual CD an iso image of Ubuntu Desktop or any linux distru that can use Gparted.  
- Open GParted and reduce the partion of the big disk for a size that fit over to the smaller disk.  
- Select the smaller disk and create the partion with the same order and size from the bigest one. Note: The last partition should be smaller.  

3. Use dd command to copy the partitions:  
`$ sudo dd if=/dev/sda1 of=/dev/sda2`  
`$ sudo dd if=/dev/sda2 of=/dev/sda2`  
> This process might take long time.

## Notes
e2fsck -f -y -v -C 0 /dev/sda2  
resize2fs -p /dev/sda2 65933312K

