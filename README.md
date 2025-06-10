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

### Disable (Optional):

* Secure Boot
* Absolute
* TPM
* Wake on AC
* Wake on Dell USB-C Dock
* UEFI Network Stack
* Intel VT-D (If you do not want to use Apple VTD)
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

3. **Set DVMT Total GFX Memory to Max (Optional)**

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
   * `RTCMemoryFixup.kext` + boot-args: `rtcfx_exclude=0x80-0xAB` (Needed only when you see CMOS reset problem)

---

## **Bluetooth fix for AX 201(Optional)**

1. **Disable intel WoV(Wake on Voice)**

   With **modGRUBShell.efi**

   ```
   setup_var_cv PchSetup 0x586  0x1 0x0
   ```

2. **Turn off when the system wake-up**

   [How to turn off bluetooth while sleep with Sleepwatcher](https://github.com/amane1234/Wakeup_bluetooth_fix).

---

## **Notes for Low Power S0 Idle**

The default value of Low Power S0 Idle is enabled, which conflicts S3 Sleep wake up and S4 Sleep.

I strongly recommand to use S3 sleep by disabling Low Power S0 Idle capability by:

   ```
   setup_var_cv Setup 0x14 0x1 0x0        // Disable Low Power S0 Idle
   ```

However, if you want to use S0 Idle, you can add SSDT-DeepIdle.aml to your ACPI folder and leave the Setup 0x14 value at its default.

---

## **Issues & Tips**

* Realtek SD card reader doesn’t work on Sequoia. However, you may build & try this: [RealtekCardReaderFriend for Sequoia](https://github.com/Lorys89/RealtekCardReaderFriend)
* Touchscreen causes erratic behavior.
* The default PL1/PL2 are set to 35W 64W, change the PL2 value for better power mamagement.

---

## (Experimental) Bios settings for better power consumption

   ```
   setup_var_cv CpuSetup 0xDA 0x1 0x0   # Overclocking Lock Disable
   setup_var_cv CpuSetup 0x39 0x1 0x0   # C-State Auto Demotion Disable
   setup_var_cv CpuSetup 0x3A 0x1 0x3   # C-State Un-demotion at C1 and C3
   setup_var_cv CpuSetup 0x3B 0x1 0x0   # Package C-State demotion Disable
   setup_var_cv CpuSetup 0x3C 0x1 0x1   # Package C-State Un-demotion Enable
   setup_var_cv PchSetup 0x04 0x1 0x3   # Deep Sx Power policy: S4-S5/Battery
   setup_var_cv Setup 0x4B4 0x1 0x3     # Enable ASPM: L0s and L1
   setup_var_cv Setup 0x38 0x1 0x1      # Enable Sensor Standby
   setup_var_cv SaSetup 0x123 0x1 0x3   # DMI ASPM: L0s and L1
   setup_var_cv PchSetup 0x4F6 0x1 0x3  # DMI ASPM: L0s and L1
   ```
---

