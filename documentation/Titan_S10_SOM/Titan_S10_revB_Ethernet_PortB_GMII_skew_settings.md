To adjust the Ethernet Port B's GMII gmac skew settings for rev.B Titan S10 SoM, the following modifications need to be made in the original **socfpga_stratix10_titan_s10_som.dts**:

1. Remove the previous SGMII information and delete the following node information from the file.

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/titan_s10/image-20250915142950610.png" height="500">

2. Add GMII information. Modify **&gmac1** to GMII mode. 

   Before modification:

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/titan_s10/image-20250915143056939.png " height="500">

   Details of the modified and usable gmac1 node:

   ```c
   &gmac1 {
       status = "okay";
       phy-mode = "gmii";
       phy-handle = <&phy1>;
       max-frame-size = <9000>;
       
       mdio1 {
           #address-cells = <1>;
           #size-cells = <0>;
           compatible = "snps,dwmac-mdio";
           phy1: ethernet-phy@1 {
               reg = <2>;
               
               txen-skew-ps = <0>; /* -420ps */
               txc-skew-ps = <480>; /* 0ps */
               rxdv-skew-ps = <420>; /* 0ps */
               rxc-skew-ps = <1680>; /* 780ps */
           };
       };
   };
   ```

   Reference table for **txc-skew-ps** and **rxc-skew-ps**:

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/titan_s10/image-20250915143232318.png " height="500">

   Corresponding register information:

<img src="https://raw.githubusercontent.com/johnnyfan1979/public_doc/main/documentation/images/titan_s10/image-20250915143314817.png" height="500">

   Packet loss rate test:

   ```c
   # soc 
   iperf3 -s
   
   # Linux PC 
   # soc TX -> PC RX
   iperf3 -c <soc ipaddress> -u -b 1G -R
   # soc RX <- PC TX
   iperf3 -c <soc ipaddress> -u -b 1G
   ```

   If there are packet loss in the soc TX, adjust the **txc-skew-ps** in the range of 0 to 1860 with an interval of 60. 


   If there are packet loss in the soc RX, adjust the **rxc-skew-ps** in the range of 0 to 1860 with an interval of 60.
