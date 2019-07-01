---
area: articles
ref: compiling-preempt-rt
title: Compile Preempt RT
lang: en
comments: true
---

I usually run this procedure (with some variations if necessary) to compile and install the Linux kernel with Preempt RT<sup>[1]</sup> on my Ubuntu box.

## Prepare the build environment

Install the toolchain to build the kernel. 

```
sudo apt-get install git
sudo apt-get install libssl-dev
sudo apt-get install dpkg-devi
sudo apt-get build-dep linux-image-$(uname -r)
```

## Create the directory for the kernel source

```
mkdir preempt-rt
cd preempt-rt
```

## Get the Linux kernel and the Preempt RT patch set

The bellow methods do almost the same thing. The main difference is the first method applies the Preempt RT patch set at once, as only one commit. In the second, the Preempt RT patch set is a sequence of smaller commits, so it is easier to understand what each of them does.

### Method 1

Download the mainline kernel.
 
```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
git fetch origin v4.9.18
git reset --hard FETCH_HEAD
```

Apply the Preempt RT patch set.

```
wget -O - https://www.kernel.org/pub/linux/kernel/projects/rt/4.9/older/patch-4.9.18-rt14.patch.gz | gunzip -c > /tmp/patch-4.9.18-rt14.patch
patch -p1 < /tmp/patch-4.9.18-rt14.patch
git add -A .
git commit -m "Apply Preempt RT patch set v4.9.18-rt14"
```

### Method 2

Get Preempt RT from its official git repositories ([stable](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git) or [development](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git)).

```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git
git fetch origin v4.9.18-rt14-rebase
git reset --hard FETCH_HEAD
```

## Configure the kernel build

The kernel build configuration is defined in the ```.config``` file.

### Initialize the kernel build configuration

There are many ways to initialize the ```.config``` file. Here, I am using the Ubuntu configuration to start the configuration of the kernel.

```
cp /boot/config-$(uname -r) .config
make olddefconfig
```

### Set build options for real-time systems

Use your preferred kernel config method (ex: ```make menuconfig```) to set the following options:

* PREEMPT_RT_FULL = y
* HIGH_RES_TIMERS = y

## Compile and install the kernel

These are the main methods I use to compile and install the kernel. 

### Method 1

The traditional method installs the kernel locally.

```
make -j $(nproc) LOCALVERSION=-rt14
sudo make modules_install install
```

### Method 2

This method creates packages which could be installed on other machines. The kernel's ```make``` has options for different kind of packages<sup>[2]</sup>. Here, I use the option ```deb-pkg``` to generate Debian packages.

```
make -j $(nproc) deb-pkg LOCALVERSION=-rt14
```

The above command creates 5 packages:

* linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14-dbg_4.9.18-rt14-rt14-5_amd64.deb
* linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-libc-dev_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb

Install the kernel packages.

```
sudo dpkg -i linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
```

## Reboot and verify

At the end of the installation, the grub configuration will be updated to include an option with the new kernel. Reboot with the new kernel and verify it with the command ```uname -a```.

## References

* [1] [Preempt RT patch](https://rt.wiki.kernel.org/index.php/CONFIG_PREEMPT_RT_Patch)
* [2] [Output of kernel's "make help"](https://www.kernel.org/doc/makehelp.txt)
