## Introduction

Debian and its derivatives are a large family of Linux operating systems. This includes Debian, Ubuntu, Xubuntu, Kubuntu, and many specialized distributions. All distributions share the same package management structure. Packages available for one version are very likely to be compatible with other versions.

The Debian family of operating systems are very versatile. As a result there are many ways to install and set-up one of these systems.

Debian focuses on stability and security of its releases. As a result, new features are not added to existing installations, and new releases are not frequent. Software packages are updated for security releases only. There are also testing and unstable versions. These are development versions, which could become the next main version. Testing and unstable versions are updated frequently, but may be unstable.

## Official SolidRun Images

### Support Matrix

| Release | Support | Window-Systems for Multimedia | Accelerated Desktops |
| --- | --- | --- | --- |
| Bullseye (testing) | Yes | Framebuffer (TBD), Wayland (TBD), X11 (TBD) | - |
| Buster | Yes | - | - |
| Stretch | Yes | - | - |

Due to the requirement of the GNU libc version 2.29 and later by the GPU sserspace binaries provided by NXP, supporting multimedia acceleration on versions of Debian before Bullseye isn't possible.
If using a stable release is desired, we suggest the use of containers for providing a newer userspace to applications.

### Download and Install

All images are available for download at https://images.solid-build.xyz/IMX8/Debian/. Please scroll to the bottom to find a log of important changes.
Several tools are available for writing them to block storage, including etcher.io, win32diskimager and dd. Please make sure to decompress them first!

**Special care must be taken while selecting the right file:**

- *\*.tar.xz* are meant for advanced users who wish to provision the root filesystem themselves
- *\*.img.xz* have to be uncompressed and written directly to a block device
- *\*-imx8mq-\*.\** indicates a variant specially created for the i.MX8M Quad
- *\*-imx8mm-\*.\** indicates a variant specially created for the i.MX8M Mini
- *\*-sdhc.\** indicates that the image includes a bootloader suitable for use with the microSD slot.

### Install to eMMC

The eMMC is accessible from the running system as `/dev/mmcblk0`. Install the
Debian software image to eMMC by writing the image directly to `/dev/mmcblk0`:

	xzcat sr-imx8-debian-buster-20191120-cli-imx8mq-sdhc.img.xz | dd of=/dev/mmcblk0 bs=4k conv=fdatasync

Set SW3 DIP switches to boot from eMMC as documented in the [HummingBoard
Pulse Boot
Select](https://developer.solid-run.com/knowledge-base/hummingboard-pulse-boot-select/)
article.

### Get Started

Once you are greeted by a login prompt, the default username and password are both "debian". For security reasons there is **no** root password! If you really need one, you can run `sudo passwd root` to set your own.

**For Cubox Pulse however the DeviceTree has to be selected on the U-Boot Shell:**

Connect to the serial console with a terminal emulator of choice, then power on the device and wait for the line that says `Hit any key to stop autoboot: 3` – then immediately press any key, to abort! The terminal will drop to the U-Boot Shell as follows:

    U-Boot SPL 2018.11-00078-g0dd51748c2a (Dec 16 2018 - 18:35:18 +0100)
    PMIC:  PFUZE100 ID=0x10
    Normal Boot
    Trying to boot from MMC2
    NOTICE:  Configureing TZASC380
    NOTICE:  BL31: v1.6(release):v1.6-110-g0eb2df45
    NOTICE:  BL31: Built : 13:56:07, Nov 29 2018
    NOTICE:  sip svc init
    U-Boot 2018.11-00078-g0dd51748c2a (Dec 16 2018 - 18:35:18 +0100)
    CPU:   Freescale i.MX8MQ rev2.0 at 1000 MHz
    Reset cause: POR
    Model: SolidRun i.MX8MQ HummingBoard Pulse
    DRAM:  3 GiB
    MMC:   FSL_SDHC: 0, FSL_SDHC: 1
    Loading Environment from MMC... OK
    In:    serial
    Out:   serial
    Err:   serial
    Net:   
    Error: ethernet@30be0000 address not set.
    Error: ethernet@30be0000 address not set.
    eth-1: ethernet@30be0000
    Hit any key to stop autoboot:  0 
    =>

Now is the time to override the name of device-tree for Cubox Pulse:

    setenv fdtfile fsl-imx8mq-cubox-pulse.dtb
    saveenv

This setting is permanent as long as the boot sectors of this particular microSD aren’t overridden, e.g. by writing a new image to it.
Reboot the device by unplugging power, and let it boot up into Debian.

### Accelerated OpenGL-ES

TBD.

### Accelerated Video Decoding and Playback

TBD.

## Customize

### Using custom Device-Tree Blobs

We are trying to follow Debian design patterns where possible especially in the boot process. The *flash-kernel* application is used for installing DTBs to /boot as well as creating a boot-script for u-boot to load kernel, ramdisk and dtb.

Using a custom DTB here is as simple as putting it in **/etc/flash-kernel/dtbs** and rerunning `flash-kernel`. From this point onwards whenever an update to the kernel, or parts of the ramdisk occurs *flash-kernel* will automatically pick up the provided DTB in /etc.

### Fork board-support packages

In order to properly support the i.MX8 including all of its multimedia capabilities, many custom debs that are not part of Debian have been created. These include an optimized kernel, the Vivante GPU userspace, gstreamer plugins and many more. There are 2 ways for getting the package sources:

1. use apt-get source, e.g.: `apt-get source linux-image-4.19.y-imx8-sr`

2. browse the [SolidRun github](https://github.com/SolidRun), and the old [packaging organization](https://github.com/mxOBS)

All of our packages can be built by invoking dpkg-buildpackage directly, or through git-buildpackage. If there are specific questions regarding packaging of customizations please contact us.

Also note that [our instance](https://obs.solid-build.xyz/) of the [OpenBuildService](https://openbuildservice.org/) which we use to automate builds is available to the public on request.

## Pure Debian (upstream)

Our long-term goal is enabling as much of our hardware upstream as part of Debian where feasible. Success is highly dependent on the efforts of the general community around the i.MX8M SoCs, as well as the amount of proprietary blobs involved. At this point no support for the SolidRun i.MX8M boards is present in Debian.
