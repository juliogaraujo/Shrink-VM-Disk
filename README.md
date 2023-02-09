
> Under progress....

# Method to shrink virtual machine disk

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

> The procedure can be used for any other type of environment, please be aware to adapt the steps.

## Steps

1. Create a new disk with smaller size via Proxmox GUI.  
webGUI: Node -> Virtual Machine -> Hardware -> Add -> Hard Disk.  
~~~

~~~

2. Insert in the virtual CD/DVD an iso image of Ubuntu Desktop or any other linux dist that can contains parted and dd command.  


> GParted can be used to reduce and create the partitions, but for simplist this procedure is going to use terminal command parted. 
 

3. Use dd command to copy the partitions:  
~~~
$ sudo dd if=/dev/sda1 of=/dev/sda2
$ sudo dd if=/dev/sda2 of=/dev/sda2
~~~
> This process might take long time.

## References
https://pve.proxmox.com/pve-docs/pve-admin-guide.html  
https://pve.proxmox.com/wiki/Storage
