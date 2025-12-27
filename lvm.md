# Logical Volume Manager in Linux

## What is LVM in Linux?
- LVM stands for logical volume manager. A mechanism that provides an alternative method of managing storage systems than the traditional partition-based one. 
- In LVM, instead of creating partitions, you create logical volumes, and then you can just easily mount those volumes in your filesystem as you'd a disk partition.
- One exception to the previous statement is that you can not use logical volumes for `/boot`. That is because GRUB (the most common bootloader for Linux) can't read from logical volumes.
- The well known alternative to GRUB, systemd-boot on the other hand reads only vfat filesystems, so that's not going to work either.

## Components of LVM
- There are three main components to LVM
  - Physical Volumes
  - Volume Groups
  - Logical Volumes
- Although the list consists of three components, only two of them are direct counterparts to the partitioning system. The following table logs that.

| Disk Partitioning System | LVM |
|--------------------------|---- |
| Partitions | Logical Volumes |
| Disks | Volume Groups |


## Why use LVM?
- The main advantage of LVM is how easy it is to resize a logical volume or volume group. It abstracts away all the ugly parts (partitions, raw disks) and leaves us with a central storage pool to work with.

## Build a test machine with Vagrant
- The below is the *Vagrantfile* content

    ```
    Vagrant.configure "2" do |config|
        config.vm.box = "bento/ubuntu-22.04"
        config.vm.hostname = "ubuntulvmlab0101"
        2.times { |i| config.vm.disk :disk, size: "5GB", name: "drive-#{i}" }
        config.vm.provider :virtualbox do |machine|
            machine.memory = 1024
            machine.cpus = 1
            machine.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
        end
    end
    ```
- Once the Vagrantfile is in place, set the environment variable `VAGRANT_EXPERIMENTAL` to `disks`.
- Finally start the virtual machine using the following command `vagrant up`
- Once the machine is running, you can use `vagrant ssh` to SSH into the machine.

## Hands-on with LVM
- We have built the vagrant machine with three external disks of size 5GB. The size of these disks are arbitrary

    ```
    vagrant@ubuntulvmlab0101:~$ lsblk
    NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    loop0                       7:0    0 33.7M  1 loop /snap/snapd/21761
    loop1                       7:1    0 59.8M  1 loop /snap/core20/2321
    loop2                       7:2    0 77.4M  1 loop /snap/lxd/29353
    sda                         8:0    0   64G  0 disk 
    ├─sda1                      8:1    0    1G  0 part /boot/efi
    ├─sda2                      8:2    0    2G  0 part /boot
    └─sda3                      8:3    0 60.9G  0 part 
      └─ubuntu--vg-ubuntu--lv 253:0    0 30.5G  0 lvm  /
    sdb                         8:16   0    5G  0 disk 
    sdc                         8:32   0    5G  0 disk 
    sdd                         8:48   0    5G  0 disk 
    ```
  
- As you can see from above output, the disks are `sdb`, `sdc`, `sdd`.

### Physical Volumes
- The very first thing you need to know about LVM, is physical volumes. Physical volumes are the raw materials or building blocks that are used to achieve the abstraction that is logical volumes. In simpler words, physical volumes are the logical unit of an LVM system
- A Physical volume can be anything, a raw disk, or a disk partition. Creating and initializing a physical volume are the same thing. Both mean you're just preparing the building blocks for further operations

#### Utilities
- All utilities that manage physical volumes start with the letters **pv** for Physical Volume, E.g: **pvcreate**, **pvchange**, **pvs**, **pvdisplay** etc.

#### Creating Physical volumes
- You can create a physical volume using a raw non-partitioned disk or the partitions themselves.
- We use the **pvcreate** command to create a physical volume. Just pass the device name to it and nothing else

    ```
    vagrant@ubuntulvmlab0101:~$ sudo pvcreate /dev/sdc
      Physical volume "/dev/sdc" successfully created.
    ```
