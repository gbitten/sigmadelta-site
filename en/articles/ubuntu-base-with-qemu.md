---
area: articles
ref: ubuntu-base-with-qemu
title: Ubuntu Base with QEMU
lang: en
comments: true
---

This procedure shows how to create a QEMU<sup>[1]</sup> virtual machine with Ubuntu Base<sup>[2]</sup> as the root filesystem. There is a great post about Ubuntu Base installation in Ask Ubuntu<sup>[3]</sup>. I am following the same path, but here I am installing on a QEMU virtual machine. It worth mentioning that this procedure will work only if the host and the virtual machine have the same architecture. Besides that, the procedure was tested on x86. I don't know if it will work on other architectures. 

## About Ubuntu Base

**Ubuntu Base** is a stripped rootfs, which can install any package from Ubuntu repositories using the `apt-get` command. Ubuntu Base was previously known as Ubuntu Core, but Canonical renamed it and gave that name to other project. 

## About QEMU

**QEMU** is an open source virtual machine monitor (VMM) or hypervisor. It can virtualize hardwares from many architectures, including x86, SPARC, ARM, PowerPC, MIPs, Microblaze and others.

## Download Ubuntu Base

Choose the Ubuntu Base version [here](http://cdimage.ubuntu.com/ubuntu-base/releases/) and download it. In my example, I chose the version 16.04. If the host is 32 bits then the virtual machine should be also 32 bits. 

If the host is 32 bits then download Ubuntu Base 32 bits:

```
wget http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/ubuntu-base-16.04-core-i386.tar.gz
```

If the host is 64 bits then download Ubuntu Base 64 bits:

```
wget http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/ubuntu-base-16.04-core-amd64.tar.gz
```

## Install QEMU

```
sudo apt-get install qemu
```

## Create QEMU image

```
qemu-img create -f raw ubuntu-base.img 500M
```

## Log as root

All of the following commands require root right.

```
sudo su
```

## Create a Network Block Device

QEMU can export a disk image as a Network Block Device (NBD). With that, the host can handle the QEMU image as any other block device.  

```
modprobe nbd max_part=16
qemu-nbd -f raw -c /dev/nbd0 ubuntu-base.img
```

## Partition the device

```
fdisk /dev/nbd0
```

Then press `n` `p` `1` `<Return>` `<Return>` `a` `1` `w`. This sequence creates a single bootable partition on the device.

## Format the partition

```
mkfs.ext4 /dev/nbd0p1
```

## Mount the partition

```
mount /dev/nbd0p1 /mnt
```

## Install Ubuntu Base

If the host is 32 bits:

```
tar xvfz ubuntu-base-16.04-core-i386.tar.gz -C /mnt
```

If the host is 64 bits:

```
tar xvfz ubuntu-base-16.04-core-amd64.tar.gz -C /mnt
```

## Copy DNS configuration

```
cp /etc/resolv.conf /mnt/etc/resolv.conf
```

## Run `chroot`

```
mount --rbind /sys /mnt/sys
mount --rbind /proc /mnt/proc
mount --rbind /dev /mnt/dev
chroot /mnt
```

### Install the kernel

```
apt-get update 
apt-get install linux-image-4.4.0-75-generic
```

**ATTENTION**: at the end of the kernel installation, will be asked: `GRUB install devices`. Choose the option related to nbd0p1 device. Any other option could damage the host bootloader. 

### Adjust grub configuration

```
sed -i "s/quiet splash/rw text/g" /etc/default/grub
update-grub
sed -i "s/nbd0p1 ro/sda1/g" /boot/grub/grub.cfg
```

###  Change the root password

```
passwd root
```

### Exit ```chroot```

```
exit
```

## Install grub
```
grub-install --root-directory=/mnt /dev/nbd0
```

## Reboot the host

```
reboot
```

## Run the virtual machine

If the virtual machine is 32 bits:

```
qemu-system-i386 ubuntu-base.img
```

If the virtual machine is 64 bits:

```
qemu-system-x86_64 ubuntu-base.img
```

## References

* [1] [QEMU](http://www.qemu.org/)
* [2] [Ubuntu Base](https://wiki.ubuntu.com/Base)
* [3] [What commands are needed to install Ubuntu Core?](https://askubuntu.com/a/70139/413551)

