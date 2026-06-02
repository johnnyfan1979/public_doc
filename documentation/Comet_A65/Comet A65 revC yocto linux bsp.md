# Comet A65 Yocto BSP Support

## 1. Overview

Terasic provides Yocto Project BSP support for **Comet A65 RevC**, addressing customer requirements for embedded system customization, footprint reduction, and production maintenance.

**Test Status**: Passed on Comet A65 RevC board 

## 2. Pre-built Image Download

| Item | Location |
| --- | --- |
| Linux Image | `sftp://terasic_china@ftp.terasic.com.tw/uploads/Image/Comet-A65/revC/yocto/` |
| QSPI Flash File (JIC) | RevC CD |

> Ready-to-use, no rebuild required.

## 3. Source Build (if modification is needed)

**GitHub Repository**:  
https://github.com/terasic/gsrd-socfpga/tree/comet-a65-revC-25.3.1

**Build Instructions**: Follow **"Default GSRD Setup"** steps in the README  
**Boot Mode**: FPGA boot first