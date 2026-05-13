# Stratix 10 USB Device Recognition Issue

## Symptom

On the Stratix 10 platform, if a USB device is already plugged in before the system is powered on, the HPS may intermittently fail to recognize the device.

## Technical Analysis

Investigation shows that this issue originates from a hardware limitation in the Stratix 10 internal USB controller (DWC2 IP) when handling the **Clock-gating** power-saving mechanism. During system initialization, when the driver attempts to toggle the clock state to reduce power consumption, the hardware is unable to respond correctly. This causes the USB controller's state machine to hang or fail to complete initialization, ultimately resulting in device recognition failure.

## Solution

By updating the Device Tree, the driver is explicitly instructed to work around this hardware defect (by reusing the fix logic already applied on the Agilex platform, which disables the unsupported clock-gating functionality). This ensures stable USB device recognition under all power-on conditions.

### Implementation Details

Modify `arch/arm64/boot/dts/altera/socfpga_stratix10.dtsi` to update the `compatible` property of both USB nodes (`usb0` and `usb1`), adding the Intel SoCFPGA-specific compatible string. This allows the DWC2 driver to correctly apply the platform-specific settings and bypass the clock-gating behavior.

Example modification:

```
usb0: usb@ffb00000 {
    compatible = "intel,socfpga-agilex-hsotg", "snps,dwc2";
    ...
};

usb1: usb@ffb40000 {
    compatible = "intel,socfpga-agilex-hsotg", "snps,dwc2";
    ...
};
```

## Reference

Original discussion thread:
[\[PATCH\] usb: dwc2: add new compatible for Intel SoCFPGA Stratix10 platform](https://lists.yoctoproject.org/g/linux-yocto/topic/patch_usb_dwc2_add_new/100210250?dir=desc)
