---
# Layout
layout: post

#Author
author: Sanam Shakya
date: 2020-05-10 14:30:05 +0545
---

- Installation of development tools Vivado and PetaLinux 2019.2 version
- Understand the Architecture of Zynq MPSoC(Multi Processor System On Chip) and role of various toolchain
- Implementing various subtasks using PetaLinux as guided by PetaLinux Tool Reference Guide UG 1144

  - Creating PetaLinux Project using BSP for Xilinx ZCU102

    `petalinux-create -t project -s <path-to-bsp>`

  - Creating PetaLinux Project using template and and hardware description

    `petalinux-create -t project --template zynqMP -name <project-name>`

    `petalinux-config --get-hw-description=<PATH-TO-HDF/XSA DIRECTORY>`

  - Doing the first build to fetch and compile the linux

    `petalinux-build`

  - Booting the petaLinux image on QEMU

    `petalinux-boot --qemu --kernel`

  - BSP packaging for distributing project between team and customers

    - Simple BSP packaging

      `petalinux-package --bsp -p <plnx-proj-root> --output MY.BSP`

    - Additional BSP packaging along with hardware source

      `petalinux-package --bsp -p <plnx-proj-root> --hwsource <hw-projectroot> --output MY.BSP`

    - Adding linux packages like python, gstreamer, QT

      `petalinux-config -c rootfs`

    This will lead to menuconfig from which you can add multiple packages from the list.

## PetaLinux Project Structure from docs Appendix B

hw-description and configuration of kernel and roofs are available at `project-spec/hw-description` and `projecct-spec/configs` folders these are auto-generated when we use above petalinux commands to configure the project.
Similarly auto-generated device tree files are present in `/components/plnx_workspace/device-tree/devicetree/` folder, it has base device tree for the required hardware from which other project specific device tree is configured
Project Specific device tree is located at `/project-spec/meta-user/recipes-bsp/devicetree/files/`
`system-user.dtsi` is used to describe the hardware description in the device tree.

## Adding custom/user application in linux rootfs

To do

## Linux Kernel Folder Structure and Location in PetaLinux

It is located at `firstProject/build/tmp/work/zcu102_zynqmp-xilinx-linux/linux-xlnx/4.19-xilinx-v2019.2+gitAUTOINC+b983d5fd71-r0/linux-zcu102_zynqmp-standard-build/source/`

Also how does linux boot works:
FSBL
UBoot
Kernel -- start_kernel -- source code located at `source/init/main.c`
