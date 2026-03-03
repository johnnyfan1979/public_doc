## Programming eMMC with USB Flash Disk

1.  Before programming eMMC, make sure the QSPI Flash has been programmed

2.  Split the eMMC image

    ```shell
    unzip titan_s10_som_revB_emmc_yocto_console_v1.1.zip
    export source_img=titan_s10_som_revB_emmc_yocto_console_v1.1.img
    split -b 1G -d ${source_img} ${source_img}.part
    ```

3.  Copy the split image files to a USB flash Disk, then connect it to the Carrier board's USB OTG port.

4.  Using a USB-C cable to connect the Carrier Board's UB3 Type-C port with PC.

5.  Open Putty, set baud rate to 115200, and select the corresponding COM port for the board.

6.  Power on the board. When the countdown appears in the Putty terminal, press Enter to enter the U‑Boot command line.

7.  Execute the following commands to program the eMMC:

    ```shell
    usb start # scanning usb for storage devices... 1 Storage Device(s) found

    fatls usb 0:1 # 0:1 indicates the first partition of the first USB device. It will list the found filenames, e.g.,titan_s10_som_revB_emmc_yocto_console_v1.1.img.part00
    # If the image is not in partition 1, check subsequent partitions incrementally. Once located, this step can be skipped in the future.

    env set image_file titan_s10_som_revB_emmc_yocto_console_v1.1.img; # Replace the image name if a newer version is used
    # Start loading files
    fatload usb 0:1 ${loadaddr} ${image_file}.part00; # Assuming the image is located in partition 1
    setexpr blkcnt1 ${filesize} / 0x200;
    setexpr cnt_ofs 0;
    mmc write $loadaddr $cnt_ofs $blkcnt1;

    fatload usb 0:1 ${loadaddr} ${image_file}.part01;
    setexpr blkcnt2 ${filesize} / 0x200;
    setexpr cnt_ofs ${cnt_ofs} + ${blkcnt1};
    mmc write $loadaddr $cnt_ofs $blkcnt2;

    fatload usb 0:1 ${loadaddr} ${image_file}.part02;
    setexpr blkcnt3 ${filesize} / 0x200;
    setexpr cnt_ofs ${cnt_ofs} + ${blkcnt2};
    mmc write $loadaddr $cnt_ofs $blkcnt3;

    fatload usb 0:1 ${loadaddr} ${image_file}.part03;
    setexpr blkcnt4 ${filesize} / 0x200;
    setexpr cnt_ofs ${cnt_ofs} + ${blkcnt3};
    mmc write $loadaddr $cnt_ofs $blkcnt4;

    fatload usb 0:1 ${loadaddr} ${image_file}.part04;
    setexpr blkcnt5 ${filesize} / 0x200;
    setexpr cnt_ofs ${cnt_ofs} + ${blkcnt4};
    mmc write $loadaddr $cnt_ofs $blkcnt5;

    fatload usb 0:1 ${loadaddr} ${image_file}.part05;
    setexpr blkcnt6 ${filesize} / 0x200;
    setexpr cnt_ofs ${cnt_ofs} + ${blkcnt5};
    mmc write $loadaddr $cnt_ofs $blkcnt6;
    ```

