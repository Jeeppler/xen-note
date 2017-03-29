# Build xen-tools

:!: Make sure that the Xen and the Xen tool version are identical.

OCaml is used to build oxenstored. Otherwise cxenstored is build and used.

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

4. Adjusting the name of the chroot environment

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
   # cd xen
   # schroot -c jessie-arm64-cross
   (chroot)# apt-get install vim wget sudo less curl
   ~~~

6. Disable recommends

    ~~~
    (chroot)# vim /etc/apt/apt.conf.d/30norecommends
    ~~~

    add

    ~~~
    APT::Install-Recommends "0";
    APT::Install-Suggests "0";
    ~~~

7. Add architecture

    Cross-toolchains for jessie are available, only cross-binutils and cross-gcc-dev (crosstoolchain builder package) are in the main archive. Other packages are from an external repository. (source: https://wiki.debian.org/CrossToolchains)

    ~~~
    (chroot)# vim /etc/apt/sources.list.d/crosstools.list
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
    (chroot)# apt-get install libc6-dev:arm64 uuid-dev:arm64 libglib2.0-dev:arm64 libssl-dev:arm64 libssl-dev:arm64 libaio-dev:arm64 libyajl-dev:arm64 python gettext gcc git libpython2.7-dev:arm64 libfdt-dev:arm64 autotools-dev libpixman-1-dev:arm64 iasl libncurses5-dev:arm64
    ~~~

9. Optional: Install Dependencies

    ~~~
    (chroot)# apt-get install ocaml camlp4 bison flex ocaml-findlib pandoc e2fslibs-dev:arm64 markdown fig2ps
    ~~~ 

10. Cross compile tools:

    ~~~
    (chroot)# export pixman_LIBS="libpixman-1-0-dev"
    (chroot)# CONFIG_SITE=/etc/dpkg-cross/cross-config.arm64 ./configure --build=x86_64-unknown-linux-gnu --host=aarch64-linux-gnu
    (chroot)# make dist-tools CROSS_COMPILE=aarch64-linux-gnu- XEN_TARGET_ARCH=arm64
    ~~~

# Copy files (install tools)

You should now have a `dist/install` directory in the xen folder containing the installed bits which can be copied into your arm64 rootfs. (source: https://wiki.xen.org/wiki/Xen_ARM_with_Virtualization_Extensions/CrossCompiling#Using_sbuild_and_schroot)

## Troubleshoot

This error occurs if Xen and Xen tool versions are different.

~~~
# xl create <vm-name>.cfg
Parsing config from android-6-g2.cfg
libxl: error: libxl_create.c:562:libxl__domain_make: domain creation fail: Permission denied
libxl: error: libxl_create.c:899:initiate_domain_create: cannot make domain: -3
libxl: error: libxl_create.c:562:libxl__domain_make: domain creation fail: Permission denied
~~~
