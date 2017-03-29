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

#### Creating guest image (can be done on any linux host)

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
extra = "console=hvc0 root=/dev/xvda ro"

# DomU settings
name = "openembedded"
memory = 512
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

# Creating CentOS Guest



# Creating Android Guest

:!: Does not work!

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