- Now I'll be partitioning the `/dev/sdd` into equal parts. use any tool, `cfdisk`, `parted`, `fdisk` etc.

    ```
    vagrant@ubuntulvmlab0101:~$ sudo parted /dev/sdd
    GNU Parted 3.4
    Using /dev/sdd
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) mklabel gpt                                                      
    (parted) mkpart primary 0% 50%                                            
    (parted) mkpart primary 50% 100%                                          
    (parted) quit                                                             
    Information: You may need to update /etc/fstab.
    
    vagrant@ubuntulvmlab0101:~$ lsblk /dev/sdd                                
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    sdd      8:48   0    5G  0 disk 
    ├─sdd1   8:49   0  2.5G  0 part 
    └─sdd2   8:50   0  2.5G  0 part 
    ```
- You can now quicky create two more physical volumes out of these two partitions in one single step, pass both of these devices to **pvcreate** at once
    ```
    vagrant@ubuntulvmlab0101:~$ sudo pvcreate /dev/sdd1 /dev/sdd2
      Physical volume "/dev/sdd1" successfully created.
      Physical volume "/dev/sdd2" successfully created.
    ```

#### Listing the available physical volumes
- There are three commands that you can use to get the list of the available physical volumes, `pvscan`, `pvs`, and `pvdisplay`. You generally don't need to pass anything to these commands

    ```
    vagrant@ubuntulvmlab0101:~$ sudo pvscan
      PV /dev/sda3   VG ubuntu-vg       lvm2 [<60.95 GiB / 30.47 GiB free]
      PV /dev/sdc                       lvm2 [5.00 GiB]
      PV /dev/sdd1                      lvm2 [<2.50 GiB]
      PV /dev/sdd2                      lvm2 [<2.50 GiB]
      Total: 4 [70.94 GiB] / in use: 1 [<60.95 GiB] / in no VG: 3 [<10.00 GiB]
    vagrant@ubuntulvmlab0101:~$ sudo pvs
      PV         VG        Fmt  Attr PSize   PFree 
      /dev/sda3  ubuntu-vg lvm2 a--  <60.95g 30.47g
      /dev/sdc             lvm2 ---    5.00g  5.00g
      /dev/sdd1            lvm2 ---   <2.50g <2.50g
      /dev/sdd2            lvm2 ---   <2.50g <2.50g
    vagrant@ubuntulvmlab0101:~$ sudo pvdisplay
      --- Physical volume ---
      PV Name               /dev/sda3
      VG Name               ubuntu-vg
      PV Size               <60.95 GiB / not usable 3.00 MiB
      Allocatable           yes 
      PE Size               4.00 MiB
      Total PE              15602
      Free PE               7801
      Allocated PE          7801
      PV UUID               S2nkSD-7RNB-W1sU-M9T4-MM8V-izts-4eSiCA
       
      "/dev/sdc" is a new physical volume of "5.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/sdc
      VG Name               
      PV Size               5.00 GiB
      Allocatable           NO
      PE Size               0   
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               nTf4Kg-51X9-jT65-sIzG-W5E9-SpBQ-GLjbpE
       
      "/dev/sdd1" is a new physical volume of "<2.50 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/sdd1
      VG Name               
      PV Size               <2.50 GiB
      Allocatable           NO
      PE Size               0   
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               M6NKR4-yUFJ-w6Hk-8gE3-ivRG-izsa-mgwfcf
       
      "/dev/sdd2" is a new physical volume of "<2.50 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/sdd2
      VG Name               
      PV Size               <2.50 GiB
      Allocatable           NO
      PE Size               0   
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               1GNr0B-w2AI-V00B-yn22-Cma8-zyA8-6KqgVP
       
    ```

