# Release notes

## 1.0.0 Tahoe reference

- macOS Tahoe 26.7.3 / build 25G342 tested on ThinkPad L380 Yoga.
- OpenCore 1.0.8 config validated with matching `ocvalidate`.
- HDMI detection fixed by retaining the L380 framebuffer connector mapping and adding `igfxagdc=0`.
- USB map retained with Tahoe port metadata.
- SimpleHDA retained for the Tahoe internal audio path.
- SMBIOS and personal NVRAM values replaced with public placeholders.

This release is a reference starting point. Hardware-specific validation is still required.
