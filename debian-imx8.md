## Introduction

Debian and its derivatives are a large family of Linux operating systems. This includes Debian, Ubuntu, Xubuntu, Kubuntu, and many specialized distributions. All distributions share the same package management structure. Packages available for one version are very likely to be compatible with other versions.

The Debian family of operating systems are very versatile. As a result there are many ways to install and set-up one of these systems.

Debian focuses on stability and security of its releases. As a result, new features are not added to existing installations, and new releases are not frequent. Software packages are updated for security releases only. There are also testing and unstable versions. These are development versions, which could become the next main version. Testing and unstable versions are updated frequently, but may be unstable.

## Official SolidRun Images

### Support Matrix

| Release | Support | OpenGL-ES | Accelerated Video Coding | Desktop |
| --- | --- | --- | --- | --- |
| Bullseye (testing) | Yes | Framebuffer, Wayland, | Framebuffer | Weston |
| Buster | Yes | - | - | - |
| Stretch | Yes | - | - | - |

Due to the requirement of the GNU libc version 2.29 and later by the GPU userspace binaries provided by NXP, supporting multimedia acceleration on versions of Debian before Bullseye isn't possible.
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

### Accelerated OpenGL-ES - Kernel-Space

**Note: Currently requires a the NXP BSP Release 1.1.0 - which hasn't received security updates in over a year and should nto be used in production! Once a better kernel is available, it will be made available through apt and this section removed.**

    # if cross-compiling, get cross-compiler
    apt-get install build-essential-arm64

    # get source code
    git clone https://github.com/Josua-SR/linux.git
    cd linux; git checkout -b rel_imx_4.19.35_1.1.0+josua origin/rel_imx_4.19.35_1.1.0+josua

    # configure
    wget -O .config https://gist.github.com/Josua-SR/66c7aaa64221ef16b94fe8aabc61ab77/raw/abd492bac4461e66d98ef920d5faa496d43a2a5a/nxp-4.19-1.1.0-solidrun-config
    # optionally customize
    make ARCH=arm64 menuconfig

    # compile
    make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4 dtbs Image modules

    # collect binaries
    RELEASE=$(make kernelrelease)
    mkdir -p O/boot O/lib/modules O/usr/lib/linux-image-$RELEASE
    make INSTALL_MOD_PATH="$PWD/O" modules_install
    cp -v arch/arm64/boot/Image O/boot/vmlinuz-$RELEASE
    cp -v System.map O/boot/System.map-$RELEASE
    cp -v .config O/boot/config-$RELEASE
    find arch/arm64/boot/dts -name "*.dtb" -exec cp -v {} O/usr/lib/linux-image-$RELEASE/ \;
    cd O; tar cf ../kernel-$RELEASE.tar *; cd ..
    ls -lh kernel-$RELEASE.tar

    # Unpack on target system
    sudo tar --keep-directory-symlink -C / -xvf kernel-$RELEASE.tar
    # make initrd
    sudo update-initramfs -c -k $RELEASE
    # select new kernel for future boots
    sudo flash-kernel --force $RELEASE

### Accelerated OpenGL-ES - User-Space

