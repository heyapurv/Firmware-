
---

## Part 1: GRUB BUILD DOCS

### 1. Install Dependencies

```bash
sudo apt update
sudo apt install -y git build-essential bison flex autoconf automake python3 libtool gettext libfreetype6-dev pkg-config mtools

```
### 2. Clone source code from github 
```bash
git clone https://git.savannah.gnu.org/git/grub.git
cd grub
```

### 3. Configure for UEFI x86_64

```bash
./bootstrap
./configure --target=x86_64 --with-platform=efi 

```

### 3. Build the Tools


```bash
make 

```

---

## Part 2: Generate EFI Binary

 make sure `vmlinuz` and `initrd.img` are already in this folder.

### 1. Create a basic `grub.cfg`



```bash
cat <<EOF > grub.cfg
set timeout=5
set default=0

menuentry "Linux Boot (Tiger Lake Simulation)" {
    linux /vmlinuz root=/dev/ram0 rw
    initrd /initrd.img
}
EOF

```

### 2. Run `grub-mkstandalone`

Use the `-d` flag to point to the `grub-core` directory that we just built.

```bash
./grub-mkstandalone -d grub-core/ -O x86_64-efi -o bootx64.efi  "boot/grub/grub.cfg=grub.cfg"

```

---

## Create the Bootable FAT32 Disk Image

### 1. Create the Raw Image

```bash

dd if=/dev/zero of=fat_disk.img bs=1M count=200

# fat32 format
mkfs.vfat -F 32 fat_disk.img

```

### 2. Set Up the UEFI Structure



```bash
mmd -i fat_disk.img ::/EFI
mmd -i fat_disk.img ::/EFI/BOOT

```

### 3. add the files

Copy the bootloader, kernel, and RAM disk from the current folder into the image.

```bash
#  this is bootloader
mcopy -i fat_disk.img bootx64.efi ::/EFI/BOOT/BOOTX64.EFI

#  these are Kernel and RAM Disk
mcopy -i fat_disk.img vmlinuz ::/vmlinuz
mcopy -i fat_disk.img initrd.img ::/initrd.img

```

---

