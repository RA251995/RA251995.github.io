## BeagleBone Black Revision C
[System Reference Manual](https://github.com/beagleboard/beaglebone-black/wiki/System-Reference-Manual)
- AM3358BZCZ100 (U5)
- 512MB DDR3 RAM (U12)
- 4GB eMMC Memory (U14)

### Push buttons
- Power Button - S3
- Boot Button - S2
    - SYSBOOT[2]
- Reset Button - S1

### Boot mode

| SYSBOOT[4:0] | Boot Order                                | Remarks                     |
| ------------ | ----------------------------------------- | --------------------------- |
| 11100        | MMC1 (eMMC), MMC0 (SD card), UART0, UART1 | Default                     |
| 11000        | SPI0, MMC0 (SD card), USB0, UART0         | S2 Pressed / P8 43 Grounded |

### UART0 Header - J1

| Pin | BBB Signal |
| --- | ---------- |
| 1   | GND        |
| 4   | RX         |
| 5   | TX         |

### UART0 Settings 
- 115200 8N1
- No Flow Control

### Minicom
```bash
sudo apt-get install minicom
sudo minicom -s
sudo minicom
```

### Web Interface
http://192.168.7.2/ (Windows)
http://192.168.6.2/ (Linux)

### SSH
```bash
ssh debian@<bbb_ip_addr>
```

### USR LEDs
```bash
cat /sys/class/leds/beaglebone:green:user<0-3>/trigger
echo <trigger> > /sys/class/leds/beaglebone:green:user<0-3>/trigger
```

### Internet Sharing

| Device | Network interface |
| ------ | ----------------- |
| Ubuntu | wlp0s20f3         |
| BBB    |enxe415f6f97e47    |

#### Configure firewall rules (Ubuntu)
*Not persistent across reboots*
```bash
sudo iptables --table nat --append POSTROUTING --out-interface wlp0s20f3 -j MASQUERADE
sudo iptables --append FORWARD --in-interface enxe415f6f97e47 -j ACCEPT
```
#### Turn on IP forwarding (Ubuntu)
*Not persistent across reboots*
```bash
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
```

#### (BBB)
*Not persistent across reboots*
```bash
sudo route add default gw 192.168.6.1
sudo sh -c "echo 'nameserver 8.8.8.8' >> /etc/resolv.conf"
```

## Boot Process
### RBL
-  Copies MLO / SPL to internal SRAM.
### MLO / SPL
- MLO is SPL with header which contains load address, image size, etc.
- Copies U-Boot to DDR
### U-Boot
### Linux Kernel

## U-Boot Kermit Download
- Open minicom and issue loadb command at U-Boot console
```bash
sudo minicom
```
```uboot
loadb
```
- Exit minicom without reset (CTRL-A Q)
- Using C-Kermit, send file
```bash
sudo kermit
set port /dev/ttyUSB0
set speed 115200
set carrier-watch off
set flow-control off
send <filename>
```

### U-Boot Image Info
```uboot
imi ${loadaddr}
```

### RootFS
[Custom RFS for Beaglebone Black using Busybox](https://embedjournal.com/custom-rfs-beaglebone-black/)

## SD Card Partitioning

| File System | Label  | Flags |
| ----------- | ------ | ----- |
| fat16       | BOOT   | boot  |
| ext3        | ROOTFS |       |

# Buildroot
#### Install dependencies
```bash
sudo apt-get install sed make binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio unzip rsync file bc wget
wget https://buildroot.org/downloads/buildroot-2022.02.3.tar.gz
```

#### Configure and build
```bash
tar -xvzf buildroot-2022.02.3.tar.gz
cd buildroot-2022.02.3
make beaglebone_defconfig
make menuconfig
make
```

### Configure Linux kernel
```bash
make linux-menuconfig
mkdir custom_linux_config/
cp output/build/linux-custom/.config custom_linux_config
make menuconfig
```
`BR2_LINUX_KERNEL_USE_CUSTOM_CONFIG=y`
`BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE=/home/ubuntu/workspace/buildroot/buildroot-2022.02.3/custom_linux_config/.config`
```bash
make
```

### Copy images and root FS to SD card
```bash
cd output/images
cp MLO /media/ubuntu/BOOT/
cp u-boot.img /media/ubuntu/BOOT/
cp zImage /media/ubuntu/BOOT/
cp am335x-boneblack.dtb /media/ubuntu/BOOT/
sudo tar -C /media/ubuntu/ROOTFS/ -xf rootfs.tar
```

### Custom loadable kernel module build
```bash
cd ~/workspace/buildroot/custom-modules
export PATH=$PATH:/home/ubuntu/workspace/buildroot/buildroot-2022.02.3/output/host/bin
make ARCH=arm CROSS_COMPILE=arm-buildroot-linux-uclibcgnueabihf- -C /home/ubuntu/workspace/buildroot/buildroot-2022.02.3/output/build/linux-custom M=$PWD modules
```

## Ubuntu DHCP Server Setup
### Install
```bash
sudo apt install isc-dhcp-server
```
### Configure
`/etc/dhcp/dhcpd.conf`
```
default-lease-time 3600; 
max-lease-time 7200;
authoritative;

subnet 192.168.3.0 netmask 255.255.255.0 {
    range   192.168.3.100   192.168.3.200;
}

host archmachine {
    hardware    ethernet    <client-mac-address>;
    fixed-address    192.168.3.10;
}
```
`/etc/default/isc-dhcp-server`
```
INTERFACESv4="enp7s0"
```
```bash
ifconfig enp7s0 192.168.3.1
```
### Restart
```bash
sudo systemctl restart isc-dhcp-server
```

## TFTP Boot
### Install TFTP Server
```bash
sudo apt-get install tftpd-hpa
```
Config file:
`/etc/default/tftpd-hpa`

### U-Boot Environment Setup
```uboot
setenv set_bootargs setenv bootargs console=${console} root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait
setenv boot_tftp dhcp\; tftpboot ${fdtaddr} am335x-boneblack.dtb\; run set_bootargs\; bootz ${loadaddr} - ${fdtaddr}
saveenv
```

### TFTP Boot
```uboot
run boot_tftp
```

## NFS

|        | IP Address   | Interface |
| ------ | ------------ | --------- |
| Ubuntu | 192.168.3.1  | enp7s0    |
| BBB    | 192.168.3.10 | eth0      |

### Disable DHCP Server
```bash
sudo systemctl stop isc-dhcp-server
ifconfig enp7s0 192.168.3.1
```
### NFS Server Setup
```bash
sudo apt-get install nfs-kernel-server nfs-common portmap
sudo mkdir /srv/nfs/bbb
sudo tar -C /srv/nfs/bbb -xf rootfs.tar
```
`/etc/exports`
```
/srv/nfs/bbb 192.168.3.10(rw,sync,no_root_squash,no_subtree_check)
```
```bash
sudo exportfs -a
sudo exportfs -rv
sudo systemctl restart nfs-kernel-server
```
### U-Boot Environment Setup
```uboot
setenv set_ip setenv serverip 192.168.3.1\; setenv ipaddr 192.168.3.10
setenv load_tftp tftpboot ${loadaddr} zImage\; tftpboot ${fdtaddr} am335x-boneblack.dtb
setenv set_nfs_bootargs setenv bootargs console=${console} root=/dev/nfs rw rootwait nfsroot=${serverip}:/srv/nfs/bbb/,nolock,vers=3 ip=${ipaddr}
setenv boot_tftp_nfs run set_ip\; run load_tftp\; run set_nfs_bootargs\; bootz ${loadaddr} - ${fdtaddr}
```
### TFTP Boot with NFS
```uboot
run boot_tftp_nfs
```