#### Removing a physical volume
- You can remove a physical volume with the `pvremove` command, Just like `pvcreate`, just pass the devices to the command

    ```
    vagrant@ubuntulvmlab0101:~$ sudo pvremove /dev/sdd2
      Labels on physical volume "/dev/sdd2" successfully wiped.
    vagrant@ubuntulvmlab0101:~$ 
    
    vagrant@ubuntulvmlab0101:~$ sudo pvs
      PV         VG        Fmt  Attr PSize   PFree 
      /dev/sda3  ubuntu-vg lvm2 a--  <60.95g 30.47g
      /dev/sdc             lvm2 ---    5.00g  5.00g
      /dev/sdd1            lvm2 ---   <2.50g <2.50g
    ```

---

### Volume Groups
- Volume Groups are collection of physical volumes. It will be the next level of abstraction in LVM. Volume groups are the storage pool that combines the storage capability of multiple raw storage devices.

#### Utilities
- All volume group utility names start with `vg`, e.g. `vgcreate`, `vgs`, `vgrename` etc.

#### Creating volume groups
- Volume groups are created using the `vgcreate` command. The first argument to `vgcreate` is the name you want to five this volume group, and the rest are the list of the physical volumes that are going to back the storage pool.

    ```
    vagrant@ubuntulvmlab0101:~$ sudo vgcreate lvm_tutorial /dev/sdc /dev/sdd1
    Volume group "lvm_tutorial" successfully created
    ```

#### Listing the volume groups
- Listing the volume groups is similar to listing physical volumes, you can use different commands with varying levels of verbosity, `vgdisplay`, `vgscan` and `vgs`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo vgs
      VG           #PV #LV #SN Attr   VSize   VFree 
      lvm_tutorial   2   0   0 wz--n-   7.49g  7.49g
      ubuntu-vg      1   1   0 wz--n- <60.95g 30.47g
    vagrant@ubuntulvmlab0101:~$ sudo vgscan
      Found volume group "lvm_tutorial" using metadata type lvm2
      Found volume group "ubuntu-vg" using metadata type lvm2
    vagrant@ubuntulvmlab0101:~$ sudo vgdisplay
      --- Volume group ---
      VG Name               lvm_tutorial
      System ID             
      Format                lvm2
      Metadata Areas        2
      Metadata Sequence No  1
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                0
      Open LV               0
      Max PV                0
      Cur PV                2
      Act PV                2
      VG Size               7.49 GiB
      PE Size               4.00 MiB
      Total PE              1918
      Alloc PE / Size       0 / 0   
      Free  PE / Size       1918 / 7.49 GiB
      VG UUID               R8fXBL-N9jX-5sqz-P3DF-VmSd-vugZ-5aQB2Y
       
      --- Volume group ---
      VG Name               ubuntu-vg
      System ID             
      Format                lvm2
      Metadata Areas        1
      Metadata Sequence No  2
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                1
      Open LV               1
      Max PV                0
      Cur PV                1
      Act PV                1
      VG Size               <60.95 GiB
      PE Size               4.00 MiB
      Total PE              15602
      Alloc PE / Size       7801 / 30.47 GiB
      Free  PE / Size       7801 / 30.47 GiB
      VG UUID               8OmLUn-Nfx2-0r8d-swbN-NxEc-bvxB-1VNzvc
       
    ```

#### Listing the physical volumes attached to a volume group
- You can list all physical volumes that are connected to a specific volume group by using the following command:
  - `pvdisplay -S vgname=<volume_group_name> -C -o pv_name`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo pvdisplay -S vgname=lvm_tutorial -C -o pv_name
      PV        
      /dev/sdc  
      /dev/sdd1 
    ```
