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
`<module-name>.c`
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
        - `vmlinux` : (Conflict of filename with *kernel proper*)

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

---
## Composite Kernel Image (ARM)
1. `vmlinux` : *Kernel proper*, in ELF format
> `System.map` : Kernel symbol table for `vmlinux`; Used for kernel debugging
2. `Image` : Stripped *kernel proper* in binary form
3. `piggy_data` : Compressed `Image` file
4. `piggy.S` : Contain `piggy_data` as payload to be used within *Bootstrap loader*
5. **Bootstrap loader** : *Glue* between bootloader and kernel; Consists of:
    - `head.o`, `head-xscale.o` : Processor initialization (Conflict of filename with kernel `head.o` in `arch/arm/kernel/`)
    - `misc.o` : Decompression and relocation
    - `big-endian.o`, etc. : Processor-specific initialization

---
## Boot Control Flow (ARM)
> PowerOn -> Bootloader (U-Boot) -> Bootstrap Loader start (`arch/arm/boot/compressed/head.o`) -> Kernel vmlinux start (`arch/arm/kernel/head.o`) -> Kernel (`main.o`)

### Kernel Startup: `main.c`
- `init\main.c`
- Called from `head-common.S` included in `head.S` (`start_kernel`)
- Calls `setup_arch` in `arch/arm/kernel/setup.c` for architecture setup
- Spawns `init` thread

### Kernel command-line parameters
- `dmesg`, `/proc/cmdline`
- Kernel source tree -> `Documentation/admin-guide/kernel-parameters.txt`
- Examples:
    - `rdinit` : init process in initial RAM disk
    - `init` : init process in actual root filesystem in disk
    - `initcall_debug` : Can be used for boot time analysis
    - `rfkill.default_state`
- `__setup` : Macro used for mapping kernel command-line paramaters to respective functions using `.init.setup` ELF section. 

---
## User Space Initialization

### Root File System (`/`)
- Required to realize the benefits of its services
- **FHS** : File System Hierarchy Standard
- Top level directories

| Directory | Contents                                                   |
|-----------|------------------------------------------------------------|
| `bin`     | Binary executables, for all users                          |
| `dev`     | Device nodes                                               |
| `etc`     | Local system configuration files                           |
| `home`    | User account files                                         |
| `lib`     | System libraries (standard C library, ...)                 |
| `sbin`    | Binary executables, for superusers                         |
| `tmp`     | Temporary files                                            |
| `usr`     | Secondary FHS for application programs, usually read-only  |
| `var`     | Variable files (System logs, Temporary configuration files)|

#### Minimal file system example
```bash
.
|-- bin
|   |-- busybox
|   ‘-- sh -> busybox
|-- dev
|   ‘-- console
|-- etc
|   ‘-- init.d
|       ‘-- rcS
‘-- lib
    |-- ld-2.3.2.so
    |-- ld-linux.so.2 -> ld-2.3.2.so
    |-- libc-2.3.2.so
    ‘-- libc.so.6 -> libc-2.3.2.so
5 directories, 8 file
```

- [Library Optimizer Tool](http://libraryopt.sourceforge.net/) : Reduce size of shared libraries
- `bitbake`, `buildroot` : Automated file system build tools

---
### `init`
> `*-ldd init` :  Print `init` exxecutable dependencies
#### **System V Init**
- Implemented using `init` program and set of startup scripts
- Older Linux systems -> Run level scripts in `/etc/rc.d` to start up services
- Newer Linux systems -> `systemd` service files in `/etc/systemd/system`

Common runlevel puposes:

| Runlevel | Purpose                                 |
|----------|-----------------------------------------|
| 0        | System shutdown                         |
| 1        | Maintenance                             |
| 2        | User-defined                            |
| 3        | General-purpose multiuser configuration |
| 4        | User-defined                            |
| 5        | Multiuser with GUI on startup           |
| 6        | System restart                          |

---
### Initial RAM Disk
- Early root file system (before mounting real root file system and spawning `init`)
- `initrd*` (Legacy) or `initramfs*` in `/boot` for each kernel
- Enabled using kernel .config file
- Usually used to load device driver to access real root file system
- **`initrd`**
    - Gzipped file system image
    - Memory location and size passed as kernel command-line parameter (`initrd=<address>,<size>`)
    - Kernel copies ramdisk from physical memory into kernel ramdisk, `/dev/ram` and frees physical memory
    - If `root` is not defined in kernel command-line parameters,
        - Runs `linuxrc` in ramdisk
        - Umounts `initrd` and mounts at `/initrd` (if `/initrd` exists in real root device)
- **`initramfs`**
    - `cpio` archive
    - Integrated into kernel source tree (`usr/`)
    - Default contents specified in `usr/default_cpio_list`
    - `Documentation/filesystems/ramfs-rootfs-initramfs.rst`
    - Minimal contents 
        - `init` symlink required at root
        - Can be overridden using `rdinit` in kernel command-line
    ```bash
        .
        |-- bin
        |
        |-- busybox
        |
        ‘-- sh -> busybox
        |-- dev
        |
        ‘-- console
        |-- etc
        |
        ‘-- init.d
        |
        ‘-- rcS
        ‘-- init -> /bin/sh
    ```
> Use `--no-absolute-filenames` option when using cpio command (`cpio --no-absolute-filenames -idm < <initramfs>.cpio`) / Use `unmkinitramfs`
- man initramfs-tools

---
## GRUB Bootloader
- After POST and BIOS
- Installed at special location on disk
- Loads kernel, initial root filesystem, sets up kernel cli and transfers control to kernel
- Can be interrupted for interaction
- `man -k grub` : GRUB utilities
### GRUB 2 Configuration
- `/etc/default/grub`
- `info -f grub -n "simple configuration"`
- Edit `/etc/grub.d/40_custom` -> Run `grub-mkconfig`

---
## U-Boot Bootloader
- `git clone git://git.denx.de/u-boot.git`
### U-Boot Commands
- `tftp <loadaddr> <uImage|fdt>`
- `iminfo` : View `uImage` header info
- `diskboot <loadaddr> <IDE device #>:<partition #>`
- `bootm <loadaddr> - <fdtaddr>`
### U-Boot Configuration
- `make <platform>_defconfig` : Configure architecture, core and platfoem
- `include/configs/<platform>.h` : Edit memory/flash size (See README)
### U-Boot Porting
- `board/<vendor>/<boardname>`
- `include/configs/<platform>.h`
- `Makefile`
> U-Boot Entry point: `arch/<arch>/cpu/<core>/start.S`

---
## Device Tree Blob (Flat Device Tree)
- To pass low-level hardware information from bootloader to kernel
- `arch/<arch>/boot/dts`
- DTS <--dtc--> DTB
    - `dtc -O dtb -o <board>.dtb -b 0 <board>.dts`
    - `dtc -I dtb -O dts <board>.dtb ><board>.dts`
- DTS 
    - **cell** :  32-bit quantitity
    - `address-cells`, `size-cells` : Number of cells (32-bit fields) required to specify an address or size in the child node
    - [Embedded Power Architecture Platform Requirements](https://elinux.org/images/c/cf/Power_ePAPR_APPROVED_v1.1.pdf)
