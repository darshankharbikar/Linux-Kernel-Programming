# LKM Internals

## Overview of compiling Kernel Modules
----------------------------------------
- kernel use  kbuild system to build the kernel modules
- kbuild system reads  the assignment of "obj-m := modulename.o"  from the makefile. 
- Now the kbuild system know that it has to build "modulename.ko" and will look for "modulename.c" for the source.
- In case these files are not present in the directory passed to "M" , the compiling will stop with an error. 
- If the files are present the source file is compiled to a "modulname.o",  and "modulename.mod.c" is created which is compiled to "modulename.mod.o". 
- The modulename.mod.c is a file that basically contains the information about the module (Version information etc).
- The modulename.o and the modulename.mod.o are linked together by modpost in the next stage to create the "modulename.ko" 

## Other Files
--------------
- "module.symvers": This will contain any of external symbols that is defined in your module and hence not present in the module.symvers of the kernel .
- "modules.order" : In case you are compiling multiple modules together, it will list out the order in which the compilation and creation of .ko takes 

- lsmod – List Modules that Loaded Already
- insmod – Insert Module into Kernel
- modinfo – Display Module Info
- rmmod – Remove Module from Kernel
- modprobe – Add or Remove modules from the kernel

 
## insmod vs modprobe
--------------------- 
- insmod:		Loads the module given 'insmod /path/to/module.ko'
- modprobe:	Loads the module only in /lib/modules/$(uname -r) 'modprobe /home/test/hello.ko' will not work
- insmod:		Dependencies if present are not loaded
- modprobe:	modprobe calculates dependencies, loads the dependencies and then the main module

 
## How modprobe calculates dependencies?
- Modprobe depends on depmod tool to calculate dependencies.
- depmod calculates dependencies of all the  modules present in /lib/modules/$(uname -r) folder, and places the dependency information in /lib/modules/$(uname -r)/modules.dep file
- E.g. kernel/drivers/net/wireless/admtek/adm8211.ko: kernel/net/mac80211/mac80211.ko kernel/net/wireless/cfg80211.ko      kernel/drivers/misc/eeprom/eeprom_93cx6.ko
- When you say modprobe adm8211.ko, eeprom_93cx6.ko, cfg80211.ko is loaded first and then adm8211.ko
- Modules are loaded right  to left and removed left to right
- So while removing adm8211.ko is removed, then cfg80211.ko and finally eeprom_93cx6.ko
- We can re-load the modules.dep file by running "depmod -a" command

- We know on insmod, the function passed in the module_init macro is called, and on rmmod, the argument passed in the module_exit is called.

Let's see the definition of this macro, it is present in linux/module.h
```
/* Each module must use one module_init(). */
#define module_init(initfn)                 \
    static inline initcall_t __inittest(void)       \
    { return initfn; }                  \
    int init_module(void) __attribute__((alias(#initfn)));

/* This is only required if you want to be unloadable. */
#define module_exit(exitfn)                 \
    static inline exitcall_t __exittest(void)       \
    { return exitfn; }                  \
    void cleanup_module(void) __attribute__((alias(#exitfn)));
```
- The purpose of defining __inittest function is to check during compile time, the function passed to module_init() macro is compatible with the initcall_t type.
```
initcall_t is defined in linux/init.h:
typedef int (*initcall_t)(void);
```
- If you declare module_init function which returns void instead of int, the compiler will throw warning:
- The last line uses the alias attribute of gcc to assign another name to init_module, so that you can have a better name as per your driver (e.g. cdrom_init instead of init_module), instead of each driver having init_module. 
- Same is the case with module_exit, giving whatever name in module_exit as parameter to cleanup_module.
