~~~
title: Xen Guest
~~~

# Links

Links manual guest creation:
- [Xen](https://help.ubuntu.com/community/Xen)
- [Xen on Foundation v8 model](https://wiki.linaro.org/LEG/Engineering/Virtualization/Xen_on_ARMv8_Foundation)

# Ready made images/isos/rootfs

## Links

- [Architectures/AArch64/F24/Installation](https://fedoraproject.org/wiki/Architectures/AArch64/F24/Installation)
- [Special Interest Group AArch64](https://wiki.centos.org/SpecialInterestGroup/AltArch/AArch64)

## Repos

- [OpenEmbedded](http://releases.linaro.org/openembedded/images/lng-armv8/)
  - [HiKey OpenEmbedded](https://builds.96boards.org/releases/reference-platform/openembedded/hikey/16.06/rpb/)
- [Debian/Ubuntu](https://wiki.debian.org/Arm64Port#Pre-built_Rootfses)
- [OpenSuse](https://en.opensuse.org/Portal:ARM/AArch64)
- [CentOS](http://mirror.centos.org/altarch/7/isos/aarch64/)
- [Fedora](https://ftp-stud.hs-esslingen.de/pub/fedora-secondary/releases/24/Everything/aarch64/)
  - [Fedora Cloud Images](https://ftp-stud.hs-esslingen.de/pub/fedora-secondary/releases/24/CloudImages/aarch64/images/)
  - [Fedora Server](https://ftp-stud.hs-esslingen.de/pub/fedora-secondary/releases/24/Server/)

# Guest console

`CTRL` + `]`

german layout: `STRG` + `ALT GR` + `]`

# Xen on ARM Guest Types

There is only one Xen guest type on ARM. This is very different from Xen on x86. The ARM Xen guest uses ARM virtualization extensions. With other words Xen on ARM guests use hardware assistet virtualization. Paravirtualization is only used for I/O.

# Creating OpenEmbedded Guest (tiny guest)

following this instructions: https://wiki.linaro.org/LEG/Engineering/Virtualization/Xen_on_ARMv8_Foundation#Create_a_DomU_guest.2C_boot_and_test

#### Creating the OpenEmbedded guest image (can be done on any linux host)

1. Download and create the disk image

~~~
host$ wget http://releases.linaro.org/15.06/openembedded/aarch64/linaro-image-minimal-genericarmv8-20150618-754.rootfs.tar.gz
host$ dd if=/dev/zero bs=1M count=128 of=disk.img
host$ mkfs.ext3 disk.img
~~~

2. Copy OpenEmbedded to disk image

~~~
host# mount -o loop disk.img /mnt
host# tar -C /mnt -xaf linaro-image-minimal-genericarmv8-20150618-754.rootfs.tar.gz
host# umount /mnt
~~~

#### Setting up guest

1. Create guest directory

~~~
# mkdir -p /opt/vms/openembedded
# cp disk.img -t /opt/vms/openembedded
# cp linux-kernel-xen -t /opt/vms/openembedded
~~~

- The kernel has to be compiled with Xen support
- The kernel can be shared with other vm's (e. g. create /opt/vms/kernels)
- `/opt/vms/openembedded` can be any mounted directory

2. Create vm config

~~~
# cd /etc/xen
# touch openembedded-pv.cfg
# vim openembedded-pv.cfg
~~~

config file:

~~~
# OpenEmbedded DomU

# Kernel paths for install
kernel="/opt/vms/openembedded/linux-kernel-xen"
extra = "console=hvc0 root=/dev/xvda rw"

# DomU settings
name = "openembedded"
memory = 48
vcpus = 1
maxvcpus = 1

# Path to HDD
disk = [
  'format=raw, vdev=xvda, access=w, target=/opt/vms/openembedded/disk.img'
]
~~~

The config file can have any name, but the following conventions are used:
- config files end with `.cfg`
- config files are named like the machine

3. Start VM

~~~
# xl create /etc/xen/openembedded.cfg
# xl list
# xl console openembedded 
~~~

- `openembedded` is the vm name

# Archlinux 

Based on: https://archlinuxarm.org/platforms/armv8/generic

1. Download and extract generic rootfs

~~~
wget http://archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
sudo tar -xvzf ArchLinuxARM-aarch64-latest.tar.gz -C archlinux
~~~

The tar archive has to be extracted with `root` rights.


2. Create empty image

~~~
dd if=/dev/zero of=archlinux.img bs=1M count=1024
mkfs.ext4 archlinux.img
~~~

3. Mount empty image and copy rootfs

~~~
sudo mount -o loop archlinux.img /mnt/removable/
cd archlinux/
sudo cp -R ./* -t /mnt/removable
~~~ 

4. Copy image to HiKey

   - for example by copying to a SD card

5. Setting

~~~
archlinux.cfg
# Archlinux

# Kernel paths for install
kernel="/opt/vms/kernels/linux-kernel-4.1-xen"
extra = "console=hvc0 root=/dev/xvda rw"

# DomU settings
name = "archlinux"
memory = 64
vcpus = 1
maxvcpus = 1

# Path to HDD
disk = [
  'format=raw, vdev=xvda, access=w, target=/opt/vms/archlinux/archlinux.img'
]
~~~

6. Start VM

~~~
xl create archlinux.cfg
xl console archlinux
~~~

user: alarm
password: alarm

user: root
password: root

# Debian

Based on: https://olimex.wordpress.com/2014/07/21/how-to-create-bare-minimum-debian-wheezy-rootfs-from-scratch/

1. Install support packages

~~~
$ sudo apt-get install qemu-user-static debootstrap binfmt-support
~~~

2. Choose Debian version and targetdir, by setting variables with the values.

~~~
$ targetdir=rootfs
$ distro=jessie
~~~

3. Build the first stage of Debian rootfs:

~~~
mkdir $targetdir
sudo debootstrap --arch=arm64 --foreign $distro $targetdir
~~~

4. Copy the `qemu-aarch64-static` binary into the right place for the binfmt packages to find it and copy in resolv.conf from the host.

~~~
sudo cp /usr/bin/qemu-aarch64-static $targetdir/usr/bin/
sudo cp /etc/resolv.conf $targetdir/etc
~~~

Now, we have a minimal Debian Rootfs

5. Enter in the chroot environment

~~~
sudo chroot $targetdir
~~~

6. Inside the chroot we need to set up the environment again

~~~
distro=jessie
export LANG=C
~~~

7. Setup second stage of deboostrap to install the packages we downloaded earlier

~~~
/debootstrap/debootstrap --second-stage
~~~

8. Update Debian package database

~~~
apt-get update
~~~

9. Set up locales dpkg scripts tend to complain otherwise, note in jessie you will also need to install the dialog package as well.

~~~
apt-get install locales dialog
dpkg-reconfigure locales
~~~

10. Install some useful packages inside the chroot

~~~
apt-get install vim openssl
~~~

11. Set a root password to be able to login later

~~~
passwd
~~~

e. g. root

12. Set a hostname

~~~
echo generic-arm64 > /etc/hostname
~~~

13. Enable serial console

~~~
echo T0:2345:respawn:/sbin/getty -L ttyS0 115200 vt100 >> /etc/inittab
~~~

14. Exit chroot and clean up

~~~
exit
sudo rm $targetdir/etc/resolv.conf 
sudo rm $targetdir/usr/bin/qemu-aarch64-static
~~~

15. Create disk Image

~~~
dd if=/dev/zero of=debian.img bs=1M count=440
sudo mkfs.ext4 debian.img
~~~

16. Copy rootfs to disk image

~~~
sudo mount -o loop debian.img /mnt/removable/
sudo cp -R $targetdir/* /mnt/removable/
sudo umount /mnt/removable
~~~

17. Transfer `debian.img` to the device and create a configuration file

~~~
$ cat debian.cfg
# Debian                                                                        
                                                                                
# Kernel paths for install                                                      
kernel="/opt/vms/kernels/linux-kernel-4.1-xen"
extra = "console=hvc0 root=/dev/xvda rw"
                                                                                
# DomU settings                                                                 
name = "debian"
memory = 128
vcpus = 1
maxvcpus = 1
                                                                                
# Path to HDD                                                                   
disk = [
  'format=raw, vdev=xvda, access=w, target=/opt/vms/debian/debian.img' 
]
~~~

# Ubuntu

1. See Debian

2. Choose Debian version and targetdir, by setting variables with the values.

~~~
$ targetdir=rootfs
$ distro=xenial
~~~

3. Download Ubuntu Keyring

~~~
sudo apt-get install ubuntu-archive-keyring
~~~

4. Build the first stage of Debian rootfs:

~~~
mkdir $targetdir
sudo debootstrap --arch=arm64 --keyring=/usr/share/keyrings/ubuntu-archive-keyring.gpg --foreign $distro $targetdir http://ports.ubuntu.com/ubuntu-ports/
~~~

5. Copy the `qemu-aarch64-static` binary into the right place for the binfmt packages to find it and copy in resolv.conf from the host.

~~~
sudo cp /usr/bin/qemu-aarch64-static $targetdir/usr/bin/
sudo cp /etc/resolv.conf $targetdir/etc
~~~

Now, we have a minimal Ubuntu Rootfs

6. Enter in the chroot environment

~~~
sudo chroot $targetdir
~~~

7. Inside the chroot we need to set up the environment again

~~~
distro=xenial
export LANG=C
~~~

8. Setup second stage of deboostrap to install the packages we downloaded earlier

:!: the switch to debian message can be safely ignored

~~~
/debootstrap/debootstrap --second-stage
~~~

9. Update Debian package database

~~~
apt-get update
~~~

10. Set up locales dpkg scripts tend to complain otherwise, note in jessie you will also need to install the dialog package as well.

~~~
apt-get install locales
dpkg-reconfigure locales
~~~

11. Install some useful packages inside the chroot

~~~
apt-get install vim openssl
~~~

12. Set a root password to be able to login later

~~~
passwd
~~~

e. g. root

13. Set a hostname

~~~
echo generic-arm64 > /etc/hostname
~~~

13. Enable serial console

~~~
echo T0:2345:respawn:/sbin/getty -L ttyS0 115200 vt100 >> /etc/inittab
~~~

14. Exit chroot and clean up

~~~
exit
sudo rm $targetdir/etc/resolv.conf 
sudo rm $targetdir/usr/bin/qemu-aarch64-static
~~~

15. Create disk Image

~~~
dd if=/dev/zero of=ubuntu.img bs=1M count=320
sudo mkfs.ext4 ubuntu.img
~~~

16. Copy rootfs to disk image

~~~
sudo mount -o loop ubuntu.img /mnt/removable/
sudo cp -R $targetdir/* /mnt/removable/
sudo umount /mnt/removable
~~~

17. Transfer `ubuntu.img` to the device and create a configuration file

~~~
$ cat ubuntu.cfg
# Ubuntu                                                                        
                                                                                
# Kernel paths for install                                                      
kernel="/opt/vms/kernels/linux-kernel-4.1-xen"
extra = "console=hvc0 root=/dev/xvda rw"
                                                                                
# DomU settings                                                                 
name = "ubuntu"
memory = 256
vcpus = 1
maxvcpus = 1
                                                                                
# Path to HDD                                                                   
disk = [
  'format=raw, vdev=xvda, access=w, target=/opt/vms/ubuntu/ubuntu.img' 
] 
~~~

# OpenSUSE

user: root
password: linux

image repository: http://download.opensuse.org/ports/aarch64/tumbleweed/images/

Image types:

E20 -> Enlightenment Desktop Environment
JeOS -> Just enough operating system
LXQT -> LXQT Desktop Envrionment
x11 -> ?
XFCE -> XFCE Desktop Environment

1. Download image and extract the image

~~~
wget http://download.opensuse.org/ports/aarch64/tumbleweed/images/openSUSE-Tumbleweed-ARM-JeOS.aarch64-rootfs.aarch64-Current.tbz
mkdir rootfs
sudo tar -xjf openSUSE-Tumbleweed-ARM-JeOS.aarch64-rootfs.aarch64-2017.01.31-Build1.1.tbz -C rootfs
~~~

2. Create an empty image

~~~
dd if=/dev/zero of=opensuse.img bs=1M count=2500
sudo mkfs.ext4 opensuse.img 
~~~

3. Copy the content of rootfs into the image

~~~
sudo mount -o loop opensuse.img /mnt/removable/
sudo cp -R rootfs/* -t /mnt/removable/
sudo umount /mnt/removable
~~~

4. Config file for opensuse

~~~
$ cat opensuse.cfg
# OpenSUSE                                                                      
                                                                                
# Kernel paths for install                                                      
kernel="/opt/vms/kernels/linux-kernel-4.1-xen"
extra = "console=hvc0 root=/dev/xvda rw"
                                                                                
# DomU settings                                                                 
name = "opensuse"
memory = 256
vcpus = 1
maxvcpus = 1
                                                                                
# Path to HDD                                                                   
disk = [
  'format=raw, vdev=xvda, access=w, target=/opt/vms/opensuse/opensuse.img'
] 
~~~

# How to find out the password of a downloaded image?

One way is to guess the passwords manually. A look in `/etc/passwd` reveals what users accounts are available. 

Most of them are very simple. Therefor it is possible to use the tool 'John the Ripper' and simply brute force the password. This could take several minutes (1-30+).

For example:

~~~
unshadow etc/passwd etc/shadow > /home/<username>/password.db
john password.db
john --show password.db
~~~

-----

# The somewhere working section

# Alpine 

Using the mini root filesystem is somewhere working:
https://www.alpinelinux.org/downloads/


# Fedora

Images: http://libguestfs.org/download/builder/

1. Download image and extract the image

~~~
wget http://libguestfs.org/download/builder/fedora-25-aarch64.xz
unxz fedora-25-aarch64.xz
~~~

2. Mount the root file system

The fedora-25-aarch64 has a GPT partition. For that reason fedora-25-aarch64 contains several partitions. (source: https://tinyapps.org/docs/mount_partitions_from_disk_images.html)

~~~
sudo losetup --partscan --find --show fedora-25-aarch64
sudo fdisk -l
~~~

The output shows two Linux filesystems on `/dev/loop0p2` and `/dev/loop0p4`. 

~~~
/dev/loop0p1    2048   221183  219136  107M EFI System
/dev/loop0p2  221184  2318335 2097152    1G Linux filesystem
/dev/loop0p3 2318336  3577855 1259520  615M Linux swap
/dev/loop0p4 3577856 12578815 9000960  4.3G Linux filesystem
~~~

The rootfs is in either one of the two Linux filesystems. Presumably in `/dev/loop0p4`, because it is the bigger one. 

~~~
sudo mount /dev/loop0p4 /mnt/removable/
ls /mnt/removable/
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
~~~

The `ls` output shows that is the root system.

3. Copy the rootfs into a folder

~~~
mkdir rootfs
sudo cp -R /mnt/removable/* -t rootfs
~~~

4. Figure out the size of the roofs

~~~
sudo du -h rootfs/
....
1.1G	rootfs/
~~~

In this case the rootfs folder is 1.1 GB in size.

5. Unmount `/dev/loop0p4` which is mounted to `/mnt/removable`

~~~
sudo umount /mnt/removable
~~~

6. Create image

The image has to be larger than 1.1 GB. To be safe the image more than double the size.

~~~
dd if=/dev/zero of=fedora-25.img bs=1M count=2400
sudo mkfs.ext4 fedora-25.img
~~~

7. Modify fstab in rootfs

~~~
sudo cat rootfs/etc/fstab

#
# /etc/fstab
# Created by anaconda on Wed Nov 23 22:42:16 2016
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=d25108a6-bf8d-4487-8e66-1a044012546a /                       xfs     defaults        0 0
UUID=833aaa14-55e5-4f89-99c7-8afeca8492b1 /boot                   ext4    defaults        1 2
UUID=AE18-948B          /boot/efi               vfat    umask=0077,shortname=winnt 0 2
UUID=b37fc4a4-ee38-449d-a61b-fff063304c63 swap                    swap    defaults        0 0
~~~

we need only the rootsystem. In addition, the UUID and file system format of the newly created image file (`fedora-25.img`) is different.

To show the UUID and file system format of the image.

~~~
blkid fedora-25.img
~~~

Take the values and create a new fstab

~~~
sudo sh -c 'echo "UUID=<the-new-image-uuid>    /                        <file-system-format>     defaults          0 0" > rootfs/etc/fstab'
~~~

for example:

~~~
sudo sh -c 'echo "UUID=4a72f425-0e86-414f-99f3-355042a20a12  /        ext4      defaults      0  0" > rootfs/etc/fstab'
~~~

8. Set password

~~~
$ openssl passwd -6 <pasword>
~~~

For example:

~~~
$ openssl passwd -1 toor
$1$IifXbiIv$syQqUC.O.6wKQF.VbXarM1
~~~

:!: The `openssl passwd -1 toor` command generates a insecure md5 hash of the string `toor`. Do not use MD5 in prodcution envrionments. Use instead `openssl passwd -6 <your-strong-password>`. The `-6` parameter generates an, as of 2017 secure, SHA256/512 hash.

Open the etc/shadow to change the password

~~~
sudo vim rootfs/etc/shadow
~~~

Replace the existing password with the new hash:

~~~
root:<password-hash-to-replace>::...
~~~


9. Copy files to new image

~~~
sudo mount -o loop fedora-25.img /mnt/removable/
sudo cp -R rootfs/* -t /mnt/removable/
sudo umount /mnt/removable
~~~

10. Create a configuration file

~~~
$ cat fedora.cfg
# Fedora                                                                        
                                                                                
# Kernel paths for install                                                      
kernel="/opt/vms/kernels/linux-kernel-4.1-xen"
extra = "console=hvc0 root=/dev/xvda rw"
                                                                                
# DomU settings                                                                 
name = "fedora"
memory = 384
vcpus = 1
maxvcpus = 1
                                                                                
# Path to HDD                                                                   
disk = [
  'format=raw, vdev=xvda, access=w, target=/opt/vms/fedora/fedora-25.img'
]
~~~

# CentOS 

1. Download image and extract the image

~~~
wget http://libguestfs.org/download/builder/centos-7.2-aarch64.xz
unxz centos-7.2-aarch64
~~~

For the rest of the steps look at the fedora section. The image creation process is the same as for Fedora.

10. CentOS configuration file

~~~
$ cat centos.cfg 
# Centos

# Kernel paths for install
kernel="/opt/vms/kernels/linux-kernel-4.1-xen"
extra = "console=hvc0 root=/dev/xvda rw"

# DomU settings
name = "centos"
memory = 384
vcpus = 1
maxvcpus = 1

# Path to HDD
disk = [
  'format=raw, vdev=xvda, access=w, target=/opt/vms/fedora/centos-7.img'
]
~~~

CentOS + Fedora -> problem whith login

# Creating Android Guest

:!: Android has problems while booting up!

## General

Android uses Yaffs1/2, which has to be transformed to work with it.

(source: http://stackoverflow.com/questions/11648748/mount-android-emulator-images#12282909)

boot.img contains 
- kernel
- ramdisk

(source: http://unix.stackexchange.com/questions/64628/how-to-extract-boot-img/65316#65316)

userdata.img contains:
- unit & system component tests

system.img contains:
- filesystem

~~~
system]# ls
app  build.prop  fake-libs    fonts      lib    lost+found  priv-app  usr     xbin
bin  etc         fake-libs64  framework  lib64  media       tts       vendor
~~~

simg2img - Tool to convert Android sparse images to raw images

### simg2img

1. build simg2img

~~~
$ git clone https://github.com/anestisb/android-simg2img
$ cd android-simg2img
$ make
~~~

2. convert images

using android build 36 (https://builds.96boards.org/snapshots/hikey/linaro/aosp-master/36?dl=/snapshots/hikey/linaro/aosp-master/36/)

convert the `system` image

~~~
$ /home/user/bin/android-simg2img/simg2img system.img system.raw.img
$ file system.raw.img
# mkdir -p /mnt/system
# mount -o loop system.raw.img /mnt/system
# cd /mnt/system
# ls
~~~

convert the `userdata` image

~~~
$ /home/user/bin/android-simg2img/simg2img userdata.img userdata.raw.img
$ file userdata.raw.img
# mkdir -p /mnt/system
# mount -o loop system.raw.img /mnt/system
# cd /mnt/system
# ls
~~~


