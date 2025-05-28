# **Dell Latitude 5410 OpenCore EFI**

OpenCore EFI for Dell Latitude 5410 (P98G007) and 5310

---

### Important Notes:

* **SMBIOS**: Generate your own to enable iServices (iMessage, FaceTime, etc.).
* **Wi-Fi**: Uses `itlwm.kext`. Install [HeliPort](https://github.com/OpenIntelWireless/HeliPort) for Wi-Fi management.
* **CFG-Lock & DVMT**: Must unlock CFG and set DVMT memory before installation.

---

## **Laptop Specs – Latitude 5410 (P98G007)**

| Component    | Specification   |
| ------------ | --------------- |
| **CPU**      | Intel i5-10310U |
| **iGPU**     | Intel UHD 620   |
| **Storage**  | SN570 1TB NVMe  |
| **Wi-Fi/BT** | Intel AX201     |
| **SMBIOS**   | MacBookPro 16,3 |
| **macOS**    | macOS Sequoia   |

---

## **Create Bootable USB**

Follow the [Dortania Guide](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/) for step-by-step instructions.

---

## **BIOS Settings**

### Enable:

* SATA Operation: AHCI
* Fast Boot: Minimal

### Disable:

* Secure Boot
* Absolute
* TPM
* Wake on AC
* Wake on Dell USB-C Dock
* UEFI Network Stack
* Intel VT-D
* Touchscreen

---

## **3.5mm Headphone Jack**

[ComboJack](https://github.com/macos86/ComboJack).

Add boot-args: `alcverbs=1` or use `DeviceProperties`.

---

## ⚠️ **Hidden BIOS Settings (modGRUBShell Required)**

This EFI doesn’t enable `AppleXcpmCfgLock`, `framebuffer-fbmem`, or `framebuffer-stolenmem`. Use **modGRUBShell.efi**:

1. **Disable CFG Lock**

   ```
   setup_var_cv CpuSetup 0x3E 0x1 0x0
   ```

2. **Set DVMT Pre-Allocated to 64MB**

   ```
   setup_var_cv SaSetup 0xF5 0x1 0x2
   ```

3. **Set DVMT Total GFX Memory to Max**

   ```
   setup_var_cv SaSetup 0xF6 0x1 0x3
   ```

---

## **Enable Hibernation (S4)**

(Supported by default in this EFI)

1. In `config.plist`:

   * `ThirdpartyDrives = true` (if using SATA M.2)
   * `Hibernatemode = NVRAM`

2. Run:

   ```
   sudo pmset hibernatemode 3
   ```

3. If it fails, in `modGRUBShell` run:

   ```
   setup_var_cv PchSetup 0x16 0x1 0x0     // Disable RTC Lock  
   setup_var_cv Setup 0x14 0x1 0x0        // Disable Low Power S0 Idle
   ```

4. Add to EFI:

   * `Hibernationfixup.kext` + boot-args: `hbfx-ahbm=129`
   * `RTCMemoryFixup.kext` + boot-args: `rtcfx_exclude=0x80-0xAB`

---

## **Bluetooth fix for AX 201**

1. **Disable intel WoV(Wake on Voice)**

   With **modGRUBShell.efi**

   ```
   setup_var_cv PchSetup 0x586  0x1 0x0
   ```

2. **Turn off while the system goes sleep**

   [How to turn off bluetooth while sleep with Sleepwatcher](https://github.com/amane1234/Wakeup_bluetooth_fix).



## **Issues & Tips**

* Realtek SD card reader doesn’t work on Sequoia. However, you may build & try this: [RealtekCardReaderFriend for Sequoia](https://github.com/Lorys89/RealtekCardReaderFriend)
* TPD1 device can’t be disabled. Disabling the device make the touchpad irresiponsive.
* Touchscreen causes erratic behavior.
* The default PL1/PL2 are set to 45W 64W, change the value for better power mamagement.
* iGPU (RC6) and NVMe may conflict. If so, add `forceRenderStandby=0`, [NVMe Panic Details](https://github.com/acidanthera/bugtracker/issues/1193)

---

