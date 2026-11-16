# ThinkPad L380 Yoga macOS Tahoe Hackintosh

OpenCore reference EFI and troubleshooting notes for the Lenovo ThinkPad L380 Yoga with Intel Core i5-8250U and Intel UHD Graphics 620 running macOS Tahoe 26.

## Tested hardware and software

- Lenovo ThinkPad L380 Yoga
- Intel Core i5-8250U
- Intel UHD Graphics 620, device `0x5916`
- Internal display: 1920x1080
- macOS Tahoe 26.7.3, build `25G342`
- OpenCore 1.0.8
- SMBIOS class: MacBookPro16,1

This repository is a reference template. It is not a universal EFI. Generate your own SMBIOS values and verify ACPI, USB, Wi-Fi, audio and input devices before using it.

## HDMI fix on Tahoe

The old Sequoia EFI used this framebuffer mapping:

```text
AAPL,ig-platform-id = 0000c087
device-id           = 16590000
con1                = 010509000004000087010000
con2                = 02040a000008000087010000
con1-enable         = 01000000
con2-enable         = 01000000
```

That mapping remains in this EFI. On Tahoe, the important additional fix is:

```text
igfxagdc=0
```

The observed failure was not missing Hot-Plug detection. Tahoe detected the HDMI port on `FB2`, but AGDC management was followed by AUX/EDID read timeouts. Disabling AGDC allowed the Intel framebuffer to complete HDMI detection.

Do not combine this with random framebuffer patches. Keep the connector mapping above and test one graphics change at a time.

## USB and input

The USB map was corrected for Tahoe by adding the port schema expected by USBMap, including `usb-port-number` and `usb-port-type`. The EFI keeps the laptop keyboard through VoodooPS2Controller and supports ordinary USB keyboards and mice through the USB map. Touchscreen, pen and I2C input are not required for the basic setup.

## Audio

Tahoe audio uses AppleALC with SimpleHDA for the internal codec. The existing `alcid=96` was retained. HDMI audio depends on display detection and should be tested after HDMI video works.

## Wi-Fi

Intel Wi-Fi uses itlwm with HeliPort. Recovery networking is a separate problem: a kext that works in the installed system is not guaranteed to load in Tahoe Recovery. USB Ethernet or another supported wired connection is the reliable recovery path.

## Installation and safety

1. Keep a known-good EFI backup.
2. Mount the EFI partition and copy this EFI to a separate test location first.
3. Generate personal SMBIOS values with GenSMBIOS.
4. Validate `config.plist` with `ocvalidate` from the same OpenCore release.
5. Boot from USB and test before changing the internal EFI.
6. Keep the original EFI available as a rollback option.

Never erase the internal disk or merge partitions as part of an EFI change.

## Public repository hygiene

The published template must not contain real serial numbers, MLB, UUID, ROM, Wi-Fi passwords, private logs, SSH details or personal paths. Replace the SMBIOS placeholders before using iMessage or other Apple services.

## Search terms

`ThinkPad L380 Yoga Hackintosh Tahoe`, `Lenovo L380 Yoga macOS 26`, `Intel UHD 620 HDMI EDID AUX timeout`, `igfxagdc=0`, `OpenCore Tahoe HDMI`, `i5-8250U Hackintosh`.

## Credits

- OpenCore and Acidanthera kext projects
- WhateverGreen documentation and source
- USBMap project
- SimpleHDA Tahoe project

Check each upstream project's license and retain its notices when publishing the repository.
