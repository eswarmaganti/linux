# Swap Space in Linux

- Swap space in linux is an extension of Physical RAM that provides virtual memory to help maintain system stability and performance. 
- It allows processes to continue running even when RAM is fully used, preventing memory errors
- Swap space is also enables hibernation and safeguards critical processes by temporarily offloading data. However, it should be a complement to RAM, as a system that relies on swap would suffer significant performance degradation.

## What is swap space?
- Swap space (also known as swap memory or paging space) is space on hard drive (SSD or HDD) that represents a substitute of physical (RAM) memory.
- This feature allows an operating system to temporarily move inactive or less frequently used memory pages from RAM to a designated area on the gard drive.
- Swap frees up RAM for more important tasks that require more processing power by transferring data to and from a designated disk space. 
- Data interchange is called swapping, and the designated space is called swap space. THe swapping rate and assertiveness are determined by a parameter called **swappiness**.
- Operating systems like windows and Linux provide a default amount of swap space that users can later adjust according to their requirement.
- Users can also use double swap space, but that means the kernel must kill some processes to free enough RAM for new ones.

## Types of swap space
- There are two types of swap space in linux: *swap files* and *swap partitions*

### Traditional Swap Space
- The classic form of swap space, in use for decades, is traditional swap space. It involves designating and using a dedicated partition on a hard drive. A **Swap Partition** is formatted specifically for this purpose and is separated from the main system partitions.
- Traditional swap space is suitable for scenarios where you have a dedicated server with specific disk partitions and need a fixed amount of swap space.
- Using swap space is common in server environments where a portion of the storage device is allocated for swap and isolated from the rest of the filesystem.
- Traditional swap spaces are also useful in performance-critical systems because thet offer better performance than a swap file. Since swap space is fixed, it also provides predicatable behaviour, which is an advantage when you must ensure specific amount of swap is always available.

### Swap File
- A swap file is a file on the filesystem that the OS uses to swap space. Swap files offer greater flexibility because users can create, resize, or remove them without having to repartition of the disk.
- In linux there are two types of swap files:
  - `Temporary swap file`: It typically uses fragmented disk space and does not reserve any space on the hard drive, making it suitable for limited disk space.
  - `Permanent swap file`: It occupies a contiguous section of the hard drive, requiring more disk space than a temporary sap file. The advantage of using a permanent swap file is that it requires fewer I/O operations, which makes it less resource intensive less a temporary swap file. 
- Swap files are often easier to manage than traditional swap spaces, and they can be placed on different storage devices, which provides greater control over swap space management. Their flexibility makes them great for scenarios with limited disk space.
- Lastly, swap files are particularly advantageous for virtualized environments. Virtual Machines often have dynamic memory requirements, and swap files allow users to easily adjust swap space without altering the underlying disk setup.

## Swap space vs virtual memory
- Swap space and virtual memory are related concepts in OS memory management. However, they server different purposes, and each has distinct characteristics.
- **Virtual Memory**:
  - Virtual memory is a combination of RAM and disk swap space. This combined resource allows running processes to utilize it effectively.
  - This memory type encompasses various memory management strategies and enables processes to use more memory than is physically available.
- **Swap Space**:
  - It's a part of the virtual memory on the hard drive that the system uses when physical memory (RAM) is full. It supplements physical RAM, enabling the system to continue functioning when there is no more free RAM.
  - Swap space is last resort when the physical RAM is fully utilized. The operating system moves less frequently used data from RAM to the swap space to free up memory for current active tasks.

## Benefits of using swap space:
- **Preventing Memory-Related Crashes**: Swap space prevents the system from crashing or become unresponsive when RAM becomes unavailable.
- **Memory Overcommitment**: Swap space enables memory overcommitment, where the system allocates more memory to processes then is physically available. This is useful when processes might not fully use their allocated memory.
- **Large Programs and Multitasking Support**: Swap space enables running multiple large programs simultaneously, even when physical RAM is limited. This is especially helpful for systems with resources-intensive applications or with extensive multitasking.
- **Improving stability**: Swap space allows the OS to continue functioning when there isn't enough RAM for all processes. This prevents abrupt crashes, improves system stability, and allows users to close applications and save their work.
- **Flexibility**: WHen using swap files, users can adjust the swap space size according to the system's needs. This allows users to tune the balance between physical RAM and swap space based on specific workloads and hardware configuration, leading to better overall performance.
- **Virtual Memory Management**: Swap space is a core component os virtual memory management. It allows the OS to provide applications with the illusion of abundant memory, even when physical RAM is limited.
- **Resource Allocation**: Swap space helps optimize resources allocation by moving less frequently used data to slower swap space while more frequently used data remains in RAM. Allocating resources is such a way helps manage memory efficiently.

