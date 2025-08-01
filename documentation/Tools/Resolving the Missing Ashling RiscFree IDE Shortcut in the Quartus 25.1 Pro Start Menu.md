## Problem Description



After installing Quartus 25.1 Pro edition and Ashling RiscFree IDE for Intel FPGAs 25.1, the shortcut for the `Ashling RiscFree IDE` does not appear in the Quartus 25.1 Pro directory within the Windows Start Menu.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_01.png" height="500">

## Solution



You can manually create the shortcut by following these steps:

1. Download the provided shortcut file to your host computer. Download from the [Link](https://terasic-my.sharepoint.com/:u:/p/johnny/Ea6LlbEDMTNOmTrFbVQRDMYB1s4SiScnulS4uHU2GqPtGA?e=HD9crE)) . 

   After downloading the file, please change the file extension from
    `Ashling RiscFree IDE for Intel FPGAs 25.1.download`
    to
    `Ashling RiscFree IDE for Intel FPGAs 25.1.lnk`.

2. Right-click on the downloaded "Ashling RiscFree IDE for Intel FPGAs 25.1" shortcut file and select `Properties` from the menu.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_02.png" height="400">

3. Modify the shortcut properties:

   - **Modify the `Target` field**: Replace the Quartus installation path in the field with the actual path on your computer.

     - For example, change:

       `E:\altera_pro\25.1\niosv\bin\niosv-shell.exe --run E:/altera_pro/25.1/riscfree/RiscFree/RiscFree.exe` 

       To your specific path, such as:

       `D:\altera_pro\25.1\niosv\bin\niosv-shell.exe --run D:/altera_pro/25.1/riscfree/RiscFree/RiscFree.exe` 

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_03.png" height="400">

   - **Modify the `Start in:` field**: Change the original content from `C:\Users\User\AppData\Local\quartus` to your Quartus installation path, such as `D:\altera_pro\25.1\`.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_04.png" height="400">

4. After making the changes, click "**OK**" to save.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_05.png" height="400">

5. Locate the Start Menu shortcut directory:

   - In the Windows Start Menu, find the `Altera 25.1.0.129 Pro Edition`directory.

   - Right-click on any shortcut within the directory (such as Quartus), then select 

     **More** > **Open file location**.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_06.png" height="400">

6. Copy the `Ashling RiscFree IDE for Intel FPGAs 25.1` shortcut that you just modified and paste it into this directory.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_07.png" height="300">

7. Once complete, reopen the "Altera 25.1.0.129 Pro Edition" directory in the Start Menu. The `Ashling RiscFree IDE for Intel FPGAs 25.1` shortcut will now be visible. Click it to launch the application.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/tools/ashling_riscFree_q251_issue_08.png" height="400">
