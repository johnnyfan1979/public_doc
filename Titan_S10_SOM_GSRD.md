

**Setting Up Yocto Build System** 

1. The command to install the required packages on Ubuntu 20.04 is:

```bash
sudo apt-get update

sudo apt-get install openssh-server mc libgmp3-dev libmpc-dev gawk wget git diffstat unzip texinfo gcc \
build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping \
python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev zstd \
liblz4-tool git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison xinetd \
tftpd tftp nfs-kernel-server libncurses5 libc6-i386 libstdc++6:i386 libgcc++1:i386 lib32z1 \
device-tree-compiler curl mtd-utils u-boot-tools net-tools swig -y
```

On Ubuntu 20.04 you will also need to point the /bin/sh to /bin/bash, as the default is a link to /bin/dash:

```bash
sudo ln -sf /bin/bash /bin/sh
```
2. Clone the Yocto script and prepare the build:

```bash
git clone -b titan-s10-som-QPDS24.3 https://github.com/terasic/gsrd-socfpga
cd gsrd-socfpga
. titan_s10_som-gsrd-build.sh
build_setup
```

**Build Yocto**

Remove reference to patch which was retired after 24.2 tag was applied:

```bash
sed -i '/fix-potential-signed-overflow-in-pointer-arithmatic.patch/d' meta-intel-fpga-refdes/recipes-connectivity/openssh/openssh_%.bbappend
```
Build Yocto:

```bash
bitbake_image
```
Gather files:
```bash
package
```



Once the build is completed successfully, you will see the following two folders are created:

- titan_s10_som-gsrd-rootfs: area used by OpenEmbedded build system for builds. Description of build directory structure - https://docs.yoctoproject.org/ref-manual/structure.html#the-build-directory-build
- titan_s10_som-gsrd-images: the build script copies here relevant files built by Yocto from the titan_s10_som-gsrd-rootfs/tmp/deploy/images/titan_s10_som folder, but also other relevant files.



The two most relevant files created in the gsrd-socfpga/titan_s10_som-gsrd-images folder are:


| File | Description      |
| ---- | ---------- |
|sdimage.tar.gz | SD Card Image |
|   u-boot-titan_s10_som-gsrd-atf/u-boot-spl-dtb.hex |U-Boot SPL Hex file         |



**Source list**


|Item   |Version     |Source     |
|:--------|:--------|--------|
|Linux Kernel       |6.6.37-lts | https://github.com/terasic/linux-socfpga/tree/titan-s10-som_v1.0       |
| U-boot       | 2024.04        | https://github.com/terasic/u-boot-socfpga/tree/titan-s10-som_v1.0       |
| ATF       | 2.11.0        | https://github.com/altera-opensource/arm-trusted-firmware/tree/QPDS24.3_REL_GSRD_PR       |
| OS       | Poky 5.0.5        | -       |
















