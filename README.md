# OpenCore-Legion-5i-17IMH05H
 
This repository contains EFI files for OpenCore 0.6.7 for Legion-5i-17IMH05H with macOS Big Sur 11.2.3

This is the main guide I follow: [OpenCore Dortania](https://dortania.github.io/OpenCore-Install-Guide/)

Note: The commit descriptions are inaccurate since all the files are uploaded at the same time.

Note: This repo contains extra properties files added by Mac. (.DS_STORE, .\__filename_ etc)

## Specifications

| | |
|-|-|
|CPU| Intel i7-10750H (10th gen Comet Lake/Final apple-supported gen :() |
|GPU| Intel UHD Graphics 630 iGPU + Nvidia GeForce RTX 2060 dGPU |
|Display| 17.3" FHD 1920x1080 |
|Wifi| Intel Wireless AX201 |
|Audio Codec| Realtek ALC257 |
|Memory| 16GB + 16GB |
|Storage| 1TB SSD + 2TB M.2 NVMe SSD |

## Status

| Features | Status |
|----------|--------|
| Battery  | Working |
| Audio | Working |
| Camera | Working |
| On Screen Brightness Control | Working |
| Sound | Working |
| USB ports | Working |
| Mouse | Working |
| WiFi/Bluetooth | Mostly Working |
| Keyboard | Mostly working |
| Touchpad | Not working |

## Troubleshooting
Most things work fine after following the guide so I will only list difficulties or irregularities I faced that are not found in Dortania guide.

### Backlight off (black screen) for few minutes after boot
The screen suddenly turns black at Apple Logo loading screen. Thanks to a commenter, I found out that the black screen is caused by backlight not working. You can shine a flashlight at the screen and you will see the computer is still on. It is fixed by adding -igfxblr to boot-args in config.plist. Make sure you have WhateverGreen kext and SSDT-PNLFCFL.aml in ACPI. This is likely caused by Intel 8th gen Whiskey Lake Processor.

### Cannot read kext error during installation of Big Sur / Duplicating Kexts
When I used my first successfully booted Catalina EFI in installing Big Sur, it showed cannot read kext from .. and then stucked in MACH reboot. I quickly found out that this is caused by duplicating VoodooInput kexts because both VoodooI2c and VoodooPS2Controller kexts have a copy of it as a plugin. I used ProperTree clean snapshot and snapshot functions multiple times to record the kexts in config.plist. However ProperTree somehow enabled both kexts even though it has a feature to prevent it. It is fixed by disabling the second VoodooInput in config.plist.

### WiFi not working if you boot into Mac after shutting down from Windows
I use [itlwm](https://github.com/OpenIntelWireless/itlwm) for supporting Intel Wifi Card in Mac. However, it has a [bug](https://openintelwireless.github.io/itlwm/FAQ.html#dual-boot-with-windows) when it is used to dual boot Windows. Windows has a fast startup feature that breaks the Wifi in Mac. If you boot into Mac after shutting down from Windows. You will get the error: iltwm is not running. If you boot into Mac again, this error is gone. Due to certain issue with my Windows, I cannot deactivate fast startup to fix this(as stated in FAQ). However I found out something interesting about fast startup. It is not activated when Windows is restarted so I can restart from Windows to avoid booting into Mac twice to fix the wifi.

I also have an archer T2U nano usb wifi adapter however it requires [Wireless USB Big Sur Adapter](https://github.com/chris1111/Wireless-USB-Big-Sur-Adapter) to works. You also need to change some settings in config.plist to make it work so I don't think it is worthwhile to use it if you have a working Wifi card.

### Keyboard
The keyboard is working however the brightness control buttons are not working. The adjust volumes buttons are fine though. You can assign F11 and F12 as adjust brightness keys in Keyboard -> Shortcuts -> Display. However the adjust brightness and adjust volumes key are different by fn. One of them will require fn key to function.

### Touchpad
According to device manager, my touchpad is Microsoft Precision Touchpad so VoodooI2C and VoodooI2CHID kexts with SSDT-GPIO theoretically make it work. However, it didnt work. This isn't worth fixing. Use a mouse instead. 

### Minimum Brightness in Windows is brighter than Mac
Not sure about the reason

## Dual Booting
It is better to use different drives for each OS. I have 2 SSD drives with Windows on the first one. Make a >200MB FAT32 partition on the second drive with macOS. It will act as an EFI partition. DON'T mess with original Windows EFI partition at all or you will end up like me with everything in MAIN DISK *gone* because I deleted important Windows bootloader and stupidly thought that it will be recovered. Use a tool like [EasyUEFI](https://www.easyuefi.com/index-us.html) to edit boot entries and set the OpenCore bootloader partition as default bootloader. 

*Remember to always backup important files even if they are in another drive because you might find a new way to mess up your life.*
