1.Install tools and libraries needed for coreboot
- sudo apt-get install -y bison build-essential curl flex git gnat libncurses-dev libssl-dev zlib1g-dev pkgconf

2. Step 2 - Download coreboot source tree
- git clone https://review.coreboot.org/coreboot
  cd coreboot
3. build Coreboot toolchain
- make crossgcc-i386 CPUS=$(nproc)

3. Build the payload - coreinfo
    make -C payloads/coreinfo olddefconfig
    make -C payloads/coreinfo
4. Configure the build
- make menuconfig

 1. Select Payload 
    - go to payload to add
        - Select edk2 Payload 
        ![alt text](image-1.png)

        - EDK II build type --> Build UefiPayloadPkg 
        - Tianocore's EDK II Payload --> official edk2 repository
        - GIT URL for edk2 --> https://github.com/tianocore/edk2 


 . Go to Main Board -
    Select main board Model as QEMU x86 q35/ich9
    ![alt text](image.png)

  - Save + Ok + Exit 

5. Check the configuration 
   ```make savedefconfig
    cat defconfig

6. Build Coreboot
   ```make 

successful build 
![alt text](image-2.png)


## Test the Build 

```
qemu-system-x86_64     -M q35     -pflash build/coreboot.rom     -drive file=/home/apurv-pt8167/Desktop/gbootloader/grub/fat_disk.img,format=raw,if=ide     -m 512     -serial mon:stdio


```
- output - shell will appear
![alt text](image-3.png)

- start the uefi shell from bootmenu/ boot device manager / uefi shell


look for the bootable device/disk

1. Enter FSO:
2. add command : ```vmlinuz initrd=initrd.img console=ttyS0```
3. Booting will start 
4. Youll see the successful control handover to kernel
![alt text](image-4.png)

7. Use -pflash with qemu  instead of BIOS as modern uefi needs to read and write in the flash chip