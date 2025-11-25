# DE25 Nano: Build Linux image from scratch

## 0. OS: Ubuntu-22.04 (Linux PC or Windows WSL)

## 1. Set Up Environment

1. Install the requirement packages on Ubuntu

   ```shell
   sudo apt-get update
   sudo apt install make bison flex python3-dev libssl-dev swig
   sudo apt install u-boot-tools
   sudo apt install qemu-user-static
   sudo apt install git
   sudo apt install build-essential
   ```

2. Create top folder

   ```shell
   mkdir de25-nano.sdmmc
   cd de25-nano.sdmmc
   export TOP_FOLDER=`pwd`
   ```

3. Download and setup the toolchain as follows

   ```shell
   cd $TOP_FOLDER
   wget https://developer.arm.com/-/media/Files/downloads/gnu/11.2-2022.02/binrel/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu.tar.xz
   tar xf gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu.tar.xz
   rm -f gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu.tar.xz
   export PATH=`pwd`/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu/bin:$PATH
   export ARCH=arm64
   export CROSS_COMPILE=aarch64-none-linux-gnu-
   ```

## 2. Build Arm Trusted Firmware

```shell
cd $TOP_FOLDER
rm -rf arm-trusted-firmware
git clone -b de25_nano_revA_v1.0 https://github.com/terasic/arm-trusted-firmware arm-trusted-firmware
cd arm-trusted-firmware
make -j $(nproc) PLAT=agilex5 bl31
cd ..
```

The following file is created:

- $TOP_FOLDER/arm-trusted-firmware/build/agilex5/release/bl31.bin (to be used for u-boot.itb generation)

## 3. Build U-Boot

```shell
cd $TOP_FOLDER
rm -rf u-boot-socfpga
git clone -b de25_nano_revA_v1.0 https://github.com/terasic/u-boot-socfpga u-boot-socfpga
cd u-boot-socfpga
make mrproper
make socfpga_agilex5_de25_nano_defconfig
# link to ATF
ln -s ../arm-trusted-firmware/build/agilex5/release/bl31.bin
make -j $(nproc)
```

The following files are created:

- $TOP_FOLDER/u-boot-socfpga/u-boot.itb
- $TOP_FOLDER/u-boot-socfpga/spl/u-boot-spl-dtb.hex

## 4. Create the Boot Script

```shell
cd $TOP_FOLDER
rm -rf uboot-script && mkdir uboot-script && cd uboot-script
wget https://releases.rocketboards.org/2023.12/qspi/agilex5/agilex5_uboot.txt
wget https://releases.rocketboards.org/2021.11/uboot-script/agilex/uboot_script.its
mv agilex5_uboot.txt uboot.txt
vi uboot.txt	# content see bloew
mkimage -f uboot_script.its boot.scr.uimg
cd ..
```

**Note: If users want to access FPGA LED for HPS, please add 'bridge enable' in uboot.txt**

> uboot.txt:
>
> ```shell
> echo "Trying to boot Linux from device ${target}";
> 
> if test ${target} = "mmc0"; then
>         bridge enable;
>         echo "Found kernel in mmc0";    
>         mmc rescan;
>         fatload mmc 0:1 ${kernel_addr_r} Image;
>         fatload mmc 0:1 ${fdt_addr_r} socfpga_agilex5_de25_nano.dtb;
>         setenv bootargs "console=ttyS0,115200 root=${mmcroot} rw rootwait";
>         booti ${kernel_addr_r} - ${fdt_addr_r};
>         exit;
> fi
> ```

The following file is created:

- $TOP_FOLDER/uboot-script/boot.scr.uimg

## 5. Build Linux Kernel

```shell
cd $TOP_FOLDER
rm -rf linux-socfpga
git clone -b de25_nano_revA_v1.0 https://github.com/terasic/linux-socfpga linux-socfpga
cd linux-socfpga
cp de25-nano.config .config
make -j $(nproc) Image modules dtbs
```

The following files are created:

- $TOP_FOLDER/linux-socfpga/arch/arm64/boot/Image
- $TOP_FOLDER/linux-socfpga/arch/arm64/boot/dts/intel/socfpga_agilex5_de25_nano.dtb

## 6. Build Ubuntu 22.04.3

1. Prepare rootfs folder

   ```shell
   cd $TOP_FOLDER
   mkdir -p rootfs
   cd rootfs
   ```

2. Switch to root privilege

   ```shell
   sudo -s
   ```

3. Download the Ubuntu 22.04.3 root filesystem

   ```shell
   wget -c https://cdimage.ubuntu.com/ubuntu-base/releases/22.04.3/release/ubuntu-base-22.04.3-base-arm64.tar.gz
   tar xvf ubuntu-base-22.04.3-base-arm64.tar.gz
   rm ubuntu-base-22.04.3-base-arm64.tar.gz
   ```

4. Copy qemu-user-static

   ```shell
   cp /usr/bin/qemu-aarch64-static usr/bin/
   ```

5. Modify etc/apt/sources.list to un-comment all the repositories except the ones starting with deb-src

   ```shell
   sed -i 's%^# deb %deb %' etc/apt/sources.list
   ```

