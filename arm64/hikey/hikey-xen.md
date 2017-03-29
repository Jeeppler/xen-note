# Xen on HiKey

https://wiki.linaro.org/LEG/Engineering/Virtualization/Xen_on_ARMv8_Foundation

## How to Cross compile

based on: 
- https://wiki.linaro.org/LEG/Engineering/Virtualization/Xen_on_ARMv8_Foundation
- https://wiki.linaro.org/LEG/Engineering/Virtualization/Xen_ARM_Guide
- http://source.android.com/source/devices.html

### Convention

Following convention is followed throughout this wiki

host$ = run as a regular user on host machine
host# = run as root user on host machine (can use sudo if you prefer)
chroot> = run as root user in chroot environment
model> = run as root user in a running Foundation Model

### Environment

Debian "Stretch" testing

### Install ARM Toolchain

host# apt-get install make gcc gcc-aarch64-linux-gnu gcc-arm-linux-gnueabi git libncurses5-dev python bc mtools

### Build Xen

~~~
host$ git clone git://xenbits.xen.org/xen.git xen
host$ cd xen

# Optional 
# Building a specific release
host$ git tag -l
host$ git checkout tags/<tag_name>           //replace <tag_name> with e.g. RELEASE-4.8.0

host$ make xen XEN_TARGET_ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- DEBUG=y
~~~

### Build Kernel

Kernels are build from the same repositories with different configuration files. The kernel has to be compiled with Xen support. Therefor a Xen configuration file is needed.

The HiKey board tree is not in the official Linux kernel repository, therefor the HiKey repository has to be used to build the kernel.

#### 4.4

user@host$ git clone https://android.googlesource.com/kernel/hikey-linaro

## Compile Xen on HiKey

The following instructions work for Xen:
http://wiki.xenproject.org/wiki/Compiling_Xen_From_Source

- problem ocamlopt is not available in Debian 8, therefor the Xen utils can not be compiled
- ocamlopt is available with Debian testing (repositories), but tools can still not be compiled

## Compiling only Xen on Debian HiKey

Debian has already all required packages installed on the system.make

~~~
$ git clone git://xenbits.xen.org/xen.git
$ ./configure --enable-systemd
# make dist-xen DEBUG=y
# make install-xen
~~~

:!: Unclear if DEBUG=y works
