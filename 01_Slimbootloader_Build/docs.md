

# Slim Bootloader (SBL) – QEMU Build 


Build Steps : 

1. Update the system :

```bash
sudo apt update && sudo apt upgrade -y
```

2. Install all required dependencies:

```bash
sudo apt install -y build-essential python3 git iasl nasm uuid-dev libssl-dev gcc-multilib
```

> **Why ?**

* iasl - Compiles ACPI tables
* nasm - Assembler for low-level firmware code
* gcc-multilib - to change the bits - 32bit
* uuid-dev / libssl-dev - Used for signing and security features

---

## clone slimbootloader ssource code with submodules 

```bash
git clone --recurse-submodules https://github.com/slimbootloader/slimbootloader.git
cd slimbootloader
```

 `--recurse-submodules` is must because SBL depends on multiple external components.

---

## Generate Signing Keys

Slim Bootloader uses cryptographic keys for image signing.

Create a folder for keys outside the source code:

```bash
mkdir -p ../SblKeys
```

Generate the keys:

```bash
python3 BootloaderCorePkg/Tools/GenerateKeys.py -k ../SblKeys
```

Export the key directory so the build system can find it:

```bash
export SBL_KEY_DIR=$(pwd)/../SblKeys
```

N0te : set this env variable everytime before build 

---

## Build slb


```bash
python3 BuildLoader.py build qemu
```

If the build is successful, the firmware binary will be generated at:

```
Outputs/qemu/SlimBootloader.bin
```

---

## Clean the build  (Optional)

If you face build issues or want a clean rebuild:

```bash
python3 BuildLoader.py clean
```

Then rebuild:

```bash
export SBL_KEY_DIR=$(pwd)/../SblKeys
python3 BuildLoader.py build qemu
```

---

## Python Compatibility issue Fix 

Some system require `python` to point to `python3`.

Install the compatibility package:

```bash
sudo apt install -y python-is-python3
```

Verify:

```bash
python --version
```

Expected output:

```
Python 3.x.x
```

---

## Run Slim Bootloader on QEMU

Install QEMU:

```bash
sudo apt install -y qemu-system-x86
```

Run Slim Bootloader:

```bash
qemu-system-x86_64 \
  -nographic \
  -machine q35 \
  -pflash Outputs/qemu/SlimBootloader.bin
```
- Output : 

![alt text](image.png)


> a*Explanation*

* `-machine q35` - Modern Intel chipset 
* `-pflash` - Loads firmware image into flash memory
* `-nographic` - Runs in terminal 

---

