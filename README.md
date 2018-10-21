## Introduction

This repo is my notes and configuration files for my hackintosh (10.14 Mojave) installation on a Thinkpad P52s. The Clover files in this repo (config.plist + ACPI/patched/ + kexts/Other/ + drivers64UEFI) _in theory_ (I didn't test extensively) should boot macOS 10.14, installer or post-install, on any P52s. It's highly likely to work on a T580 or T480, too.

For T480s, see <https://github.com/kk1987/T480s-hackintosh>, of which this repo was initially based off.

The configuration here is tailored for my P52s which has a 4k UHD display. For a different screen configuration, it might be necessary to first boot without QE/CI by injecting an invalid ig-platform-id like 0x12344321, then fix the IGPU & EDID configurations accordingly.

(Hint: refer also to the T480s repo, which is for a FHD touch screen, and might also work with FHD and 1440p non-touch screens. Differences include which ig-platform-id/device-id to inject in config.plist/Devices/Properties, and whether EDID injection is required.)

## Basic Info

### Hardware:

* OEM
  * i7 8650U CPU w/ Intel UHD 620
  * Nvidia Quadro P500 dGPU
  * 4k screen (BOE NV156QUM-N44, matte non-touch)
  * Realtek ALC3287 ("ALC257")
  * Intel Ethernet I219-LM
  * Synaptics TrackPoint & TrackPad
  * Synaptics Fingerprint Reader + IR Camera
* Upgraded/replaced
  * 32GB RAM (16G DDR4-2400 x2)
  * ADATA SX8200 960GB NVMe SSD
  * BCM94352Z M.2 WiFi + BT

### UEFI Settings:

* UEFI Firmware version 1.16
* "Thunderbolt BIOS Assist": disable (default) to make front type-C port work in macOS; enable to reduce idle power consumption in Linux

### Partitioning & other OSes:

I did a tri-boot setup with installation order as follows:

* macOS -> Windows 10 -> Arch Linux (Antergos)

## Status

### Working
* most features do work
  * Keyboard & TrackPoint/TrackPad (as mouse)
  * Ethernet, WiFi, Bluetooth
  * Screen brightness & brightness shortcut keys
  * Basic audio including speaker, internal mic, headphone jack
  * Camera
  * Card reader
  * All USB ports
  * HDMI video port, with audio
  * USB type-C video output, with audio
  * Thunderbolt (as long as you didn't enable "Thunderbolt BIOS Assist")

### Limited functionality

* Audio
  * (I don't see why it's ever needed but) there's no external mic support
  * "out of the box" headphone sound may be glitchy. Disabling "Use ambient noise reduction" in Settings -> Sound -> Input seems to fix this
  * Alternatively, VoodooHDA can be used. It supports enternal mic via combo jack (without auto-switching) but doesn't support HDMI audio. Configuration in `Info.plist` need to be updated with `iGain=0, PCM=100, Rec=50`
* HDMI/DP output
  * TODO

### Not Working / Untested

* Sleep/wake
* NVIDIA P500 GPU (no way to utilize, disabled via SSDT)
* IR Camera (no way to use in macOS / breaks FaceTime; disabled)
* Fingerprint reader (no driver / no way to use)

## Clover UEFI Setup

### Clover

* Verified working with sf.net versions r4674, r4700

### drivers64UEFI

* ApfsDriverLoader-64
* NvmExpressDxe-64
* AptioMemoryFix-64 (or OsxAptioFixDrv-64 + EmuVariableUefi-64)

### config.plist

* (archived in this repo)

### kexts

I put all extra kexts under EFI/CLOVER/kexts/Other.

* FakeSMC (w/ all plugins), for Hackintosh to boot
* ACPIBatteryManager, for battery status
* AppleBacklightFixup, for brightness
* AppleALC + CodecCommander, for audio
* Lilu, for various stuff below to work
* WhateverGreen, for iGPU
  * See Devices/Properties patches in config.plist (set ig-platform-id & device-id + patch for 32MB DVMT-prealloc)
* USBInjectAll, for USB ports
  * Touchscreen, Bluetooth etc. won't work without this
  * See SSDT-UIAC.dsl for machine-specific patch
* VoodooPS2Controller, for keyboard/touchpad/trackpad
* IntelMausiEthernet, for Ethernet
* AirportBrcmFixup, for WiFi; BrcmFirmwareData + BrcmPatchRAM2, for Bluetooth
  * Refer to toledo's [guide](https://www.tonymacx86.com/threads/broadcom-wifi-bluetooth-guide.242423/)

## ACPI Patching

* Based on RehabMan's [guide](https://www.tonymacx86.com/threads/guide-patching-laptop-dsdt-ssdts.152573/) and linusyang92's [T480s repo](https://github.com/linusyang92/macOS-ThinkPad-T480s)
