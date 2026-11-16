# Validation record

## Scope

This record describes the public template prepared from a tested L380 Yoga setup. Personal SMBIOS and NVRAM values were removed before publication preparation.

## Known tested results

- Internal display: working at 1920x1080
- Intel UHD 620 Metal acceleration: working
- HDMI video on Tahoe: working with `igfxagdc=0`
- Internal audio: working with AppleALC and SimpleHDA
- USB keyboard and mouse: working with the corrected USB map
- Laptop keyboard: working through VoodooPS2Controller
- Sequoia baseline EFI: preserved separately

## Validation required by each user

- Generate unique SMBIOS values.
- Run matching `ocvalidate`.
- Confirm Wi-Fi method and Recovery network separately.
- Test sleep/wake, HDMI hot-plug and HDMI audio.
- Keep a rollback EFI before replacing an internal EFI.
