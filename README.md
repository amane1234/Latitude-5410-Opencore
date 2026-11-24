# **Dell Latitude 5410 OpenCore EFI**

OpenCore EFI for Dell Latitude 5410 (P98G007) and 5310

There will be no more update for this repo.

---

### Important Notes:

* **SMBIOS**: Generate your own to enable iServices (iMessage, FaceTime, etc.).
* **Wi-Fi**: It uses ForbiddenItlwm for Sequoia.
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
| **macOS**    | Sequoia 15.8.2  |

---

## **Create Bootable USB**

Follow the [Dortania Guide](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/) for step-by-step instructions.

---

## **BIOS Settings**

### Enable:

* SATA Operation: AHCI (Important)

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

2. In `modGRUBShell.efi` run:

   ```
   setup_var_cv PchSetup 0x16 0x1 0x0     // Disable RTC Lock  
   setup_var_cv Setup 0x14 0x1 0x0        // Disable Low Power S0 Idle
   ```

4. Add to EFI:

   * `Hibernationfixup.kext` + boot-args: `hbfx-ahbm=129`
   * `RTCMemoryFixup.kext` + boot-args: `rtcfx_exclude=0x80-0xAB` (Needed only when you see CMOS reset problem)

---

## **Force unlock on all GPIO pads(Optional)**

The Dell Latitude 5410 uses an APIC-mode trackpad and blocks changes to it for security reasons.

This EFI enables interrupt mode for both the trackpad and touchscreen via APIC. If you prefer GPIO pinning instead, you will need to modify the GPIO device settings through a BIOS mod

With **modGRUBShell.efi**

   ```
   setup_var_cv PchSetup 0x18 0x01 0x01
   ```

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

## **Notes for rps-control VS Apple GuC**

Without injecting RPS control or Apple GuC, the Intel iGPU clock remains fixed at 300 MHz, resulting in reduced performance.

Injecting RPS control allows dynamic scaling of the iGPU clock from 300 MHz to 1100 MHz, improving performance under load.

Alternatively, using Apple GuC raises the base frequency to 500 MHz, enabling a range of 500 MHz to 1100 MHz. This offers better performance but slightly bad iGPU power management.

However, there is a known issue with RPS control: after S3 (sleep) or S4 (hibernate) resume, the GPU may remain locked at 100% usage with the clock stuck at 1100 MHz, even when idle. This can lead to unnecessary power consumption and thermal load.

This efi uses forked whatevergreen, allowing  dynamic scaling of the iGPU clock from 300 MHz to 1100 MHz with Apple GuC

Recommendation:

   - If you frequently use S3/S4 sleep, it is safer to use Apple GuC or no injection to avoid the RPS bug.

   - If you do not use sleep/hibernate, RPS control is recommended for better performance.

---

## **Issues & Tips**

* Realtek SD card reader seems does not work on Sequoia.

* The default PL1/PL2 are set to 35W 64W, change the PL2 value for better power mamagement.

---

