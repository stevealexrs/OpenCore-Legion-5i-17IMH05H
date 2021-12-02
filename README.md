# OpenCore-Legion-5i-17IMH05H
 
This repository contains EFI files for OpenCore 0.7.5 for Legion-5i-17IMH05H with macOS Monterey 12.0.1

This is the main guide I follow: [OpenCore Dortania](https://dortania.github.io/OpenCore-Install-Guide/)

Note: The commit descriptions are inaccurate since all the files are uploaded at the same time.

Note: This repo contains extra properties files added by Mac. (.DS_STORE, .\__filename_ etc)

## Specifications

| | |
|-|-|
|CPU| Intel i7-10750H (10th gen Comet Lake/Final apple-supported gen 😞) |
|GPU| Intel UHD Graphics 630 iGPU + Nvidia GeForce RTX 2060 dGPU |
|Display| 17.3" FHD 1920x1080 |
|Wifi| Intel Wireless AX201 |
|Audio Codec| Realtek ALC257 |
|Ethernet| Realtek PCIe GBe Family Controller |
|Memory| Crucial DDR4-3200 SODIMM (2x32GB) |
|Storage| SkHynix 1TB SSD + Kingston 2TB M.2 NVMe SSD |

## Status

| Features | Status |
|----------|--------|
| Battery  | Working |
| Audio | Working |
| Camera | Working |
| On Screen Brightness Control | Working |
| USB ports | Working |
| Mouse (inclduing Bluetooth) | Working |
| WiFi/Bluetooth | Working |
| Keyboard | Mostly working |
| Touchpad | Not working |
| Mac App Store Login | Not working |

## Troubleshooting
Make sure you have followed all recommended steps in Dortania guide before continuing.
### Kernel Panic
You need 3 additional things to fix this (in order):
1. Add CpuTscSync.kext
2. Set DevirtualiseMmio = False
3. Disable unsupported SkHynix NVMe

Adding CpuTscSync.kext and setting DevirtualiseMmio = false in config.plist are quite straightforward. Disabling NVMe takes some time. You can first verify whether the NVMe SSD is actually causing issue by adding *nvme=-1* to (probably) disable all ssd in boot-args. If the kernel panic is gone, then the culprit is the NVMe SSD. However, this will stop other NVMe SSD too so you need to add an ssdt. 

Go to Device Manager on Windows and click *Storage Controllers*. There will be multiple *Standard NVM Express Controller* under it. The controller that stores Window is the one with driver that cannot be disabled. Then, go to *Details* and search for *BIOS device name*. You should see this:

![NVMe Controller BIOS device name](https://user-images.githubusercontent.com/59494379/144441672-da749fb9-b30c-48cc-89d1-9fce33e05cf1.png)

You can get the ssdt from [SSDT-DNVMe](https://github.com/programbw/y9000x/blob/master/EFI/CLOVER/ACPI/patched/SSDT-DNVMe.aml) or make one yourself. You can decompile the aml to dsl and change the relevant values to the value you got from *BIOS device name*. It should be quite obvious. Here is an example of the ssdt code:

```
DefinitionBlock ("", "SSDT", 2, "hack", "NVMe-Pcc", 0x00000000)
{
    External (_SB_.PCI0.RP09.PXSX, DeviceObj)     // change this if doesn't match

    Method (_SB.PCI0.RP09.PXSX._DSM, 4, NotSerialized)     // change this if doesn't match
    {
        If ((!Arg2 || !_OSI ("Darwin")))
        {
            Return (Buffer (One)
            {
                 0x03
            })
        }

        Return (Package (0x02)
        {
            "class-code", 
            Buffer (0x04)
            {
                 0xFF, 0x08, 0x01, 0x00                                    
            }
        })
    }
}
```
Don't forget to add the ssdt to the EFI and snapshot it with ProperTree.

### Ethernet
This is easily fixed by adding RealtekRTL8111.kext

### Backlight Slider Fix
Set *disable-external-gpu* field under *PciRoot(0x0)/Pci(0x2,0x0)* **or** add *-wegnoegpu* to boot-args

| disable-external-gpu | Data | 01000000 |
|-|-|-|

**Warning**: Disabling laptop dGPU through Optimus Method from this guide [Disabling laptop dGPUs (SSDT-dGPU-Off/NoHybGfx)](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/laptop-disable.html) will result in fan turning at max and computer auto shutdown.



