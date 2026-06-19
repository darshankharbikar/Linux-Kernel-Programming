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

## Hello World Kernel Module
----------------------------
- In C/C++ Programming we have the main() as the entry point and exit point.
- Kernel modules must have at least two functions:
  -	a "start" (initialization) function : which is called when the module is loaded into the kernel
  -	an "end" (cleanup) function called : which is called just before it is removed
- This is done with the module_init() and module_exit() macros

## Licensing
---------
- Module should specify which license you are using MODULE_LICENSE() macro

	"GPL"				[GNU Public License v2 or later]
	"GPL v2"			[GNU Public License v2]
	"GPL and additional rights"	[GNU Public License v2 rights and more]
	"Dual BSD/GPL"			[GNU Public License v2
					 or BSD license choice]
	"Dual MIT/GPL"			[GNU Public License v2
					 or MIT license choice]
	"Dual MPL/GPL"			[GNU Public License v2
					 or Mozilla license choice]

	"Proprietary"			[Non free products]


## Header Files
------------
- Every kernel module needs to include linux/module.h. for macro expansion of module_init and module_exit
	linux/kernel.h only for the macro expansion for the printk() log level

- To Build Modules:
```
	make -C /lib/modules/`uname -r`/build M=${PWD} modules
```
- To clean:
```
	make -C /lib/modules/`uname -r`/build M=${PWD} clean
```

- The above commands starts by changing its directory to the one provided with the -C option (that is your kernel source directory)
- There it finds the kernel's top level makefile. The M= option causes the Makefile to move back into your module source directory before trying to build the modules.

Note: M is not make option but argument passed to it

- obj-m refers to the list of modules 
- The kernel Makefile will read our Makefile to find out what to build, we specify that by writing obj-m += hello.o

## printf vs  printk
------------------
- printf() is a function in the C Standard Library
- printk() is a kernel level function
- The printk() is called with one more argument than printf(), like this:
- printk(KERN_log_priority "hello world\n");
- Here, log_priority is one of the eight values (predefined in linux/kernel.h, similar to /usr/include/sys/syslog.h)
```
	EMERG, 
	ALERT, 
	CRIT, 
	ERR, 
	WARNING, 
	NOTICE, 
	INFO, 
	DEBUG (in order of decreasing priority).
```
- printk() writes to the kernel buffer, whereas printf() writes on the standard output

 
## What happens when we do insmod on a module 
============================================
- What is a kernel module?
	- Kernel module is a piece of kernel code which can be added to the running kernel when loaded and can be removed from the kernel when the functionality is removed.
	- When we do insmod on a module, it performs a series of steps:

	a) It calls init_module() to intimate the kernel that a module is attempted to be loaded and transfers the control to the kernel

	b) In kernel, sys_init_module() is run. It does a sequence of operations as follows:

		--> Verifies if the user who attempts to load the module has the permission to do so or not

		--> After verification, load_module function is called.

		-->	The load_module function assigns temporary memory and copies the elf module from user space to kernel memory using copy_from_user

		--> It then checks the sanity of the ELF file ( Verification if it is a proper ELF file )
		--> Then based on the ELF file interpretation, it generates offset in the temporary memory space allocated. This is called the convenience variables
		-->	User arguments to the module are also copied to the kernel memory
		-->	Symbol resolution is done
		-->	The load_module function returns a reference to the kernel module.

		-->	The reference to the module returned by load_module is added to a doubly linked list that has a list of all the modules loaded in the system

		-->	Then the module_init function in the module code is called

## Linux dmesg command Tutorial 
 
### What does dmesg command do?
- Kernel keeps all the logs in a ring buffer.
- This is done to avoid the boot logs being getting lost until the syslog daemon starts and collects them and stores them in /var/log/dmesg.
- We will loss the boot up logs if we don't store them in ring buffer.
- dmesg command is used to control or print kernel ring buffer. Default is to prints messages from the kernel ring buffer on to console.

 
### Important dmesg commands:


1. Clear Ring buffer: 
```	
	$dmesg -c -> Will clear the ring buffer after printing
	$dmesg -C -> Will clear the ring buffer but does not prints on the console.
```
2. Don't Print Timestamps: 
```
	$dmesg -t -> Will not print timestamps
```
3. Restrict dmesg command to list of levels.
```
	$ dmesg -l err,warn will print only error and warn messages
```
4. Print human readable timestamps:
```
	$dmesg -T will print timestamps in readable format. Note: Timestamp could be inaccurate.
```
5. Display the log level in the output:
```
	$dmesg -x will add loglevel to the output.
```
6. You can combine options, so dmesg -Tx will print both human readable time and loglevel.




