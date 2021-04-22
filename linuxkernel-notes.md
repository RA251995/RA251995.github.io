# Linux Kernel

### Program
- `vmlinuz-<KERNEL_VERSION>`
- Bootloader (GRUB) loads kernel from disk to RAM
- Has cmd line parameters

### API
- System calls
- Virtual file system entries
    - `proc`
    - `sys`
    - `debugfs`

### Device files

### Gatekeeper
- Enforces privileges (capabilities)
- Executes supervisor instructions
- Implements security policies
- Controls access to hardware and other resources

### Modular
- Small (few MB)
- Sufficient to boot to user space
- Optional functionality added after booting

---
```bash
/boot/vmlinuz-<KERNEL_VERSION>
uname -r # View current loaded kernel
```

---
### Cmds for HW Info
```bash
lshw lspci
lsusb lsblk
lscpu lsdev
```
### Cmds for HW Control and Config
Write to proc, dev, or sys files
```bash
hdparm
inb
outb
setpci
```

---
## System calls (~300)
- Functions implemented by kernel, meant to be called from user space
- `/usr/src/linux-headers-$(uname -r)/include/uapi/asm-generic/unistd.h`
- Documented in man section 2
- Called through standard library (e.g., `libc`)
### System Call Return
- On error, returns -ve value to library
- On error, library sets `errno = abs(return value)` and returns `-1`

---
## Kernel messages
- `printk()` : Sent to RAM buffer and/or system console (important messages)
- `dmesg` : Shows RAM buffer message from kernel
- `dmesg | grep command` : Kernel boot info 
- Log file (eg., `/var/log/messages`) has kernel messages and more
- `journalctl -t kernel` : (systemd) (Stored in disk) 
- `journalctl -t kernel -f` : Follow kernel journal messages
- `journalctl -t kernel | grep command` : Kernel boot info

---
## /proc, /sys, and device files
- Virtual Filesystems (proc, sysfs)
    - Contents not stored on disk
    - Each file and directory has an associated function in kernel that procuces the contents on demand
    - `/proc`
        - Mounted at boot
        - 'process' info and more
        - `ps -l`, `proc -ef`
        - Kernal tunable variables can be written through `/proc` files
        - **Process Info**
            - Each process has a directory with PID as it's name
            - Threads have under `task` directory
    - `/sys`
        - Mounted at boot
        - Kernel object info (HW info)
        - `lspci`
    - Device files (`/dev`)
        - Character or block device
            - Major number = Used by kernel to identify driver
            - Minor number = Used by driver to identify instance
            - Type (`c` or `b` (`ls -l`))
        - Block device : Used to mount file systems
        - `mknod <device_filename> c <major#> <minor_#>`

---
## Booting  
### GRUB Bootloader
- After POST and BIOS
- Installed at special location on disk
- Loads kernel, initial root filesystem, sets up kernel cli and transfers control to kernel
- Can be interrupted for interaction
- `man -k grub` : GRUB utilities
### GRUB 2 Configuration
- `/etc/default/grub`
- `info -f grub -n "simple configuration"`
- Edit `/etc/grub.d/40_custom` -> Run `grub-mkconfig`
- Kernel cli parameters
    - dmesg, /proc/cmdline
    - Kernel source tree -> `Documentation/admin-guide/kernel-parameters.txt`
    - `rdinit` : init process in `initrd` (initial RAM disk)
    - `init` : init process in actual root filesystem in disk
    - `rfkill.default_state`

**Services**
- Older Linux systems -> Run level scripts in `/etc/rc.d` to start up services
- Newer Linux systems -> `systemd` service files in `/etc/systemd/system`

**init RAMFS image**
- `initrd*` or `initramfs*` in `/boot` for each kernel
- `.cpio.gz` or something else
> Use `--no-absolute-filenames` option when using cpio command (`cpio --no-absolute-filenames -idm < <initramfs>.cpio`) / Use `unmkinitramfs`
- man initramfs-tools

---
## Loadable Kernal Modules (LKMs)
- Object file (`*.ko`)
- Run in kernel space
- Dynamically adding functionality to running kernel
- C code compiled for a particular kernel version
> Best practice : Donot remove LKM from runnning kernel
- `/lib/modules/<kernel-version>/kernel/`

**Commands**
```bash
lsmod    # Currently loaded LKMs (First one -> Last loaded)
rmmod    # Remove LKMs
modinfo  # Info from .ko file
depmod   # Generates module config files for modprobe (/lib/modules/<kernel-version>/modules.dep)
insmod   # Insert module
modprobe # Loads / removes module and dependencies
``` 

### **Building LKMs**
```<module-name>.c```
```c
#include <linux/init.h>
#include <linux/module.h>
int init_simple(void) {
    printk("in init module simple\n");
    return 0;
}
void cleanup_simple(void) {
printk("in cleanup module simple\n");
    return;
}
module_init(init_simple);
module_exit(cleanup_simple);
MODULE_LICENSE("GPL");
```
```bash
echo 'obj-m := <module-name>.o' > Makefile
make -C /lib/modules/<kernel-version>/build M=$PWD modules
```

---
## Kernel source
```bash
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-<release codename>.git`
make menuconfig
make help
```

### Searching
```bash
grep -rli <search-query> <search-folder>
```
**cscope**
```bash
make cscope
cscope -d
```
- Use `TAB` to navigate; `^D` to exit

**tags**
```bash
make tags
vi -t <TAG>
```
**vi** tags commands

| Command           | Description          |
|-------------------|----------------------|
| `:tag <TAG/FILE>` |                      |
| `:tn`             | Next definition      |
| `^]`              | Go to definition     |
| `^t`              | Go back              |
| `:ts`             | List all definitions |
| `:tags`           | List tags            |

---
### Kernel driver source
- `drivers/`
- `drivers/char/`

---
### Kernel startup file
- `arch/<arm>/kernel/head.S`

---
### Kernel version
- `Makefile`
    - `KERNELVERSION`
    - `KERNELRELEASE` : `cat /proc/version`
    - `EXTRAVERSION` : User-defined kernel version

---
## Configuring and building kernel
- `make help`

- `make menuconfig`
    - `< >` : `*` - Static, `M` - Module, `<EMPTY>`
    - `[ ]` : `*` - Include, `<EMPTY>`
    - `/`   : Search for config options (Shows entries from Kconfig files)
- `make xconfig`
    - Edit -> Find
    - Option -> Show Name (Config Name)

- `.config` file
    - Distro config file : `/boot/config*`
    - Other platform config files : `arch/arm/configs/<platform>_defconfig`

> `.config` in C language : `autoconf.h` (*Donot edit manually*)

- `make -j<#_JOBS> <bzImage|uImage|vmlinux|...>`
    - `<bzImage|uImage|vmlinux|...>` : Bootloader dependent
        - `bzImage` : GRUB (x86 / PC architecture)
        - `uImage` : U-Boot
        - `vmlinux` : Part of bootable image in ELF (*kernel proper*)

> `System.map` : Kernel symbols; Used for kernel debugging

- `make install`
    - Installs `bzImage` into `/boot/` renaming to `vmlinuz-<KERNEL_VERSION>`
- Update grub config
- Create initramfs/initrd

- `make modules`
- `make modules_install`
    Run `depmod` and copy to `/lib/modules/<KERNEL_VERSION>/`

- `make clean`

### Cross-compilation
- Install cross-compiler `<arm-linux-gnueabi-gcc>`
- `export CROSS_COMPILE=<arm-linux-gnueabi->`
- `make ARCH=<arm> help`
- `make ARCH=<arm> <keystone_defconfig>`
- `make ARCH=<arm> <uImage>`

    