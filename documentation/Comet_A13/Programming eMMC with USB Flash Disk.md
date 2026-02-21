## Programming eMMC with USB Flash Disk

1.  Split the eMMC image

    ```shell
    unzip comet_a13_revA_emmc_console_v1.0.zip
    export source_img=comet_a13_revA_emmc_console_v1.0.img
    split -b 1G -d ${source_img} ${source_img}.part
    ```

2.  Copy the split image files to a USB flash Disk, then connect it to the board's USB port.

3.  Insert the microSD card (with the boot image) into the board.

4.  Set SW3 to OFF (low).

5.  Open Putty, set baud rate to 115200, and select the corresponding COM port for the board.

6.  Power on the board. When the countdown appears in the Putty terminal, press Enter to enter the U‑Boot command line.

7.  Execute the following commands to program the eMMC:

    ```shell
    usb start # scanning usb for storage devices... 1 Storage Device(s) found

    fatls usb 0:1 # 0:1 indicates the first partition of the first USB device. It will list the found filenames, e.g., comet_a13_revA_emmc_console_v1.0.img.part00
    # If the image is not in partition 1, check subsequent partitions incrementally. Once located, this step can be skipped in the future.

    # Set SW1.2 to OFF (boot from eMMC)
    mmc rescan

    env set image_file comet_a13_revA_emmc_console_v1.0.img; # Replace the image name if a newer version is used
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