**Note: This section requires Debian Bullseye or later! On older releases, container technologies such as docker can be used to avoid a distribution upgrade. Please consider [this Dockerfile](https://gist.github.com/Josua-SR/5667c005ca668e0c2a442401ba49ff55) as a starting-point.**

The drivers for the Vivante GPU that is part of i.MX8 SoCs are available as packages from our repository to be installed with apt. There are variants for Framebuffer, ~~Wayland and X11~~:

- Framebuffer:
  - runtime: `imx-gpu-viv imx-gpu-viv-fb`
  - development: `imx-gpu-viv-dev imx-gpu-viv-fb-dev`
- Wayland:
  - runtime: `imx-gpu-viv imx-gpu-viv-wl`
  - development: `imx-gpu-viv-dev imx-gpu-viv-wl-dev`
- ~~X11 (TBD)~~

When all variants are installed side by side, the default is selected through `update-alternatives` by configuring the `vivante-gal` link group:

    sudo update-alternatives --config vivante-gal

#### Demos
- eglinfo

  A small application for printing version and feature information of the active EGL and OpenGl-ES implementation. It is available [here on github](https://github.com/dv1/eglinfo) and installable through apt from our bsp repository:

      apt install eglinfo-fb
      # or any of eglinfo-wl eglinfo-x11

- glmark2

  Benchmark for OpenGL-ES 2.0 - available [here on github](https://github.com/glmark2/glmark2).
  Instructions for building and running from source:

      sudo apt install build-essential git imx-gpu-viv-fb-dev imx-gpu-viv-dev libjpeg-dev libpng-dev libwayland-dev libx11-dev pkg-config python
      git clone -b fbdev https://github.com/Josua-SR/glmark2.git
      cd glmark2
      ./waf configure --with-flavors=drm-glesv2,wayland-glesv2,x11-glesv2
      ./waf build -j4
      sudo ./waf install

      glmark2-es2-drm # on Framebuffer
      #glmark2-es2-wayland # on Wayland
      #glmark2-es2 # on X

- [gbm_es2_demo](https://github.com/ds-hwang/gbm_es2_demo)

- QT5 cube demo

       sudo apt install libqt5gui5-gles qtbase5-gles-dev qt5-eglfs-integration-vivante qt5-eglfs-integration-vivante-wayland
       git clone https://code.qt.io/qt/qtbase.git
       cp -r qtbase/examples/opengl/cube ./qt5-cube
       cd qt5-cube

       export QT_SELECT=5
       export QMAKE_LIBS_EGL=/usr/lib/galcore/libEGL.so QMAKE_LIBS_OPENGL_ES2=/usr/lib/galcore/libGLESv2.so
       qmake QMAKE_LIBS_OPENGL_ES2=/usr/lib/galcore/libGLESv2.so
       make
       env QT_QPA_PLATFORM=eglfs QT_QPA_EGLFS_INTEGRATION=eglfs_viv ./cube

### Accelerated Video Decoding and Playback

**Note: Currently requires an updated kernel to be released soon(TM); Once done this note will be removed.**

While directly using the VPU libraries provided by NXP is possible, that use-case goes beyond the scope of this document. Instead we are using the GStreamer 1.0 plugins for i.MX platforms to make use of those acceleration blocks in the SoC.

The plugins are available in our repos and installed by `apt install gstreamer1.0-imx`

Note that it is essential to configure GStreamer for using OpenGL-ES.

- On Framebuffer:

       export GST_GL_PLATFORM=egl GST_GL_API=gles2 GST_GL_WINDOW=gbm

- On Wayland: *TBD*

- On X11: *TBD*

#### Examples

The examples below depend on a number of additional gstreamer elements and utilities:

    sudo apt install gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-alsa gstreamer1.0-tools
    sudo apt install gstreamer1.0-imx

- automatic with playbin (720p)

       gst-launch-1.0 playbin3 uri=http://distribution.bbb3d.renderfarming.net/video/mp4/big_buck_bunny_720p_surround.avi
       # for wayland, append video-sink=waylandsink

- automatic with playbin (1080p)

       gst-launch-1.0 playbin3 uri=http://distribution.bbb3d.renderfarming.net/video/mp4/bbb_sunflower_1080p_30fps_normal.mp4
       # for wayland, append video-sink=waylandsink

### Wayland

While a special release of Weston by NXP is available, the default weston package in Debian can be used instead with the drm backend.

Installation including the vivante OpenGL implementation is as simple as

    sudo apt install imx-gpu-viv imx-gpu-viv-wl weston

Then, to start - use weston-launch on a terminal session (**not ssh or uart!**)

    weston-launch

### XWayland

TBD.

### X11

Abandoned. NXP does not officially support using Xorg on the i.MX8M SoCs - causing a gap in functional xorg drivers required for DRI and GLX.

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
