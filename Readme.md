Author: Minh Nhat  
Date: 13/3/2024  
Description: Introduce Yocto project and how to build, flash A Yocto image for NXP board  

## Table of contents

* [Introduction](#general-info)
* [Build and flash yocto image](#build-and-flash-yocto-image)
* [Host setup](#host-setup)
* [Yocto project setup](#yocto-project-setup)
* [Image-build](#image-build)
* [Building-an-image](#building-an-image)
* [Flashing an SD card image](#flashing-an-SD-card-image)

Today I will be talking about the Yocto project quickly and how to build and flash an image for NXP board less bug 
Ok. Let's get started

## Introduce concept of yocto project quickly

What is Yocto project ?
- Yocto Project is an open source collaboration forcused on developing, flexible and customizable Linux embedded systems. 
Yocto project provide a collection of tools, metadata and development enviroments that allow developers to create the Linux distributions or customize for specific purposes 
- Input, output of yocto project:
    + Input: Set of data that describes what you want, that is your specification (Kernel Config, Hardware name, Package/Binaries to be installed) 
    + Output: Linux based Embedded Product (Bootloader,Linux Kernel Rootfs and device tree)

The compenents of yocto project
* Poky: It is a sample linux distribution of yocto. Yocto uses poky to build a Linux images(kernel, system, and application software) for targeted hardware
* OpenEmbedde-Core(OE-core): This is A tool collection used to build Linux embedded distributions
* Bitbake: A tool build an automatic distribution, it has ability build the software packages and distribution from an open source
* Meta-yocto-bsp: A (BSP) is a collection of information that defines how to support a particular hardware device, set of devices, or hardware platform
* Metadata: It refer to infomations describe, config and regulate for building, compiling and deploy system
Metadata is collection of
    + Configuration (.conf)
    + Recepies (.bb, .bbappend)
    + Classes (.bbclass)
    + Includes (.inc)
* Recipe: It is a set of instruction that decribe how to prepare or make some thing, especially a dish
+ Etension of recipe: .bb
+ A recipe decribes:
    > Where you get source code
    > which pathces options
    > Configuration options
    > Compile option (Library dependencies)
    > Install
    > License
    Examples of recipes: dhcp_4.4.1.bb, gstreamer1.0_1.16.1.bb
## Build and flash yocto image
# Preparing:
    * A SSD disk at least 120GB
    * The minimum Ubuntu version is 20.04 or later
    * A NXP embedded board (Here I will use the imx8 board)
# Host Setup
Install the nessary packages to build yocto:
```
$ sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential -y
$ sudo apt install chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils -y
$ sudo apt install iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev -y
$ sudo apt install python3-subunit mesa-common-dev zstd liblz4-tool file locales -y
```
Setting up the Repo utility:
```
$ mkdir ~/bin (this step may not be needed if the bin folder already exists)
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
$ export PATH=~/bin:$PATH
```

# Yocto project setup
First, make sure that Git is set up properly with the commands below:
```
$ git config --global user.name "Your Name"
$ git config --global user.email "Your Email"
$ git config --list
$ mkdir imx-yocto-bsp
$ cd imx-yocto-bsp
$ repo init -u https://github.com/nxp-imx/imx-manifest -b imx-linux-mickledore -m imx-6.1.55-2.2.0.xml
$ repo sync
```

# Image Build
i.MX provides a script, imx-setup-release.sh, that simplifies the setup for i.MX machines. To use the script, the name of the specific machine to be built for needs to be specified as well as the desired graphical backend:
```
$ DISTRO=fsl-imx-xwayland MACHINE=imx8mm-ddr4-evk source imx-setup-release.sh -b build-myir
```
Note:
    * DISTRO=<distro configuration name> is the distro, which configures the build environment and it is stored in meta-imx/meta-sdk/conf/distro.
    * MACHINE=<machine configuration name> is the machine name which points to the configuration file in conf/machine in meta-freescale and meta-imx.
    -b <build dir> specifies the name of the build directory created by the imx-setup-release.sh script

Note that (This is important):
    The default configuration file of Yocto will utilize all CPU cores to fetch, compile the recipes when building the image on your machine. Consequently, your machine may become very hot and overloaded. To mitigate this, you can modify the number of CPU cores used by excuting the following command:

```
$cd ~/imx-yocto-bsp/build-myir/conf
$vim local.conf
$PARALLEL_MAKE = "-j${your core number}"
```
# Building an image
```
$ bitbake imx-image-core
```
Bitbake options:
    bitbake <parameter> <component>
    -c fetch        (Fetches if the downloads state is not marked as done.)
    -c deploy       (Deploys an image or component to the rootfs.)
    -c compile -f   ( Use this option to force a recompile after the image is deployed)

# Modify u-boot-imx and kernel-imx by check git commit
u-boot:
Please click with this link to check: https://github.com/AndroidCockpit/uboot-imx/commit/develop#

```
$ cd ~imx-yocto-bsp/build-myir/tmp/work/imx8mm_ddr4_evk-poky-linux/u-boot-imx/2022.04-r0/git/arch/arm/dts
$ nano imx8mm-evk.dtsi
```
After modify code, let recompile with this command:

```
$bitbake -c compile -f u-boot-imx
$bitbake imx-boot
```

kernel-imx:
Please click with this link to check: https://github.com/AndroidCockpit/kernel_imx/commit/ac3b9715f3caef407da99f1e751289d7f5d918cc?fbclid=IwAR288WUZT9Zsdkn1is6mTFh-xU-w8DujugeXMCGNIIdbG710hh64uj_SSU0

```
$cd imx-yocto-bsp/build-myir/tmp/work/imx8mm_ddr4_evk-poky-linux/linux-imx/5.15.71+gitAUTOINC+95448dd0dc-r0/git/arch/arm64/boot/dts/freescale
$ nano imx8mm-evk.dtsi
```
After modify code, let recompile with this command:

```
$bitbake linux-imx -c compile -f
$bitbake linux-imx -c deploy
$bitbake imx-image-core
```
# Flashing an SD card image 
An SD card image file .wic contains a partitioned image (with U-Boot, kernel, rootfs, etc.) suitable for booting the corresponding hardware.
```
$cd ~imx-yocto-bsp/build-myir/tmp/deploy/images/imx8mm-ddr4-evk
```
To flash an SD card image, run the following command:
```
$zstdcat <image_name>.wic.zst | sudo dd of=/dev/sd<partition> bs=1M conv=fsync
```
