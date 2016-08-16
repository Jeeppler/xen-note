# Build xen-tools

based on:
- http://wiki.xen.org/wiki/Xen_ARM_with_Virtualization_Extensions/CrossCompiling#Build_arm64_tools

1. Rootfs is already on the SD-Card after following [Build Kernel 4.1](hikey-xen-centos.md)

2. Build system: `debian testing (stretch)`

~~~
# apt-get install sbuild
# sbuild-adduser $USER
~~~

3. Creating a base chroot

The `httpredir` is a special debian service.

~~~
sudo sbuild-createchroot jessie /srv/chroot/jessie-arm64-cross http://httpredir.debian.org/debian
~~~

4. Correcting the name

~~~
# mv /etc/schroot/chroot.d/jessie-amd64-sbuild-* /etc/schroot/chroot.d/jessie-arm64-cross
# vim /etc/schroot/chroot.d/jessie-arm64-cross
[jessie-arm64-cross]
type=directory
union-type=overlay
description=Debian jessie/arm64 crossbuild
directory=/srv/chroot/jessie-arm64-cross
groups=root,sbuild
root-groups=root,sbuild
profile=default
~~~

5. Install basic utilities

~~~
# schroot -c jessie-arm64-cross
(chroot)# apt-get install vim wget sudo less curl
~~~

6. Disable recommends

~~~
vim /etc/apt/apt.conf.d/30norecommends
~~~

add

~~~
APT::Install-Recommends "0";
APT::Install-Suggests "0";
~~~

7. Add architecture

Cross-toolchains for jessie are available, only cross-binutils and cross-gcc-dev (crosstoolchain builder package) are in the main archive. Other packages come from an external repository. (source: https://wiki.debian.org/CrossToolchains)

~~~
(chroot)# vim /etc/apt/sources.list.d/crsstools.list
deb http://emdebian.org/tools/debian/ jessie main
(chroot)# curl http://emdebian.org/tools/debian/emdebian-toolchain-archive.key | sudo apt-key add -
~~~

add the architecture

~~~
(chroot)# dpkg --add-architecture arm64
(chroot)# apt-get update
(chroot)# apt-get install crossbuild-essential-arm64
~~~

8. Instal the build dependencies required to build Xen:

~~~
(chroot)# apt-get install python gettext uuid-dev libncurses5-dev:arm64

(chroot)# apt-get install libc6-dev:arm64 libncurses-dev:arm64 uuid-dev:arm64 libglib2.0-dev:arm64 libssl-dev:arm64 libssl-dev:arm64 libaio-dev:arm64 libyajl-dev:arm64 python gettext gcc git libpython2.7-dev:arm64 libfdt-dev:arm64 autotools-dev libpixman-1-0-dev:arm64
(chroot)# exit
~~~

9. Cross compile tools:

~~~
(chroot)# export pixman_LIBS="libpixman-1-0-dev"
(chroot)# CONFIG_SITE=/etc/dpkg-cross/cross-config.arm64 ./configure --build=x86_64-unknown-linux-gnu --host=aarch64-linux-gnu
(chroot)# make dist-tools CROSS_COMPILE=aarch64-linux-gnu- XEN_TARGET_ARCH=arm64
~~~

You should now have a dist/install directory containing the installed bits which can be copied into your arm64 rootfs. (source: https://wiki.xen.org/wiki/Xen_ARM_with_Virtualization_Extensions/CrossCompiling#Using_sbuild_and_schroot)