- You can also get the count of physical volumes
  - `pvdisplay -S vgname=<volume_group_name> -C -o pv_count`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo pvdisplay -S vgname=lvm_tutorial -C -o pv_count
      #PV
        2
        2
    ```
    ```
    vagrant@ubuntulvmlab0101:~$ sudo pvdisplay -S vgname=lvm_tutorial -C
      PV         VG           Fmt  Attr PSize  PFree 
      /dev/sdc   lvm_tutorial lvm2 a--  <5.00g <5.00g
      /dev/sdd1  lvm_tutorial lvm2 a--  <2.50g <2.50g
      
    vagrant@ubuntulvmlab0101:~$ sudo pvdisplay -S vgname=lvm_tutorial
     --- Physical volume ---
      PV Name               /dev/sdc
      VG Name               lvm_tutorial
      PV Size               5.00 GiB / not usable 4.00 MiB
      Allocatable           yes 
      PE Size               4.00 MiB
      Total PE              1279
      Free PE               1279
      Allocated PE          0
      PV UUID               nTf4Kg-51X9-jT65-sIzG-W5E9-SpBQ-GLjbpE
       
      --- Physical volume ---
      PV Name               /dev/sdd1
      VG Name               lvm_tutorial
      PV Size               <2.50 GiB / not usable 3.00 MiB
      Allocatable           yes 
      PE Size               4.00 MiB
      Total PE              639
      Free PE               639
      Allocated PE          0
      PV UUID               M6NKR4-yUFJ-w6Hk-8gE3-ivRG-izsa-mgwfcf 
    ```

#### Extending a volume group
- Extending a volume group means adding additional physical volumes to a volume group. To do so the `vgextend` command is used. The syntax is 
  - `vgextend <volume_group> <physical_colume1> <physical_volume2> ...`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo vgextend lvm_tutorial /dev/sdd2
      Physical volume "/dev/sdd2" successfully created.
      Volume group "lvm_tutorial" successfully extended
      
    vagrant@ubuntulvmlab0101:~$ sudo pvdisplay -S vgname=lvm_tutorial -C -o pv_name
      PV        
      /dev/sdc  
      /dev/sdd1 
      /dev/sdd2 
    ```

#### Reducing a volume group
- Just like extending a volume group means adding physical volume, reducing it means removing one or more physical volumes.
- We use the `vgreduce` command to do this,
  - `vgreduce <vgname> <pv1> <pv2> ...`

- Let's remove the physical volumes `/dev/sdc` and `/dev/sdd1`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo vgreduce lvm_tutorial /dev/sdc /dev/sdd1
      Removed "/dev/sdc" from volume group "lvm_tutorial"
      Removed "/dev/sdd1" from volume group "lvm_tutorial"
    vagrant@ubuntulvmlab0101:~$ sudo pvdisplay -S vgname=lvm_tutorial -C -o pv_name
      PV        
      /dev/sdd2 
    ```
- NOTE: If a volume group has any active logical volumes, you won't be able to reduce it like this.

#### Removing a volume group
- You can remove a volume group with the `vgremove` command.
  - `vgremove lvm_tutorial `

---

### Logical Volumes
- A logical volume is like a portion, but instead of sitting on top of a raw disk, it sits on top of a volume group, you can,
  - Format a logical volume with whichever filesystem you want
  - Mount it anywhere in the filesystem you want

#### Utilities
- All logical volume utility names start with `lv`, e.g. `lvcreate`, `lvs`, `lvextend`, `lvreduce`.

#### Creating logical volumes
- Logical volumes are created using the `lvcreate` command. The commonly used syntax looks as follows,
  - `lvcreate -L <size> -n <lvname> <vgname>`
  - The `-L` option is for the size of the new logical volume, you can use any integer with "GB", "MB" or "KB" at the end. E.g. "1GB"
  - The `-n` option is for naming this logical volume
  - Finally, you need to pass the name of the volume group this logical volume is going to be part of.

    ```
    vagrant@ubuntulvmlab0101:~$ sudo lvcreate -L 5GB -n lv1 lvm_tutorial
      Logical volume "lv1" created.
    ```

#### Common operations on a logical volume
- You can out a filesystem on a logical volume as well as mount it anywhere on the filesystem.
- Once created you can find the logical volume in `/dev/<vgname>/<lvname>` path. FOr example in our case the volume is going to be in `/dev/lvm_tutorial/lv1`
    
    ```
    vagrant@ubuntulvmlab0101:~$ ls -l /dev/lvm_tutorial/lv1 
    lrwxrwxrwx 1 root root 7 Dec 27 01:46 /dev/lvm_tutorial/lv1 -> ../dm-1
    ```
- Now you can use it like any partition. Format it with ext4.

    ```
    vagrant@ubuntulvmlab0101:~$ sudo mkfs.ext4 /dev/lvm_tutorial/lv1 
    mke2fs 1.46.5 (30-Dec-2021)
    Creating filesystem with 1310720 4k blocks and 327680 inodes
    Filesystem UUID: e345da43-3da1-466f-b7f9-66b86c8e123a
    Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736
    
    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (16384 blocks): done
    Writing superblocks and filesystem accounting information: done 
    ```
- Mount it at some place in your current directory structure like `/mnt`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo mount -t ext4 /dev/lvm_tutorial/lv1 /mnt
    vagrant@ubuntulvmlab0101:~$ df -h | grep /mnt
    /dev/mapper/lvm_tutorial-lv1       4.9G   24K  4.6G   1% /mnt
    ```

