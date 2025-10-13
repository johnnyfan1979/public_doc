# **Change List from DE25-Standard Rev. B to Rev. C**

1.  **FPGA** changed from **A5ED065BB32AE4SR0 (Rev. B)** to **A5ED013BB32AE4SR1 (Rev. C)**.
2.  **USB Blaster II** (Rev. B) updated to **USB Blaster III** (Rev. C).
3.  **Remove System MAX 10**:
    i. Remove MAX reset button.
    ii. Modify power sequencing circuit.
    iii. Add JP6 for JTAG chain configuration.
    iv. Modify HPS cold reset circuit
4. Modify clock signal names. see Table 1


**Table 1**

| Rev.B | Rev.C |
| :--- | :--- |
| clock_50 | clock0_50 |
| clock2_50 | clock1_50 |
| clock3_50 | clock3_50 |

5.  **ADC** changed from **LTC2308CUF (Rev. B)** to **TI TLA2518 (Rev. C)**, along with corresponding signal name changes (pin assignment remains the same.).


**Table 2**

| Rev.B | Rev.C |
| :--- | :--- |
| adc_sclk | adc_sck |
| adc_dout | adc_sdo |
| adc_din | adc_sdi |
| adc_convst | adc_cs_n |


6.  **QSPI Flash** changed from **512Mb (MT25QU512ABB8E12, Rev. B)** to **128Mb (MT25QU128ABA8E12, Rev. C)**.
7.  **MIPI**: Change **CAM_I2C_SCL/SDA** connection to **FPGA_I2C_SCL/SDA**.



<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/de25_standard/de25_standard_01.png " height="700">

8.  **FPGA\_I2C\_SCL/SDA** pin assignments changed from **(PIN\_BR112 / PIN\_BM109)** to **(PIN\_BF120 / PIN\_BH118)**. See Table 3.
9.  Added **HSMC I2C bus** at rev C **(PIN\_BR112 / PIN\_BM109)**.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/de25_standard/de25_standard_02.png " height="700">

10. LED/SW pin assignments have changed; refer to Table 3 for details.

**Table 3**

| Signal | Rev.B | Rev.C |
| :--- | :--- | :--- |
| hsmc_i2c_scl | Not Assigned | PIN_BR112 |
| hsmc_i2c_sda | Not Assigned | PIN_BM109 |
| cam_i2c_scl | PIN_BF120 | Not Assigned |
| cam_i2c_sda | PIN_BH118 | Not Assigned |
| fpga_i2c_sclk | PIN_BR112 | PIN_BF120 |
| fpga_i2c_sdat | PIN_BM109 | PIN_BH118 |
| sw[7] | PIN_BR62 | PIN_CF59 |
| ledr[1] | PIN_CA71 | PIN_BH78 |
| ledr[7] | PIN_CH62 | PIN_BM69 |
| ledr[8] | PIN_CF59 | PIN_CA71 |
| ledr[9] | PIN_BH78 | PIN_BR62 |
| hex5[6] | PIN_BM69 | PIN_CH62 |