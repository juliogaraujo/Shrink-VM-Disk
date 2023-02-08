
> The procedure DOES NOT WORK!

# Metod to shrink virtual machine disk

## Problem Description
There are cases where it might be need to reduce the vm-disks because the lack of storage space for the proxmox server.
For these cases the procedure below can be used to shrink the vm-disks, but keep in mind that you might need to have 
at least enough space left in the proxmox storage to build a new smaller disk.
This procedure has been tested with a Zabbix installed in the Proxmox Virtual Machine, and the disk was recuded from 
120Gb to 62Gb.

## WARNING
This procedure might cause data loss, please be aware that there is no ga
You understand the possible dangers and data loss that this procedure might cause and have been warned about them

1. Create a new disk with smaller size via Proxmox GUI
`#check via cmd both disks
$ sudo qemu-img info /dev/pve/vm-102-disk-0
image: /dev/pve/vm-102-disk-0
file format: raw
virtual size: 120 GiB (128849018880 bytes)
disk size: 0 B

$ sudo qemu-img info /dev/pve/vm-102-disk-1
image: /dev/pve/vm-102-disk-1
file format: raw
virtual size: 62 GiB (66571993088 bytes)
disk size: 0 B`

2. Insert in the virtual CD an iso image of Ubuntu Desktop or any linux distru that can use Gparted.
- Open GParted and reduce the partion of the big disk for a size that fit over to the smaller disk.
- Select the smaller disk and create the partion with the same order and size from the bigest one. Note: The last partition should be smaller.

3. Use dd command to copy the partitions:
`$ sudo dd if=/dev/sda1 of=/dev/sda2
$ sudo dd if=/dev/sda2 of=/dev/sda2`
> This process might take long time.

## Notes
e2fsck -f -y -v -C 0 '/dev/sda2'
resize2fs -p '/dev/sda2' 65933312K

