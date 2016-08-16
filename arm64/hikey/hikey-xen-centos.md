# HiKey Xen + CentOS

"Beware that to run Xen on HiKey you need the 96Boards UART Serial Adapter, available [here](http://linaro.co/uart-seeed)."

## Build Kernel 4.1 

The instructions are based on:
- https://wiki.xen.org/wiki/HiKey
- https://wiki.linaro.org/LEG/Engineering/Virtualization/Xen_on_ARMv8_Foundation

~~~
# clone the kernel tree
user@host$ git clone https://github.com/96boards/linux.git 96boards_linux
user@host$ cd 96boards_linux 
# this is the android-hikey-linaro-4.1 commit mentioned in the article
user@host: ~/96boards_linux$ git checkout 9e740973c9a71214a5873d2a5de42e5be7522989
user@host: ~/96boards_linux$ git show HEAD

# setup cross build environment
user@host: ~/96boards_linux$ export ARCH="arm64"
user@host: ~/96boards_linux$ export CROSS_COMPILE="aarch64-linux-gnu-"
user@host: ~/96boards_linux$ make hikey_defconfig #optional

# show config file
user@host: ~/96boards_linux$ less .config

# download Xen config file
user@host: ~/96board_linux$ wget http://xenbits.xen.org/people/sstabellini/config-hikey
user@host: ~/96board_linux$ rm .config #optional
user@host: ~/96board_linux$ mv config-hikey .config

# cross build the kernel
make -j24 Image hisilicon/hi6220-hikey.dtb

# Copy files to SD card (FAT format)
user@host$ wget http://mirror.centos.org/altarch/7/isos/aarch64/CentOS-7-aarch64-rolling.img.xz
# alternatively CentOS-7-aarch64-rootfs-1606.tar.xz could be used
user@host$ sudo dd if=CentOS-7-aarch64-rolling.img of=/dev/sdX bs=4M
user@host$ mkdir efi
# mount efi FAT partition to efi
user@host$ sudo mount /dev/sdX1 efi
user@host$ cd 96board_linux
user@host: ~/96board_linux$ cp arch/arm64/boot/Image ../efi
user@host: ~/96board_linux$ cp arch/arm64/boot/dts/hisilicon/hi6220-hikey.dtb ../efi
host: ~$ cd xen
host: ~/xen$ cp xen/xen ../efi
host: ~/xen$ cd ../efi
host: ~$ mv xen xen.efi
#startup.nsh and xen.cfg is from the wiki article
host: ~/efi$ ls 
EFI  Image  startup.nsh xen.cfg  xen.efi
~~~

## Misc

- according to: http://www.xenproject.org/help/questions-and-answers/build-xen-on-hikey-board/voted.html

~~~
make dist-xen XEN_TARGET_ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
~~~

- DTB Xen solution
http://www.gossamer-threads.com/lists/xen/devel/433472

~~~
options=dom0_mem=1024M dom0_max_vcpus=8 conswitch=x console=dtuart
dtuart=/smb/uart [at] f711300
kernel=Image console=hvc root=/dev/mmcblk0p9 rootwait rw 3
dtb=hi6220-hikey.dtb 
~~~

Login credentials:
- user:     root
- password: centos


Console log: (XEN) I/O virtualisation disabled 

The UEFI trick seems only to work with an UEFI build from Nov 2015

.nsh -> stands for network shell script

## CentOS setting up Xen

See [HiKey Xen Build Tools](hikey-xen-build-tools.md) to build the Xen tools.

The missing files can be copied over by using a flash drive.

### Missing Packages

List of mssing libraries:
- `libfdt`
- `libyajl`
- `pixman`
- `net-tools`

CentOS 7 Aarch64 Packages: http://buildlogs.centos.org/centos/7/os/aarch64/Packages/

### Finding/Installing Missing Packages

If libraries are missing, then they are may just not installed on the system.

For example:
`libfdt` and `libyajl` is mssing.

The solution is simple. Downloading those packages from the distribution repository either directly on the board or downloading them manually and installing them afterwards with the low-level packaging tools `rpm` or `dpkg`.

CentOS 7 Aarch64 Packages: http://buildlogs.centos.org/centos/7/os/aarch64/Packages/

#### XL libraries missing

Show which libraries are missing:

~~~
# ldd `which xl`
~~~

use find to search for the libraries:

~~~
# find / -name "libxl*"
~~~

e. g. `/usr/local/lib/libxlutil.so `

either `/usr/local/` is already in the linker (ldd) path and running `ldconfig` does the work. Or you have to add `/usr/local` (see: http://www.cyberciti.biz/tips/linux-shared-library-management.html).

#### Starting Services

~~~
# systemctl status -l xenstored
...
Jan 01 02:05:17 localhost xencommons[3167]: /usr/local/lib/xen/bin/qemu-system-i386: error while loading shared libraries: libpixman-1.so.0: cannot open shared object file: No such file or directory
...
~~~

With other words libpixman-1.so.0 is not installed. The librarie can be found in the `pixman` RPM package.

After installing `pixman` the `xl list` command can be used.

~~~
# systemctl enable xencommons
# systemctl start xencommons
# systemctl status xencommons
~~~

### Misc

Logs will only be created if the directory `console` exists. Furthermore `/local/domain/${domain-id}/console/tty` will hava a `pts` assigned.

~~~
mkdir /var/log/xen/console
~~~

~~~
vim /etc/default/xencommons
#XENCONSOLED_TRACE=[none|guest|hv|all]
XENCONSOLED_TRACE=all
~~~