#### Resizing a logical volume
- You can extend a logical volume using `lvextend` command and reduce its size using `lvreduce` command. Or yoy can use the single command `lvresize` to accomplish the both tasks.
- First let's see if the volume group has any space left in it or not.

    ```
    vagrant@ubuntulvmlab0101:~$ sudo vgs -S vgname=lvm_tutorial -o vg_free
      VFree 
      <4.99g
    ```
- The general syntax to use the `lvresize` is
  - `lvresize -L [+|-][size] <vgname>/lvname`
- According to the output, I have some space left, so lets increate the volume size by 2GB.

    ```
    vagrant@ubuntulvmlab0101:~$ sudo lvresize -L +2GB lvm_tutorial/lv1
      Size of logical volume lvm_tutorial/lv1 changed from 5.00 GiB (1280 extents) to 7.00 GiB (1792 extents).
      Logical volume lvm_tutorial/lv1 successfully resized.
    ```
- After the volume size has increased, the filesystem must be resized as well. For ext4, the command to use is `resize2fs`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo resize2fs /dev/lvm_tutorial/lv1 
    resize2fs 1.46.5 (30-Dec-2021)
    Filesystem at /dev/lvm_tutorial/lv1 is mounted on /mnt; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 1
    The filesystem on /dev/lvm_tutorial/lv1 is now 1835008 (4k) blocks long.
    
    vagrant@ubuntulvmlab0101:~$ df -h | grep /mnt
    /dev/mapper/lvm_tutorial-lv1       6.9G   24K  6.5G   1% /mnt
    ```

#### Removing a logical volume
- You can remove a logical volume with the `lvremove` command. The command syntax as follows
  - `lvremove <vgname>/<lvname>`

    ```
    vagrant@ubuntulvmlab0101:~$ sudo umount /mnt
    vagrant@ubuntulvmlab0101:~$ df -h | grep /mnt
    vagrant@ubuntulvmlab0101:~$ 
    vagrant@ubuntulvmlab0101:~$ sudo lvchange -an lvm_tutorial/lv1
    vagrant@ubuntulvmlab0101:~$ 
    vagrant@ubuntulvmlab0101:~$ sudo lvremove lvm_tutorial/lv1
    Do you really want to remove and DISCARD logical volume lvm_tutorial/lv1? [y/n]: y
      Logical volume "lv1" successfully removed
    ```

- Verify the logical volume is deleted successfully or not
    ```
    vagrant@ubuntulvmlab0101:~$ sudo lvs
      LV        VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      ubuntu-lv ubuntu-vg -wi-ao---- 30.47g 
    ```