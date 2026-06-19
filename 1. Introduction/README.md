# INTRODUCTION TO LINUX KERNEL PROGRAMMING
## What is a device driver
--------------------------
- A device driver (often referred to as driver’) is a piece of software that controls a particular type of device which is connected to the computer system
- A device driver has three sides:
  - one side talks to the rest of the kernel
	- one talks to the hardware, 
	- and one talks to the user

# Linux Device Driver Architecture

```text
+------------------------+
|      User Space        |
|  (Applications)        |
+-----------+------------+
            |
            | System Calls
            | (open, read, write,
            |  ioctl, mmap, poll)
            v
+------------------------+
|      Kernel Space      |
|                        |
|   Device Driver        |
| (Character/Block/Net)  |
+-----------+------------+
            |
            | Hardware Access
            | (Registers, DMA,
            |  Interrupts, Buses)
            v
+------------------------+
|       Hardware         |
|                        |
| UART, SPI, I2C, GPIO   |
| USB, Ethernet, Display |
| Sensors, Storage, etc. |
+------------------------+
```

---

## Device Driver Communication Path

```text
+------------------------+
|      User Space        |
|                        |
|  Application Program   |
+-----------+------------+
            |
            | open("/dev/mydevice")
            | read()
            | write()
            | ioctl()
            v
+------------------------+
|      Device File       |
|      /dev/mydevice     |
+-----------+------------+
            |
            v
+------------------------+
|     Device Driver      |
|                        |
| file_operations        |
|  - open()              |
|  - release()           |
|  - read()              |
|  - write()             |
|  - ioctl()             |
+-----------+------------+
            |
            v
+------------------------+
|       Hardware         |
|                        |
| Registers              |
| DMA Engine             |
| Interrupt Controller   |
| Peripheral Bus         |
+------------------------+
```

---

## Complete Driver Stack

```text
+--------------------------------------------------+
|                 User Application                 |
+-------------------------+------------------------+
                          |
                          | System Calls
                          v
+--------------------------------------------------+
|                  Virtual File System (VFS)       |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|                 Device Driver                    |
|--------------------------------------------------|
| Character Driver                                 |
| Block Driver                                     |
| Network Driver                                   |
+-------------------------+------------------------+
                          |
                          | Bus Framework
                          v
+--------------------------------------------------+
|            I2C / SPI / USB / PCI Driver          |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|                  Hardware Device                 |
+--------------------------------------------------+
```

---

## Example: UART Driver Flow

```text
User Application
       |
       | write(fd, "Hello", 5)
       v
Device File
(/dev/ttyS0)
       |
       v
UART Driver
       |
       | Programs UART TX Register
       v
UART Hardware
       |
       | Serial Data Transmission
       v
External Device
```

---

## Key Point

A **device driver** is a kernel-space software component that acts as a translator between:

```text
User Application
       ↕
Device Driver
       ↕
Hardware Device
```

Without a device driver, user-space applications cannot safely and directly access hardware.

## What is a Kernel Module
--------------------------
- Traditional way of adding code to the kernel was to recompile the kernel and reboot the system
- Kernel Modules are piece of code that can be loaded/inserted and unloaded/removed from the kernel as per the demand/need.


## Other Names
---------------
1. Loadable Kernel Modules (LKM)
2. Modules
 
- Extension: .ko (Kernel Object)

## Standard Location for Kernel Modules
---------------------------------------
-  Modules are installed in the /lib/modules/<kernel version> directory of the rootfs by default. 

## Device Driver vs Kernel Modules
----------------------------------
- A kernel module may not be a device driver at all.
- A driver is like a sub-class of module.
- Modules are used for the below:

1. Device Drivers.
2. File System 
3. System Calls
4. Network drivers: Drivers implementing a network protocol (TCP/IP)
5. TTY line disciplines: For terminal devices

## Advantages of Kernel Modules
-------------------------------

1. All parts of the base kernel stay loaded all the time. Modules can save you memory, because you have to have them loaded only when you're actually using them

2. Users would need to rebuild and reboot the kernel every time they would require a new functionality.

3. A bug in driver which is compiled as a part of kernel will stop system from loading, whereas module allows systems to load. 

4. Faster to maintain and debug

5. Makes it easier to maintain multiple machines on a single kernel base.

## Disadvantages of Kernel Modules
----------------------------------

1. Size:  Module management consumes unpageable kernel memory. 
	a basic kernel with a number of modules loaded will consume more memory than an equivalent kernel with the drivers compiled into the kernel image itself	  
	This can be a very significant issue on machines with limited physical memory

2. As the kernel modules are loaded very late in the boot process, hence core functionality has to go in the base kernel (E.g. Memory Management)

3. Security: If you build your kernel statically and disable Linux's dynamic module loading feature, you prevent run-time modification of the kernel code. 

## Configuration
------------------
- In order to support modules, the kernel must have been built with the following option enabled:

CONFIG_MODULES=y


## Types of Modules
-------------------

1. **In-Source Tree**: Modules present in the Linux Kernel Source Code

2. **Out-of-Tree**: Modules not present in the Linux Kernel Source Code.

- All modules start out as "out-of-tree" developments, that can be compiled using the context of a source-tree. 
- Once a module gets accepted to be included, it becomes an in-tree module.

## Basic Commands
-----------------

1. **List Modules**: (lsmod) lsmod gets its information by reading the file /proc/modules.

2. **Module Information**: (modinfo) : prints the information of the module.