6. Copy your system's (host machine's) /etc/resolv.conf to etc/resolv.conf

   Set proxies if necessary

   ```shell
   cp /etc/resolv.conf etc/resolv.conf
   ```

7. Download a simple bash script for devices mounting and un-mounting later

   ```shell
   cd ..
   wget https://raw.githubusercontent.com/psachin/bash_scripts/master/ch-mount.sh
   ```

8. Mount proc, sys, dev, dev/pts to the new filesystem

   ```shell
   chmod a+x ch-mount.sh
   ./ch-mount.sh -m rootfs/
   export LANG=C.UTF-8
   unset LC_NUMERIC
   unset LC_TIME
   unset LC_MEASUREMENT
   unset LC_TELEPHONE
   unset LC_IDENTIFICATION
   unset LC_PAPER
   unset LC_MONETARY
   unset LC_NAME
   unset LC_ADDRESS
   ```

9. Update the repositories

   ```shell
   apt update
   ```

10. Install minimal packages required for core utils

    ```shell
    apt install apt-utils dialog --yes
    apt install language-pack-en-base sudo ssh net-tools ethtool iputils-ping rsyslog bash-completion vim-tiny kmod --yes
    apt install isc-dhcp-client network-manager --yes
    apt clean
    cat <<EOF >/etc/NetworkManager/conf.d/allow-ethernet.conf
    [keyfile]
    unmanaged-devices=*,except:type:wifi,except:type:gsm,except:type:cdma,except:type:ethernet
    EOF
    ```

11. Install tools

    ```shell
    apt install usbutils --yes
    apt install gpiod --yes
    apt install memtester --yes
    apt install fdisk --yes
    apt clean
    ```

12. Install packages required for VNC XFCE Desktop

    ```shell
    apt install xfce4 xfce4-goodies --yes
    apt install tightvncserver --yes
    apt clean
    ```

13. Install packages required for text edit, network test

    ```shell
    apt install vim --yes
    apt install iperf3 --yes
    apt clean
    ```

14. Install packages required for WiFi USB Dongle (required 1GB)

    ```shell
    apt install --reinstall linux-firmware --yes
    apt clean
    ```

15. Install bluez packages required for Bluetooth USB Dongle

    ```shell
    apt install bluez --yes
    apt clean
    ```

16. Solve RFCOMM issue

    ```shell
    vi /lib/systemd/system/bluetooth.service # modify ExecStart=/usr/lib/bluetooth/bluetoothd --compat
    ```

17. Add a user account and include it in suitable groups

    ```shell
    useradd terasic -m -s /bin/bash
    echo terasic:123 | chpasswd
    addgroup terasic adm && addgroup terasic sudo
    addgroup terasic audio && addgroup terasic video
    ```

18. Add host entry to /etc/hosts

    ```shell
    echo "127.0.0.1    localhost" >> /etc/hosts
    ```

19. Exit chroot and unmount proc, sys, dev, dev/pts

    ```shell
    exit
    ./ch-mount.sh -u rootfs/
    ```

20. Return to user privilege

    ```shell
    exit
    ```

## 7. Build SD Card Image

```shell
cd $TOP_FOLDER
sudo rm -rf sd_card && mkdir sd_card && cd sd_card
wget https://releases.rocketboards.org/release/2020.11/gsrd/tools/make_sdimage_p3.py
# remove mkfs.fat parameter which has some issues on Ubuntu 22.04
sed -i 's/\"\-F 32\",//g' make_sdimage_p3.py
chmod +x make_sdimage_p3.py
mkdir fatfs &&  cd fatfs
cp $TOP_FOLDER/u-boot-socfpga/u-boot.itb .
cp $TOP_FOLDER/linux-socfpga/arch/arm64/boot/Image .
cp $TOP_FOLDER/linux-socfpga/arch/arm64/boot/dts/intel/socfpga_agilex5_de25_nano.dtb .
cp $TOP_FOLDER/uboot-script/boot.scr.uimg .
cd ..

mkdir rootfs && cd rootfs # renew: sudo rm -rf rootfs && mkdir rootfs && cd rootfs
sudo cp -a  $TOP_FOLDER/rootfs/* .
cd $TOP_FOLDER/linux-socfpga
sudo make -j $(nproc) modules_install INSTALL_MOD_PATH=$TOP_FOLDER/sd_card/rootfs

cd $TOP_FOLDER/sd_card
sudo python3 make_sdimage_p3.py -f \
-P fatfs/*,num=1,format=fat32,size=500M \
-P rootfs/*,num=2,format=ext3,size=5000M \
-s 5500M \
-n sdcard.img
cd ..
```

## 8. Build QSPI Image

1. Make sure Quartus Pro 25.1 is installed
2. Go to GHRD project directory (copy from DE25 Nano System CD)
3. Copy $TOP_FOLDER/u-boot-socfpga/spl/u-boot-spl-dtb.hex to software/u-boot/spl/u-boot-spl-dtb.hex
4. Execute sof_with_hps.bat to generate output_files/golden_top_hps.sof
5. Execute sof_to_jic.bat to generate output_files/golden_top_hps.jic