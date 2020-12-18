---
title: 'Installing FreeBSD 9-RELEASE onto a ZFS root'
date: '2013-07-20T20:20:10-04:00'
description: >-
  This tutorial will guide you through installing FreeBSD 9-RELEASE onto a ZFS root with UFS boot. We will also cover a small amount of post-install items to ensure your system is available for immediate use.
---
**Disclaimer**: This should work for you, but all systems are different. I have compiled this information from many sources and will include them at the bottom of this post. The reason I am creating this is because I feel as if the other sources do not “finish” the installation, so a new user would be oblivious as to what they should do next.

### Step 1: Download FreeBSD 9-RELEASE

Go to this link, [Get FreeBSD](http://www.freebsd.org/where.html), and click the ISO link on the architecture that you would like to install FreeBSD on. If you are certain you have an AMD64 (an Intel or AMD 64bit capable CPU), or a SPARC, or Itanium, or any of the others, make sure to grab that ISO architecture. When in doubt, download the [i386 ISO](ftp://ftp.freebsd.org/pub/FreeBSD/releases/i386/i386/ISO-IMAGES/9.0/). If you are burning it to a memory stick, USB drive, etc., use the memstick image. Else, download the dvd.iso image. To create a bootable USB drive with the memstick image, please refer to this article: [2.3.7 Prepare the Boot Media](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/install-pre.html).

_Note: Make sure all of your hardware is compatible with FreeBSD before attempting to install it. Please read the [Hardware Notes](http://www.freebsd.org/releases/9.0R/hardware.html) before continuing before continuing._


### Step 2: Boot FreeBSD

Insert the DVD into your DVD drive or plug in the USB drive. If your BIOS is not set to automatically boot from CD/DVD or USB, make sure to manually select the boot device. When FreeBSD boots to its menu, choose “Live CD”.


### Step 3a: Create the BOOT and ZFS partitions

If you are doing a single disk installation, create these partitions:
```bash:title=bash
gpart create -s gpt ada0 $ gpart add -b 34 -s 94 -t freebsd-boot ada0
gpart add -t freebsd-zfs -l disk0 ada0
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada0
```

If you are doing a ZFS Mirror, repeat the procedure onto the second drive:
```bash:title=bash
gpart create -s gpt ada1
gpart add -b 34 -s 94 -t freebsd-boot ada1
gpart add -t freebsd-zfs -l disk1 ada1
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada1
```

_Note: On your system it may not be ```ada[n]```. If that is the case, substitute the correct hard drive identifiers above. To check, ```cd``` to ```/dev``` and run ```ls``` and see what your disks are listed as._


### Step 3b: Creating the ZFS pool

If you have a single disk installation, use this:

```bash:title=bash
zpool create zroot /dev/gpt/disk0
```

However, if you are creating a ZFS Mirror, use this instead:

```bash:title=bash
zpool create zroot mirror /dev/gpt/disk0 /dev/gpt/disk1
```

This will create a zpool named “zroot”. Even though we now have a pool, it is useless without any filesystems or mountpoints.


### Step 4: BOOTFS property, ZFS checksum, and mounting

This is just a simple task. It makes sure your system is bootable, while setting the ZFS checksum (this helps detect bitrot), and then mounts zroot.

```bash:title=bash
zpool set bootfs=zroot zroot
zfs set checksum=fletcher4 zroot
zfs set mountpoint=/mnt zroot
```


### Step 5: Export/Import pool

_Note: **Do not skip this step!** If anything happens to your pool, this cache file can help you. Later in the guide you will save this file onto a permanent location in your new system._

```bash:title=bash
zpool export zroot
zpool import -o cachefile=/var/tmp/zpool.cache zroot
```


### Step 6: Create Filesystems

_Note: You may change the options and filesystems. You can add more or less, it’s up to you. This is what is advised on the FreeBSD Wiki however._

```bash:title=bash
zfs create zroot/usr
zfs create zroot/usr/home
zfs create zroot/var
zfs create -o compression=on -o exec=on -o setuid=off zroot/tmp
zfs create -o compression=lzjb -o setuid=off zroot/usr/ports
zfs create -o compression=off -o exec=off -o setuid=off zroot/usr/ports/distfiles
zfs create -o compression=off -o exec=off -o setuid=off zroot/usr/ports/packages
zfs create -o compression=lzjb -o exec=off -o setuid=off zroot/usr/src
zfs create -o compression=lzjb -o exec=off -o setuid=off zroot/var/crash
zfs create -o exec=off -o setuid=off zroot/var/db
zfs create -o compression=lzjb -o exec=on -o setuid=off zroot/var/db/pkg
zfs create -o exec=off -o setuid=off zroot/var/empty
zfs create -o compression=lzjb -o exec=off -o setuid=off zroot/var/log
zfs create -o compression=gzip -o exec=off -o setuid=off zroot/var/mail
zfs create -o exec=off -o setuid=off zroot/var/run
zfs create -o compression=lzjb -o exec=on -o setuid=off zroot/var/tmp
```


### Step 7: Create SWAP partition

_Note: You may change the SWAP size depending on your usage. General rule of thumb is twice the amount of RAM, but whether that applies or not still today is a matter of preference. Checksumming will also be disabled, for obvious reasons._

```bash:title=bash
zfs create -V 4G zroot/swap
zfs set org.freebsd:swap=on zroot/swap
zfs set checksum=off zroot/swap
```


### Step 8: Fixing /usr/home permissions

```bash:title=bash
chmod 1777 /mnt/tmp
cd /mnt ; ln -s usr/home home
chmod 1777 /mnt/var/tmp
```


### Step 9: Installing FreeBSD

No need to change anything here.

[^//]: Fix this so "do" is considered part of the same output!
```bash:title=bash
sh
cd /usr/freebsd-dist
export DESTDIR=/mnt
for file in base.txz lib32.txz kernel.txz doc.txz ports.txz src.txz;
  do (cat $file | tar --unlink -xpJf - -C ${DESTDIR:-/}); done
```


### Step 10: Backing up zpool.cache

_Note: **Make sure to not skip this step.**_

```bash:title=bash
cp /var/tmp/zpool.cache /mnt/boot/zfs/zpool.cache
```


### Step 11: Configuring FreeBSD

This will help get your FreeBSD system up and running.

```bash:title=bash
echo 'zfs_enable="YES"' >> /mnt/etc/rc.conf
echo 'ifconfig_re0="DHCP"' >> /mnt/etc/rc.conf
echo 'hostname="pc.mydomain.local"' >> /mnt/etc/rc.conf.local
echo 'zfs_load="YES"' >> /mnt/boot/loader.conf
echo 'vfs.root.mountfrom="zfs:zroot"' >> /mnt/boot/loader.conf
touch /mnt/etc/fstab
```

_Note: If your ethernet card is not a Realtek card and is an Intel card, use ```em[n]```. Otherwise, check out [12.8 Setting Up Network Interface Cards](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/config-network-setup.html)._


### Step 12: Unmounting and fixing mountpoints

No need to change anything here.

```bash:title=bash
zfs set readonly=on zroot/var/empty
zfs umount -af
zfs set mountpoint=legacy zroot
zfs set mountpoint=/tmp zroot/tmp
zfs set mountpoint=/usr zroot/usr
zfs set mountpoint=/var zroot/var
```

_Note: You may need to exit out of the user and then re-login as root if zroot does not unmount._


### Step 13: Post Installation

Reboot!

### Sources
[Root on ZFS FreeBSD 9](http://www.aisecure.net/2011/11/28/root-zfs-freebsd9/)  
[Installing FreeBSD Root on ZFS (Mirror) using GPT](http://wiki.freebsd.org/RootOnZFS/GPTZFSBoot/Mirror)  
