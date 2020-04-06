## Introduction

Below are details how to build ATF (ARM Trusted Firmware), U-Boot (boot loader) and Linux kernel for i.MX8M HummingBoard Pulse and CuBox-Pulse

## Building U-Boot from Sources

### Toolchain

You can either build or download a ready-to-use toolchain. An example of such toolchains are from Linaro website.

When writing this document the following toolchain was used – http://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu.tar.xz

Linaro updates it’s toolchain quite often and more frequent one can be downloaded from here – http://releases.linaro.org/components/toolchain/binaries/

Download and extract the toolchain into some place; and as instructed below the CROSS_COMPILE environment variables needs to be set to the path of the toolchain prefex.

For instance if the toolchain was extracted under /opt/imx8m/toolchain/gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu/ then the CROSS_COMPILE needs to be set as follows –

    export ARCH=arm64
    export CROSS_COMPILE=/opt/imx8m/toolchain/gcc-linaro-7.3.1-2018.05-i686_aarch64-linux-gnu/bin/aarch64-linux-gnu-

### Download Source and Firmware

    git clone https://source.codeaurora.org/external/imx/imx-atf.git -b imx_4.19.35_1.0.0 arm-trusted-firmware
    git clone https://github.com/SolidRun/u-boot.git -b v2018.11-solidrun
    wget https://www.nxp.com/lgfiles/NMG/MAD/YOCTO/firmware-imx-7.9.bin

### ATF

Building ATF is as follows – *make sure you have set your ARCH and CROSS_COMPILE environment variables as noted above*

    cd arm-trusted-firmware
    make PLAT=imx8mq bl31
    cp build/imx8mq/release/bl31.bin ../u-boot/
    cd ..

### Extract and copy firmware

Extract the NXP firmware archive and accept the end user agreement

    chmod +x firmware-imx-7.9.bin
    ./firmware-imx-7.9.bin
    cp firmware-imx-7.9/firmware/hdmi/cadence/signed_hdmi_imx8m.bin u-boot/
    cp firmware-imx-7.9/firmware-imx-7.9/firmware/ddr/synopsys/lpddr4*.bin u-boot/

### U-Boot

Build U-Boot and generate the image - *make sure you have set your ARCH and CROSS_COMPILE environment variables as noted above*

## Deploying U-Boot

### to microSD

    sudo dd if=flash.bin of=/dev/sd[x] bs=1024 seek=33

## Linux Kernel

Kernel sources are found on SolidRun’s github site – https://github.com/SolidRun/linux-fslc.git

To build from source; follow the following steps –

    git clone https://github.com/SolidRun/linux-fslc.git -b solidrun-imx_4.9.x_imx8m_ga
    cd linux-fslc
    export CROSS_COMPILE=<point to you ARM64 cross compiler as indicated above>
    export ARCH=arm64
    make Image dtbs

The result files are –

    Image - kernel image in FIT format
    arch/arm64/boot/dts/freescale/fsl-imx8mq-hummingboard-pulse.dtb - HummingBoard pulse device tree

## Linux Kernel Support Matrix

Following is the support matrix for the HummingBoard Pulse and CuBox-Pulse –

| Feature | HummingBoard Pulse + i.MX8M SOM | CuBox Pulse + i.MX8M SOM |
| --- | --- | --- |
| i.MX8M quad | v | v |
| Micron 3GByte LPDDR4 (via ATF)| v | v
| SOM PCIe based 11ac / bt|v|N/A |
| SOM BT 5.0 | v | N/A |
| i.MX8M Gigabit Ethernet (with PoE 11af PD) | v | v |
| i.MX8M GPU (Vivante GC7000 Lite Driver) | v | v |
| i.MX8M Video Engine | v | v |
| HummingBoard Pulse Intel i210 NIC | v | N/A |
| eMMC | v | v |
| USB type C (USB 3.0) | v | N/A |
| 2x USB 3.0 | v (via onboard USB 3.0 hub) | v (native from i.MX8M) |
| mini PCIe (SIM card) | v | N/A |
| M.2 | v | N/A |
| HDMI 2.0 with CEC | v | v |
| Onboard analogue audio codec | x | N/A |
| FPC connector (digital audio etc…) | v | N/A |
| Reset button | v | v |
| 1x Configurable push button | x | N/A |
| MikroBus click interface | x | N/A |
| 3x LED indicators | x | N/A |
| RTC via AM1805AQ | x | N/A |
| RTC via RX6110SA | N/A | x |
| GPIO based IR receiver | x | x |
| MIPI-DSI | x | N/A |
| On SOM MIPI-CSI | x | x |
| On carrier MIPI-CSI | x | N/A |
| PWM front LED | N/A | x |
