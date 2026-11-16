# ThinkPad L380 Yoga macOS Tahoe 26 OpenCore

Reference OpenCore EFI and troubleshooting record for the Lenovo ThinkPad L380 Yoga with Intel Core i5-8250U and Intel UHD Graphics 620. The configuration was tested on macOS Tahoe 26.7.3, build `25G342`.

This repository documents a device-specific result. It is a starting point for similar machines, not a universal installer. Firmware settings, panel revisions, USB wiring, Wi-Fi cards, audio codecs and SMBIOS values can differ between L380 Yoga units.

## Hardware and software tested

| Item | Tested value |
|---|---|
| Computer | Lenovo ThinkPad L380 Yoga |
| CPU | Intel Core i5-8250U |
| iGPU | Intel UHD Graphics 620, PCI device `0x5916` |
| Internal panel | 1920x1080 |
| macOS | Tahoe 26.7.3, build `25G342` |
| Bootloader | OpenCore 1.0.8 |
| SMBIOS class | MacBookPro16,1 |
| Wi-Fi | Intel card using itlwm and HeliPort |
| Audio | AppleALC with SimpleHDA Tahoe support |

## What was verified

- OpenCore boot picker and internal boot from the EFI partition.
- Intel UHD 620 acceleration and Metal 3 on the internal display.
- Internal keyboard through VoodooPS2Controller.
- USB keyboard and USB mouse through the corrected USB map.
- Internal audio and microphone with AppleALC and SimpleHDA.
- Intel Wi-Fi in the installed system using the existing itlwm method.
- HDMI video after applying `igfxagdc=0`.

HDMI audio, sleep/wake variations, Recovery Wi-Fi and hardware-specific touchscreen or pen behavior must be tested separately on each machine.

## The HDMI problem and the confirmed fix

The Sequoia baseline already contained the correct L380 connector mapping:

```text
AAPL,ig-platform-id = 0000c087
device-id           = 16590000
framebuffer-con1-alldata = 010509000004000087010000
framebuffer-con1-enable  = 01000000
framebuffer-con2-alldata = 02040a000008000087010000
framebuffer-con2-enable  = 01000000
```

The final mapping assigns the external HDMI connector to framebuffer `FB2`, using Bus 04. It was copied from the known-good Sequoia baseline and kept unchanged during the Tahoe troubleshooting.

On Tahoe, the failure was diagnosed from kernel and AGDC logs:

1. The HDMI cable generated Hot-Plug events, so the physical connector and HPD path were visible to macOS.
2. Tahoe assigned the connector to `FB2` and identified it as HDMI.
3. AGDC began managing the display path.
4. AUX/DPCD and EDID reads timed out, leaving the external display absent from `system_profiler`.
5. `igfxonln=1` did not repair the EDID path; it only forced an online status.
6. `-igfxhdmidivs` did not change the result because the failure was not a divider calculation failure.
7. `igfxagdc=0` disabled AGDC registration for the framebuffer, after which HDMI video appeared.

The working Tahoe boot argument is:

```text
igfxagdc=0
```

The current configuration keeps the proven framebuffer mapping and adds only this Tahoe-specific argument. Do not add random Bus IDs, EDID blobs, LSPCON patches or framebuffer overrides without a log showing that they address the actual failure.

## Boot arguments

The public template contains:

```text
-v keepsyms=1 debug=0x144 alcid=96 igfxagdc=0
```

`-v`, `keepsyms=1` and `debug=0x144` are diagnostic settings retained from the tested configuration. After a user has completed testing, they may create a private release with quieter boot arguments. `alcid=96` selects the tested internal audio layout. `igfxagdc=0` is the HDMI fix and should remain while HDMI is required.

## USB, keyboard and mouse

The USB map was corrected for Tahoe by adding the per-port metadata expected by USBMap, including `usb-port-number` and `usb-port-type`. Duplicate USB port entries were removed. The map is specific to the L380 Yoga port layout and should be replaced with a new map if a different model or motherboard revision is used.

The laptop keyboard is supplied by VoodooPS2Controller and its keyboard plugin. USB keyboards and USB mice depend on the USB map and the controller being correctly enumerated. The public configuration does not add `ps2kbdonly=1`.

Touchscreen, pen, TrackPoint and I2C touch input are not required for the basic keyboard and mouse target. Avoid enabling additional I2C input stacks just because they exist in another laptop's EFI.

## Audio