## Challenges of Using swap space
- **Performance Degradation**: Accessing data from swap space on a hard drive or SSD is significantly slower than accessing data from RAM. Thus, relying on swap space can degrade performance as processes wait longer to fetch data from slower storage.
- **Hard Drive Wear**: Using swap space frequently on a hard drive can accelerate wear and shorten the device's lifespan. Additionally, SSD's have limited write cycles, and excessive swapping can wear them out more quickly.
- **Resource Conflicts**: When the OS uses swap space actively, it competes with other I/O operations on the drive for resources. This can lead to conflicts and cause a slowdown across the system.
- **Reduced Disk Space**: Swap space uses up a portion of the hard drive, leaving less disk space available to the user. This can be especially challenging with permanent swap partitions, which cannot be changes after the disk is partitioned.
- **Complex Memory Management**: While swap space helps manage memory efficiently, configuring and tuning it for optimal system performance introduces complexity.
- **Security Considerations**: Placing sensitive data on swap space could potentially lead to unauthorized access if proper encryption and management measures are not in place.

## How much swap space do you need?
- It was a common belief that swap space should be double the amount of RAM. however, such a setup isn't always applicable in modern systems. The required swap space depends on several factors, sush as RAM  and workload size, hibernation support, hard drive type and speed etc. 
- **RED HAT**: Red Hat recomments a swap space size of 20% of the system's total available RAM. The recommendation applied to modern systems with at least 4GB is RAM.
- **CentOS**:
  - Swap space should be twice the size of RAM, if it is below 2 GB.
  - If RAM is more than 2 GB, then swap space should be RAM + 2GB
- **Ubuntu/Debian**:
  - Ubuntu's recommendations account for hibernation when determining the swap space size. If you use hibernation, the swap space size should equal the amount of RAM plus the square root of the RAM amount.
  - If you don't need hibernation:
    - If there is less than 1GB of RAM, the swap size should be at least the amount of RAM and at most twice that amount
    - If there is more than 1GB of RAM, the swap size should be at least the square root of the RAM amount and at most twice the RAM amount.

## How to check swap space
- There are several different ways to check Linux swap space size and usage. Choose one of the methods in the following sections:

### Use swapon command
- The `swapon` command activates a swap partition in a specified device, but it can also show the details about an existing swap space. Follow the steps below:
    ```
    vagrant@ubuntulvmlab0101:~$ swapon -s
    Filename				Type		Size		Used		Priority
    /swap.img                               file		3873788		0		-2
    ```
- The above command outputs the path to the swap file, the type of swap space (partition or file), its size, and the amount of currently in use. The *Priority* column determines the order in which to use swap devices when swapping data.

### Check */proc/swaps* File
- The */proc/swaps* file measures swap space and its utilization. use the cat command to view the file and see which swap areas are in use on your system:

    ```
    vagrant@ubuntulvmlab0101:~$ cat /proc/swaps 
    Filename				Type		Size		Used		Priority
    /swap.img                               file		3873788		0		-2
    ```
### Use *free* Command
- The free command in linux is a utility that provides information about the system's about the system's memory usage, including virtual memory. Run the following command to see RAM and swap space usage in Linux:
    
    ```
    vagrant@ubuntulvmlab0101:~$ free -m
                   total        used        free      shared  buff/cache   available
    Mem:             834         150         453           0         230         600
    Swap:           3782           0        3782
    ```

### Use top or htop command
- The Linux top command shows the usage of the system's virtual resources in real time. The `htop` command is an alternative to `top` and provides a more user-friendly interface. Use `top` or `htop` to see swap space utilization in Linux.

    ```
    vagrant@ubuntulvmlab0101:~$ top
    
    top - 02:30:17 up 12 min,  1 user,  load average: 0.00, 0.00, 0.00
    Tasks:  86 total,   1 running,  85 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    MiB Mem :    834.1 total,    453.3 free,    150.2 used,    230.7 buff/cache
    MiB Swap:   3783.0 total,   3783.0 free,      0.0 used.    601.6 avail Mem 
    
        PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                       
       1635 vagrant   20   0   18700   7420   5096 S   0.3   0.9   0:00.13 sshd                          
          1 root      20   0  166672  10712   7460 S   0.0   1.3   0:00.44 systemd 
    ```

---

## How to increase/Decrease swap space
- Changing swap space involves different methods for temporary changes (swap space for the current session) and permanent changes (which survive reboots)

1. Check the existing swap space
    ```
    vagrant@ubuntulvmlab0101:~$ swapon --show
    NAME      TYPE SIZE USED PRIO
    /swap.img file 3.7G   0B   -2
    ```
2. Turn off the swap
    ```
    vagrant@ubuntulvmlab0101:~$ sudo swapoff -v /swap.img 
    swapoff /swap.img    
    ```
3. Resize the partition
    ```
    vagrant@ubuntulvmlab0101:~$ sudo fallocate -l 6GB /swap.img 
    vagrant@ubuntulvmlab0101:~$ ls -l /swap.img 
    -rw------- 1 root root 6000000000 Jan 13 02:39 /swap.img
    ```
4. Recreate the swap signature
    ```
    vagrant@ubuntulvmlab0101:~$ sudo mkswap /swap.img 
    mkswap: /swap.img: warning: wiping old swap signature.
    Setting up swapspace version 1, size = 5.6 GiB (5999992832 bytes)
    no label, UUID=2437e7a2-b99f-4299-ba73-33e26de571f1
    ```
5. Enable the swap space
    ```
    vagrant@ubuntulvmlab0101:~$ sudo swapon /swap.img
    vagrant@ubuntulvmlab0101:~$ swapon --show
    NAME      TYPE SIZE USED PRIO
    /swap.img file 5.6G   0B   -2
    ```












