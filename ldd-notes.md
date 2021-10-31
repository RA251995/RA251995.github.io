# Linux Device Drivers

## Loadable Kernel Module (LKM)

`hello-world.c`

```c
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");

static int __init hello_init(void)
{
    printk(KERN_ALERT "Hello, World!\n");
    return 0;
}

static void __exit hello_exit(void)
{
    printk(KERN_ALERT "Goodbye, World!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

### LKM Build
`Makefile` for source file `hello-world.c`
```makefile
obj-m := hello-world.o
```
`Makefile file` for source files `file1.c`, `file2.c`
```makefile
obj-m := module.o
module-objs := file1.o file2.o
```
```bash
make -C <KERNEL_DIR> M=$PWD modules
```

`<KERNEL_DIR>` can be either of the following:
- Kernel source directory : `/lib/modules/$(shell uname -r)/build`
- Kernel header directory : `/usr/src/linux-headers-$(uname -r)`

### LKM Loading & Unloading

- Load using `insmod <module-name>.ko` / `modprobe` 
- Unload using `rmmod <module-name>.ko`
- List using `lsmod`

### Module Parameters

```c
module_param(<var>, <type>, <perm>);
module_param_array(<var>, <type>, <num>, <perm>);
```

`<type>`: `bool`, `invbool`, `charp`, `int`, `long`, `short`, `uint`, `ulong`, `ushort`

`<perm>`: `S_IRUGO` (See `./include/linux/stat.h`)

```bash
insmod <module-name> <var-1>=<val-1> <var-2> <val-2>
```

## Char Drivers
### Major Number & Minor Number
- Major Number : Identifies driver associated with device
- Minor number : Identifies device

#### Internal Representation
`<linux/kdev_t.h>`
Convert `dev_t` to major number and minor number

```c
MAJOR(dev_t dev);
MINOR(dev_t dev);
```

Convert major number and minor number to `dev_t`
```c
MKDEV(int major, int minor);
```

#### Allocation & Freeing Device Numbers
https://www.kernel.org/doc/html/v5.10/core-api/kernel-api.html#char-devices

`<linux/fs.h>`

```c
int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);
```

```c
void unregister_chrdev_region(dev_t first, unsigned int count);
```

- `name` appears in `/proc/devices` and sysfs.

### Char Device Registration
https://www.kernel.org/doc/html/v5.10/core-api/kernel-api.html#char-devices

`<linux/cdev.h>`

```c
void cdev_init(struct cdev *cdev, struct file_operations *fops);
int cdev_add(struct cdev *dev, dev_t num, unsigned int count);
void cdev_del(struct cdev *dev);
```

