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
* Fast Boot: Minimal (optional)

### Disable:

* Secure Boot
* Absolute
* Intel SGX
* Wake on AC
* Wake on Dell USB-C Dock
* UEFI Network Stack
* Touchscreen

---

## **3.5mm Headphone Jack**

Install [ComboJack](https://github.com/macos86/ComboJack).
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

   * `Hibernationfixup.kext` + `hbfx-ahbm=129`
   * `RTCMemoryFixup.kext` + `rtcfx_exclude=0x80-0xAB`

---

## **Known Issues & Tips**

* Realtek SD card reader doesn’t work on Sequoia.
* TPD1 device can’t be disabled. Disabling the device make the touchpad irresiponsive.
* Touchscreen causes erratic behavior.
* The default PL1/PL2 are set to 45W 64W, change the value for better power mamagement.
* iGPU (RC6) and NVMe may conflict. If so, add `forceRenderStandby=0`
  [NVMe Panic Details](https://github.com/acidanthera/bugtracker/issues/1193)

---
