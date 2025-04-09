# Dell Latitude 5410 OC EFI
Opencore EFI for Dell Latitude 5410

### Important Notes:
- **Generate Your Own SMBIOS**: To ensure the proper functioning of iServices (iMessage, FaceTime, etc.), it’s critical to **generate your own valid SMBIOS**.
  
- **Wi-Fi Support**: This EFI uses `itlwm` (Intel Wi-Fi) for wireless connectivity. To enable Wi-Fi, download and install [HeliPort](https://github.com/OpenIntelWireless/HeliPort) for easier management.
  
- **CFG-Lock & DVMT**: One must unlock CFG & allocated DMVT memory before installation.
  
- **NVME Storage**: The default intel NVME does not support Hackintosh, you must replace it.

---

## Laptop Specifications

| Component                  | Specification                         |
|----------------------------|---------------------------------------|
| **CPU**                     | Intel i5-10310U                      |
| **iGPU**                    | Intel® UHD 620 Graphics              |
| **Storage**                 | SN 570 1TB NVME                      |
| **SMBIOS**                  | MacBookPro 16,3                      |
| **macOS Version**           | macOS Sequoia                        |

---

## Creating a Bootable USB

To create a bootable USB installer for macOS with OpenCore, follow the official [Dortania OpenCore Installation Guide](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/). This guide provides step-by-step instructions for setting up a USB installer, downloading necessary files, and configuring OpenCore for macOS installation.

---

## BIOS Settings

Before booting into macOS, it’s essential to configure specific BIOS settings to ensure macOS boots properly and functions optimally on the Dell Latitude 5410.

### Enable:
- **SATA Operation**: AHCI
- **Fastboot**: Minimal (Optional)

### Disable (optional):
- **Secure Boot**
- **Absolute**
- **Intel SGX**
- **Wake on AC**
- **Wake on Dell USB-C Dock**
- **Enable UEFI Network Stack**

---

## ⚠️ Important! Hidden BIOS Setting Adjustments ⚠️

This EFI configuration does not enable `AppleXcpmCfgLock`, `framebuffer-fbmem`, or `framebuffer-stolenmem`. To address this, you will need to use **modGRUBShell.efi** to make the necessary adjustments before installation. Make sure to follow these steps:

1. **Disable CFG Lock**  
   Run the following command in `modGRUBShell`:  
   ```
   setup_var_cv CpuSetup 0x3E 0x1 0x0
   ```

2. **Set DVMT Pre-Allocated Memory to 64MB**  
   To configure DVMT pre-allocated memory, run:  
   ```
   setup_var_cv SaSetup 0xF5 0x1 0x2
   ```

3. **Set DVMT Total Graphics Memory to Maximum**  
   To allocate the maximum memory to the iGPU, run:  
   ```
   setup_var_cv SaSetup 0xF6 0x1 0x3
   ```

These adjustments ensure compatibility with macOS and prevent boot issues related to graphics and power management.

---

## Enable Hibernation (S4) (This EFI has S4 enabled by default)

To enable **S4 Hibernation (Write-to-Disk)**, follow these steps:

1. Set `ThirdpartyDrives` to `true` in your `config.plist`. (Only required if you use SATA M.2)
2. Set `Hibernatemode` to `Auto` in the `config.plist`.
3. Open the terminal and run:  
   ```
   sudo pmset hibernatemode 3
   ```
If hibernation function is still not working,

4. Put `Hibernationfixup.kext` to your EFI and add `hbfx-ahbm=XX` to your boot-args [Hibernationfixup](https://github.com/acidanthera/HibernationFixup)
5. Put `RTCMemoryFixup.kext` to your EFI and add `rtcfx_exclude=0x80-0xAB` to your boot-args [RTCMemoryFixup](https://github.com/acidanthera/RTCMemoryFixup)
6. Run the following command in `modGRUBShell.efi`:  
   ```
   setup_var_cv PchSetup 0x16 0x1 0x0
   setup_var_cv PchSetup 0x04 0x1 0x3
   ```
---
