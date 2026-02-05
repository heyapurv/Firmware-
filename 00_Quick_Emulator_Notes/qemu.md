

# QEMU Notes 


Quick Emulator (qemu) is opensource machine emulator, it acts as a virtual motherboard + cpu + devices 
we can use it to test : 
* coreboot
* EDK2 (UEFI)
* Slim Bootloader (SBL)

**No real hardware needed. No risk of bricking laptops.**

---

## Why Use QEMU

| Problem                 | How QEMU Helps              |
| ----------------------- | --------------------------- |
| No physical test board  | Emulates Intel/PC platforms |
| Debugging firmware logs | Serial console output       |
| Testing bootloaders     | Boots kernel & disk images  |
| Learning BIOS/UEFI flow | Safe playground             |

Think of QEMU as **“hardware inside a terminal”**.

---

## Structure of a QEMU Command

```bash
qemu-system-x86_64 [machine] [firmware] [memory] [disk] [console]
```

### Meaning

| Part       | What it Represents       |
| ---------- | ------------------------ |
| `machine`  | Motherboard / chipset    |
| `firmware` | BIOS / UEFI / Bootloader |
| `memory`   | RAM size                 |
| `disk`     | Hard disk / boot image   |
| `console`  | Where logs are printed   |

---

## Machine & CPU Options (Motherboard)

| Option              | Meaning                     | When to Use               |
| ------------------- | --------------------------- | ------------------------- |
| `-M q35`            | Modern Intel chipset (PCIe) | **EDK2, Slim Bootloader** |
| `-M pc` or `i440fx` | Legacy chipset              | coreboot + GRUB           |
| `-accel kvm`        | Uses real CPU (fast)        | Linux host + slow boot    |

inshort : 

* Modern firmware -> `q35`
* Old/legacy -> `pc`

---

## Firmware Loading Options (BIOS / UEFI)

| Option             | What It Does                    | Best For        |
| ------------------ | ------------------------------- | --------------- |
| `-bios file.rom`   | Loads firmware as read-only ROM | coreboot + GRUB |
| `-pflash file.bin` | Emulates writable flash chip    | EDK2 (UEFI)     |
| `-drive if=pflash` | Advanced flash mapping          | Slim Bootloader |

### Why `pflash`?

UEFI needs **NVRAM** (variables).
`-bios` - does NOT allow writing
`-pflash` - allows writing

---

##  Disk & Storage Options

| Option                 | Meaning                   |
| ---------------------- | ------------------------- |
| `-drive file=disk.img` | Disk image file           |
| `format=raw`           | Raw disk format           |
| `if=ide`               | Old, compatible interface |
| `if=virtio`            | Fast, modern virtual disk |

### Disk Interface Guide

| Firmware        | Recommended Disk     |
| --------------- | -------------------- |
| GRUB            | `if=ide`             |
| Slim Bootloader | `if=virtio`          |
| UEFI            | `if=ide` or `virtio` |

---

##  Console & Debugging (VERY IMPORTANT)

| Option              | Purpose                   |
| ------------------- | ------------------------- |
| `-nographic`        | No GUI, terminal only     |
| `-serial stdio`     | Firmware logs in terminal |
| `-serial mon:stdio` | Logs + QEMU monitor       |

**Always use serial output in firmware work**

---

##  Commands for : 

###  Slim Bootloader

```bash
qemu-system-x86_64 \
  -machine q35 \
  -nographic \
  -pflash SlimBootloader.bin \
  -drive file=disk.img,if=virtio
```

---

###  coreboot + GRUB

```bash
qemu-system-x86_64 \
  -bios coreboot.rom \
  -m 2G \
  -serial stdio \
  -drive file=disk.img,if=ide
```

---

###  coreboot + EDK2

```bash
qemu-system-x86_64 \
  -M q35 \
  -pflash coreboot.rom \
  -m 2G \
  -serial mon:stdio \
  -drive file=disk.img,if=ide
```

---

##  Keyboard Shortcuts (When Using Terminal)

| Keys             | Action            |
| ---------------- | ----------------- |
| `Ctrl + A` -> `X` | Exit QEMU         |
| `Ctrl + A` -> `C` | Open QEMU monitor |
| `system_reset`   | Reboot VM         |
| `info registers` | CPU state         |
| `quit`           | Exit              |

---

## Common Problems & Fixes

| Problem            | Reason            | Fix                    |
| ------------------ | ----------------- | ---------------------- |
| Black screen       | No serial output  | Add `-serial stdio`    |
| Kernel panic (VFS) | Wrong root device | Check `root=/dev/sdaX` |
| No `fs0:` in UEFI  | Disk not FAT32    | Format EFI partition   |
| Very slow boot     | No acceleration   | Add `-accel kvm`       |

---

## Beginner Mental Model (Important)

| Component   | Real World     | QEMU                |
| ----------- | -------------- | ------------------- |
| Motherboard | Physical board | `-machine`          |
| BIOS chip   | Flash ROM      | `-bios` / `-pflash` |
| Hard disk   | SSD/HDD        | `disk.img`          |
| UART logs   | Serial cable   | `-serial stdio`     |

---
