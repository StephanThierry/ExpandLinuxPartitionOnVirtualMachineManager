# Expand Linux Partition On VM in Synology Virtual Machine Manager
How to Expand a Linux (Ubuntu) Partition On a VM in the Synology Virtual Machine Manager

**Making the space available to the OS:**
1. Login to DSM and goto Virtual Machine Manager  
1. Select "Virtual Machine" and select the Virtual Machine  
1. At the top select [Action] - Edit  
1. In the "Edit Virtual Machine"-window select the "Storage"-tab  
1. Set the size in GB of the Partition you want to increase - for example: "Virtual Disk 1" from 10 to 20 GB  
  
**Using the space in the OS:**  
* Login to the instance as your admin account using a terminal emulator. (For example Putty)
* Use command:  
`sudo lsblk`

You should now see something like this:  
```
	NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
	loop0                       7:0    0  40.9M  1 loop /snap/snapd/20290
	loop1                       7:1    0  63.5M  1 loop /snap/core20/2015
	loop2                       7:2    0  40.8M  1 loop /snap/snapd/20092
	loop3                       7:3    0 111.9M  1 loop /snap/lxd/24322
	loop4                       7:4    0  63.4M  1 loop /snap/core20/1974
	sda                         8:0    0    20G  0 disk
	├─sda1                      8:1    0     1M  0 part
	├─sda2                      8:2    0   1.8G  0 part /boot
	└─sda3                      8:3    0   8.2G  0 part
	  └─ubuntu--vg-ubuntu--lv 253:0    0   8.2G  0 lvm  /
	sr0                        11:0    1  1024M  0 rom
	sr1                        11:1    1  47.4M  0 rom
```
Notice! The "sda"-drive is expanded to 20GB but the space is not yet fully used by the partitions.

Lets assume the partition we want to expand is the one mounted to "/" - so that would be "ubuntu--vg-ubuntu--lv" of type "lvm" - Logical volume manager  

* Type command:  
`sudo growpart /dev/sda 3` - meaning the 3rd partition of "sda"
   
* Type command:  
`sudo lvdisplay` to get the LV Path of the logical volume

You should see something like:  
```
	  --- Logical volume ---
	  LV Path                /dev/ubuntu-vg/ubuntu-lv
	  LV Name                ubuntu-lv
	  VG Name                ubuntu-vg
	  LV UUID                vCsYuG-wvCf-pDGd-omCG-LJ94-Y3ck-p1EtDM
	  LV Write Access        read/write
	  LV Creation host, time ubuntu-server, 2023-06-08 15:01:13 +0000
	  LV Status              available
	  # open                 1
	  LV Size                <18.25 GiB
	  Current LE             4671
	  Segments               1
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     256
	  Block device           253:0
```

* Look for "LV path" and use that to form this command to extend the volume:  
`lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv`
* You now need to expand the filesystem - so you need the filesystem path. Type command:  
`df -h`
You should see something like this:
```
  Filesystem                         Size  Used Avail Use% Mounted on
  tmpfs                              297M  8.5M  289M   3% /run
  /dev/mapper/ubuntu--vg-ubuntu--lv    8G  7.7G  0.3G  96% /
  tmpfs                              1.5G     0  1.5G   0% /dev/shm
  tmpfs                              5.0M     0  5.0M   0% /run/lock
  tmpfs                              1.5G     0  1.5G   0% /run/qemu
  /dev/sda2                          1.7G  129M  1.5G   8% /boot
  tmpfs                              297M  4.0K  297M   1% /run/user/1000
```

Look for the filesystem mounted on `/` - in this case `/dev/mapper/ubuntu--vg-ubuntu--lv` and use that referance in this command:   
`sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv`  

* Run `df -h` again and you should now see that the filesystem has been expanded to use the full available size.

```
	Filesystem                         Size  Used Avail Use% Mounted on
	tmpfs                              297M  8.5M  289M   3% /run
	/dev/mapper/ubuntu--vg-ubuntu--lv   18G  7.7G  9.5G  45% /
	tmpfs                              1.5G     0  1.5G   0% /dev/shm
	tmpfs                              5.0M     0  5.0M   0% /run/lock
	tmpfs                              1.5G     0  1.5G   0% /run/qemu
	/dev/sda2                          1.7G  129M  1.5G   8% /boot
	tmpfs                              297M  4.0K  297M   1% /run/user/1000
```
