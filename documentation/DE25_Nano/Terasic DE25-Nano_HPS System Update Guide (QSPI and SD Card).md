# Terasic DE25-Nano: HPS System Update Guide (QSPI & SD Card)

## Special Note for Cyclone V Users: Agilex 5 Architecture Differences

* **Boot Architecture:** The boot architecture of Agilex 5 differs significantly from Cyclone V. It utilizes the **SDM (Secure Device Manager)** to manage the boot process.
* **Key Difference:** The **SOF** file for the FPGA must include the **HPS First Stage Bootloader (FSBL/SPL)** information to be correctly converted into a **JIC** file and programmed into QSPI.
* **Operational Recommendation:** Even if you are simply updating the **FPGA logic**, you must execute the process to merge it with the HPS (`sof_with_hps`). Otherwise, the HPS may fail to boot after programming.

---

## Part 1: Updating QSPI Flash (Windows Environment)

### Purpose
To update the FPGA hardware logic (SOF) and the initial boot program (Bootloader/SPL).

### Prerequisites
Please ensure **Quartus Prime Pro** is installed.

### Procedure

#### 1. Compile GHRD Hardware Project
Compile the project in Quartus to ensure the latest `.sof` file is generated.

#### 2. Prepare Bootloader File (SPL)
* **File Location:** `GHRD/software/u-boot/spl/u-boot-spl-dtb.hex`
* **Description:** This is the First Stage Bootloader. If you have **not** modified the U-Boot source code, you do not need to update this file (use the existing file directly). If you have modified U-Boot, please overwrite this path with the newly compiled hex file.

#### 3. Generate Programming File (.jic)
Please execute the following scripts in order (located under `GHRD/`):

* **a. Execute `sof_with_hps.bat`**
    * **Function:** Merges the FPGA SOF file with the HPS FSBL (u-boot-spl). **This is a mandatory step for Agilex 5**.
* **b. Execute `sof_to_jic.bat`**
    * **Function:** Converts the merged file into the JIC format suitable for QSPI Flash.

#### 4. Execute Programming
1.  Connect the USB Blaster to the PC and the board.
2.  Execute the script: `GHRD/output_files/program_qspi_flash/flash_program.bat`.
3.  **Tip:** After programming is complete, it is recommended to completely power off the board and turn it back on (**Power Cycle**).

---

## Part 2: Updating SD Card (Linux Environment)

### Purpose
To update U-Boot (Second Stage), Linux Kernel, Device Tree, and Driver Modules.

### Prerequisites
Insert the SD Card into a Linux PC and check the partitions (usually `p1` is the FAT partition, and `p2` is the Rootfs partition).

### Procedure

#### 1. Update Boot Image Files (FAT Partition)
Copy the following files to the FAT32 partition of the SD Card:

* **`u-boot.itb`** (Located under `u-boot-socfpga/`)
    * **Note:** Agilex 5 uses the `.itb` (FIT Image) format, which contains both **U-Boot proper** and **ATF (Arm Trusted Firmware)**.
* **`Image`** (Located under `linux-socfpga/arch/arm64/boot/`)
    * **Note:** Please use the **Uncompressed Image**.
* **`socfpga_agilex5_de25_nano.dtb`** (Located under `linux-socfpga/arch/arm64/boot/dts/intel/`)

#### 2. Update Kernel Modules (Rootfs Partition)
If you have modified Kernel functions or the Config, you must update the Modules inside Rootfs.

**Command Reference (Bash):**

```bash
# a. Mount Rootfs partition (Assuming it is sdx2)
sudo mount /dev/sdx2 /mnt/

# b. Install Modules (Please execute in the Linux Source Code directory)
# Note: Ensure ARCH=arm64 and CROSS_COMPILE environment variables are set
sudo make modules_install INSTALL_MOD_PATH=/mnt

# c. Unmount partition (Important! Be sure to execute sync to ensure writing)
sudo umount /mnt/
sync
```



## Appendix: Cyclone V vs Agilex 5 Differences Summary

| **Item**                  | **Cyclone V**                     | **Agilex 5**                                        |
| ------------------------- | --------------------------------- | --------------------------------------------------- |
| **Boot Flow**             | BootROM -> Preloader -> U-Boot    | SDM -> SPL (FSBL) -> U-Boot (SSBL)                  |
| **FPGA Programming File** | Directly convert `.sof` to `.jic` | Must merge SPL into `.sof` via `sof_with_hps` first |
| **U-Boot Format**         | `u-boot.img`                      | `u-boot.itb` (FIT Image, includes ATF)              |
| **Kernel Image**          | Usually uses `zImage`             | Usually uses `Image` (Uncompressed)                 |