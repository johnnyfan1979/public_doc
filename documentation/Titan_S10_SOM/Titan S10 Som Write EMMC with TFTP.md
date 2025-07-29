# Detailed Guide: Flashing the Terasic S10 SOM eMMC

This guide explains how to flash a Linux OS image (`.img`) onto the eMMC (embedded MultiMediaCard) storage of the Terasic S10 System on Module (SOM). The entire process is broken down into four main stages:

1.  **Preparation**: Prepare the image file and TFTP server on your Host PC.
2.  **Hardware Connection & Initial Setup**: Connect the board and flash a temporary bootloader to the OSPI Flash.
3.  **Execution**: Use a serial terminal (UART) to control the on-board U-Boot bootloader, download the image from the TFTP server, and write it to the eMMC.
4.  **Verification**: Reboot the board to confirm that the system boots correctly from the eMMC.

---

## Stage 1: Preparation (On the Host PC)

The goal of this stage is to split a large image file into several smaller chunks and place them in the TFTP server's directory for transfer.

#### Why split the image file?
The development board's RAM (Random Access Memory) is limited and may not be able to load the entire multi-gigabyte image file at once. By splitting it, we can load the image piece by piece into RAM and then write each piece to the eMMC, ensuring a stable flashing process.

#### Steps:

1.  **Unzip the File**:
    First, unzip the downloaded [titan_s10_som_revA_emmc_yocto_console_v1.0.zip](https://terasic-my.sharepoint.com/:u:/p/johnny/EdzfybN0xy1EjiliasVPFwUBT20EHGY28cHiagdOJf4RXA?e=ub1S0z) file.
    
    ```bash
    unzip titan_s10_som_revA_emmc_yocto_console_v1.0.zip
    ```
This will extract the image file: `titan_s10_som_revA_emmc_yocto_console_v1.0.img`.
    
2. **Split the Image File**:
   Use the `dd` command to split the `.img` file into 5 parts. For convenience, you can set an environment variable first.
   ```bash
   # Set an environment variable for the image path
   export source_img=titan_s10_som_revA_emmc_yocto_console_v1.0.img
   
   # Execute the split command (this creates 5 files: part1 to part5)
   dd if=$source_img of=$source_img.part1 bs=512 skip=0 count=1000000
   dd if=$source_img of=$source_img.part2 bs=512 skip=1000000 count=1000000
   dd if=$source_img of=$source_img.part3 bs=512 skip=2000000 count=1000000
   dd if=$source_img of=$source_img.part4 bs=512 skip=3000000 count=1000000
   dd if=$source_img of=$source_img.part5 bs=512 skip=4000000 count=96000
   ```

3.  **Set up the TFTP Server**:
    Copy the 5 split files into your TFTP server's root directory (commonly `/srv/tftp/` or `/tftpboot/` on Linux).
    
    ```bash
    sudo cp titan_s10_som_revA_emmc_yocto_console_v1.0.img.part* /srv/tftp/
    ```
    **Important**: Ensure your TFTP server is running and that your PC's firewall allows TFTP connections (usually on UDP port 69).

---

## Stage 2: Hardware Connection & Initial Setup

This stage prepares the board to enter a "programming mode" where it can receive commands.

1.  **Hardware Connections**:
    - Use a **USB-C cable** to connect your PC to the **J10** port on the carrier board (for serial communication).
    - Use an **RJ45 Ethernet cable** to connect the **J11** port on the carrier board to your local network (ensure it's on the same network as your Host PC).

2.  **Power On**:
    Connect the power supply to the development board.

3.  **Flash the Initial Bootloader (OSPI Flash)**:
    Execute the `flash_program_hps.bat` script. This batch file flashes a small "programming bootloader" into the board's OSPI Flash. This program is essential for the subsequent eMMC flashing steps.

---

## Stage 3: Flashing the eMMC

Now, you will use a serial terminal (like PuTTY) to issue commands that instruct the board to download the files from your TFTP server and write them to the eMMC.

1.  **Open a Serial Terminal**:
    - Open **PuTTY** or another serial terminal application on your PC.
    - Select the connection type **Serial**.
    - Configure the correct COM Port (you can find this in your Device Manager) and Baud Rate (check your board's documentation, typically 115200).

2.  **Reboot the Board into Programming Mode**:
    - **Power cycle** (turn off and on) the development board.
    - You should now see boot messages appearing in the PuTTY window, confirming the serial connection is working. The board will boot from the OSPI Flash you just programmed and stop at the U-Boot command prompt.

3. **Execute the Flashing Commands**:
   At the U-Boot prompt in PuTTY, **sequentially** enter all the following commands. It is highly recommended to copy and paste them line by line to avoid typos.

   ```bash
   # --- 1. Environment Setup ---
   # Set the base image filename
   env set image_file titan_s10_som_revA_emmc_yocto_console_v1.0.img
   # Disable autoload and get an IP address via DHCP
   setenv autoload no; dhcp
   # !!! CRITICAL: Set the IP address of your Host PC (the TFTP server) !!!
   setenv serverip 192.168.1.222
   
   # --- 2. Initialize Write Offset ---
   setexpr cnt_ofs 0
   
   # --- 3. Flash Part 1 ---
   tftp ${loadaddr} ${image_file}.part1
   setexpr blkcnt1 ${filesize} / 0x200
   mmc write ${loadaddr} ${cnt_ofs} ${blkcnt1}
   setexpr cnt_ofs ${cnt_ofs} + ${blkcnt1}
   
   # --- 4. Flash Part 2 ---
   tftp ${loadaddr} ${image_file}.part2
   setexpr blkcnt2 ${filesize} / 0x200
   mmc write ${loadaddr} ${cnt_ofs} ${blkcnt2}
   setexpr cnt_ofs ${cnt_ofs} + ${blkcnt2}
   
   # --- 5. Flash Part 3 ---
   tftp ${loadaddr} ${image_file}.part3
   setexpr blkcnt3 ${filesize} / 0x200
   mmc write ${loadaddr} ${cnt_ofs} ${blkcnt3}
   setexpr cnt_ofs ${cnt_ofs} + ${blkcnt3}
   
   # --- 6. Flash Part 4 ---
   tftp ${loadaddr} ${image_file}.part4
   setexpr blkcnt4 ${filesize} / 0x200
   mmc write ${loadaddr} ${cnt_ofs} ${blkcnt4}
   setexpr cnt_ofs ${cnt_ofs} + ${blkcnt4}
   
   # --- 7. Flash Part 5 ---
   tftp ${loadaddr} ${image_file}.part5
   setexpr blkcnt5 ${filesize} / 0x200
   mmc write ${loadaddr} ${cnt_ofs} ${blkcnt5}
   
   ```

   

---

## Stage 4: Verification

1.  **Reboot**:
    After all commands have been executed successfully, **power cycle the development board one more time**.

2.  **Verify the Boot Process**:
    In the PuTTY window, you should see the Linux system's boot messages, ending with a `login:` prompt. This indicates the board has successfully booted from the new OS on the eMMC.

3.  **Log in to the System**:
    - Enter the username: `root`
    - Press Enter to log in.

The eMMC flashing process is now complete.

````

````