The tested Tahoe setup uses Lilu, AppleALC, SimpleHDA Tahoe support and `alcid=96`. SimpleHDA is included for the Tahoe internal audio path. HDMI audio is a separate path and should only be judged after HDMI video is detected. Do not change the working audio layout while diagnosing video output.

## Wi-Fi and Tahoe Recovery networking

The installed system uses the Intel Wi-Fi method already proven on the machine: itlwm with HeliPort. The public repository deliberately removes any Wi-Fi profile, SSID and password.

Installed-system Wi-Fi does not prove that Tahoe Recovery has Internet access. Recovery may not load the same Wi-Fi kext or support the same user-space helper. For Online Recovery, test USB Ethernet, USB tethering exposed as Ethernet, or Windows Wi-Fi-to-Ethernet Internet Sharing. Do not add an experimental Tahoe Wi-Fi kext merely because the installed system works.

## EFI layout

```text
EFI/
├── BOOT/
│   └── BOOTX64.EFI
└── OC/
    ├── ACPI/
    ├── Drivers/
    ├── Kexts/
    ├── OpenCore.efi
    └── config.plist
```

The public tree contains only the EFI template and documentation. Personal backups, debug logs, SSH data, Wi-Fi profiles and historical configs are intentionally excluded.

## First use on another L380 Yoga

1. Keep the machine's known-good EFI in a separate backup folder.
2. Mount the EFI partition only after identifying the correct disk and partition.
3. Copy this repository's `EFI` folder to a USB test device first.
4. Generate unique SMBIOS values with GenSMBIOS. Never reuse the placeholders or another person's identity.
5. Confirm the Wi-Fi method and remove or add only the kexts required by the target card.
6. Validate the configuration with `ocvalidate` from the same OpenCore release.
7. Boot from USB and test the internal display, HDMI, audio, keyboard, USB, Wi-Fi, sleep and wake.
8. Keep the original internal EFI available as the rollback option.
9. Only after successful testing should the EFI be copied to the internal EFI partition.

This workflow does not erase disks, format partitions or merge APFS volumes.

## Validation commands

Run the validator that belongs to the OpenCore release used by the EFI:

```text
ocvalidate EFI/OC/config.plist
```

Also verify that every enabled `Kernel -> Add` entry exists under `EFI/OC/Kexts`, and that `BundlePath`, `ExecutablePath` and `PlistPath` match the actual kext layout.

For graphics diagnosis, collect `system_profiler SPDisplaysDataType`, `ioreg -l -r -c AppleIntelFramebuffer` and `AGDCDiagnose -a` without exposing personal logs publicly. Look for the connector's framebuffer index, connector type, Hot-Plug events, AUX/DPCD errors and EDID results. The last visible verbose line is not necessarily the root cause.

## Rollback

If HDMI, input, audio or boot behavior regresses, boot from the known-good USB EFI, mount the internal EFI, restore the previous EFI or `config.plist`, and reboot. Do not erase the internal disk to recover from an EFI configuration error.

The HDMI test sequence used separate backups for `igfxhdmidivs`, `igfxonln=1` and `igfxagdc=0`. The working result was retained only after the display was observed in macOS.

## Known limitations

- This is not an Apple-supported configuration.
- Tahoe updates can change framebuffer and Recovery behavior.
- HDMI audio is independent from HDMI video detection.
- Recovery Wi-Fi is not guaranteed by installed-system Wi-Fi.
- L380 and L380 Yoga are similar but not identical; do not assume their ACPI or USB maps are interchangeable.
- SMBIOS placeholders must be replaced for Apple services and should never be copied between users.

## Public repository hygiene

The published template contains no real serial number, MLB, UUID, ROM, SSID, Wi-Fi password, SSH key, private log or personal path. The `itlwm` Wi-Fi profile was removed before publication. Upstream kext binaries may contain normal build metadata and source paths belonging to their original projects; these are not credentials or identifiers from this machine.

## Search terms

`ThinkPad L380 Yoga Hackintosh Tahoe`, `Lenovo L380 Yoga macOS 26`, `Intel UHD 620 HDMI EDID AUX timeout`, `igfxagdc=0`, `OpenCore Tahoe HDMI`, `i5-8250U Hackintosh`, `L380 Yoga OpenCore EFI`, `Tahoe Intel framebuffer HDMI`.

## Credits and licensing

This repository packages configuration work around upstream projects. Review and retain the license and notices for OpenCore, Lilu, WhateverGreen, VirtualSMC, AppleALC, itlwm, Intel Bluetooth kexts, VoodooPS2, VoodooI2C, VoodooSMBus, USBMap and SimpleHDA when redistributing the repository.
