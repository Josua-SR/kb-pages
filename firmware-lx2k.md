
## Build Firmware from Source

We provide a general tool for building U-Boot and operating system block images [here on GitHub](https://github.com/SolidRun/lx2160a_build). For detailed usage please consult its README. **Currently recommend is the master branch!**

UEFI is a Work in Progress and will be documented eventually :(

## Binaries

At this point in time we do not provide firmware binaries due to the necessity of deployment-specific configuration. While we are working on simplifying this situation, firmware can be compiled using our tool linked above, and explicitly taking into account the **particular RAM installed** on the system, as well as the intended firmware media (SPI, microSD, ...).

## Selecting Firmware Media

By default the CEx7 module will load firmware from microSD. Different media can be selected through the dip switch labeled *SW1* - which is described in the schematics available in the *Documentation* tab of the [product page for the CEx7 LX2160A module](https://developer.solid-run.com/products/cex7-lx2160a/).
