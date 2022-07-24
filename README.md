# SolidRun's i.MX8DXL V2X SoM build scripts

## Introduction
Main intention of this repository is to a buildroot based build environment for i.MX8DXL based product evaluation.

The build script provides ready to use images that can be deployed on eMMC or booted via USB-OTG.

## Build with Docker
A docker image providing a consistent build environment can be used as below:

1. build container image (first time only)
   ```
   docker build -t imx8dxl_build docker
   # optional with an apt proxy, e.g. apt-cacher-ng
   # docker build --build-arg APTPROXY=http://127.0.0.1:3142 -t imx8mm_build docker
   ```
2. invoke build script in working directory
   ```
   docker run -i -t -v "$PWD":/work imx8dxl_build -u $(id -u) -g $(id -g)
   ```

### rootless Podman

Due to the way podman performs user-id mapping, the root user inside the container (uid=0, gid=0) will be mapped to the user running podman (e.g. 1000:100).
Therefore in order for the build directory to be owned by current user, `-u 0 -g 0` have to be passed to *docker run*.

## Build with host tools
Simply running ./runme.sh, it will check for required tools, clone and build images and place results in images/ directory.

## Boot via USB-OTG

All steps in this section requiring using NXPs `uuu` application to interface with the Boot-ROM inside the SoC. Precompiled binaries are available [here on GitHub](https://github.com/NXPmicro/mfgtools/releases), and through the package managers on some distributions.

### U-Boot only

0. Configure the DIP Switch to boot from USB. This step is optional *only before flashing the eMMC*
1. Connect the serial console to the computer, and open it.
2. Connect the first USB-OTG port via a type A to type A cable to the computer.
3. Acquire the full path to the U-Boot binary (`build/mkimage/iMX8DXL/flash.bin`) or copy it next to the uuu binary.
4. Instruct NXPs `uuu` command to send U-Boot:
   `uuu [path-to-]flash.bin`
5. Connect to power, or reset the device.
6. The serial console should now provide access to the early boot log, and u-boot commandline:

       U-Boot 2021.04-00001-g6f4a2fe897 (Jul 24 2022 - 11:18:25 +0000)

       CPU:   NXP i.MX8DXL RevA1 A35 at 1200 MHz at 37C

       Model: NXP i.MX8DXL EVK Board
       Board: iMX8DXL EVK
       Boot:  USB
       DRAM:  1019.8 MiB
       MMC:   FSL_SDHC: 0, FSL_SDHC: 1
       Loading Environment from MMC... MMC: no card present
       *** Warning - No block device, using default environment

       In:    serial
       Out:   serial
       Err:   serial

        BuildInfo:
         - SCFW c1e35e09, SECO-FW b3c3cbc7, IMX-MKIMAGE 22346a32, ATF 05f788b
         - U-Boot 2021.04-00001-g6f4a2fe897
         - V2X-FW 2c8f793d version 0.0.4

       MMC: no card present
       Detect USB boot. Will enter fastboot mode!
       Net:   pca953x gpio@20: Error reading direction register

       Warning: ethernet@5b050000 (eth1) using random MAC address - ca:ef:1c:f5:5c:db
       eth1: ethernet@5b050000 [PRIME]
       Fastboot: Normal
       Boot from USB for mfgtools
       *** Warning - Use default environment for                                mfgtools
       , using default environment

       Run bootcmd_mfg: run mfgtool_args;if iminfo ${initrd_addr}; then if test ${tee} = yes; then bootm ${tee_addr} ${initrd_addr} ${fdt_addr}; else booti ${loadaddr} ${initrd_addr} ${fdt_addr}; fi; else echo "Run fastboot ..."; fastboot auto; fi;
       Hit any key to stop autoboot:  0

       ## Checking Image at 83100000 ...
       Unknown image format!
       Run fastboot ...
       auto usb 0

   The last line indicates that the bootloader is waiting for further USB commands via the fastboot protocol. By pressing Ctrl+C the U-Boot commandline can be accessed instead.

## Flash Bootloader to eMMC

TBD explain, think about boot0/boot1, etc.
