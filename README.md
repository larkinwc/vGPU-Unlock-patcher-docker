# vGPU-Unlock-patcher
A solution to patch vGPU_Unlock into nvidia driver, including possibility to create a merged one.

**_Support:_** [Join VGPU-Unlock discord for Support](https://discord.com/invite/5rQsSV3Byq)

This repository contains a submodule, so please clone this project recursively, i.e. using `git clone --recursive` command.

## Usage

1. Pre-download original `.run` files:
   | Name | Version | Links |
   | ----------- | ----------- | ----------- |
   | NVIDIA-GRID-Linux-KVM-525.85.07-525.85.05-528.24 | Grid v15.1 | [Nvidia Download](https://enterprise-support.nvidia.com/s/login/?startURL=%2Fs%2F%3Ft%3D1657093205198), [Google Drive](https://drive.google.com/drive/folders/1Mwk0diSegzHx-7BeJdujPa1Vgyw5fd3s) |

2. optionally edit `patch.sh` file to add support for your gpu if it was not included yet
   - search for "vcfgclone" lines, like for example:
        ```shell
        vcfgclone ${TARGET}/vgpuConfig.xml 0x1E30 0x12BA 0x1E84 0x0000
        ```
     the first two hex numbers select officially supported gpu as listed in vgpuConfig.xml (which can be extracted from vgpu kvm .run file)
     
     the example above is for Quadro RTX 6000 listed in the xml file as following:
        ```xml
        <pgpu><devId deviceId="0x1E30" subsystemId="0x12BA"/></pgpu>
        ```
     (fields that are not interesting for this example have been omitted)
     
   - the "vcfgclone" line example above is adding support for RTX 2070 Super, which has 10de:1e84 pci device id (that you can find for your gpu via `lspci -nn` command), so we are using the device id part, the last number 0x0000 is subdevice id, which may be used to differentiate some specific models, usually not needed, so we can use zero number there

   - just try to match the gpu architecture when adding a vcfgclone line, i.e. clone an officially supported pascal gpu if your gpu is pascal based

   - another example would be adding support for GTX 1080 Ti by cloning Tesla P40:
     Tesla P40 has 10de:1b38 pci device id and 10de:11d9 subsystem device id, listed in the xml as
        ```shell
        <pgpu><devId deviceId="0x1B38" subsystemId="0x0"/></pgpu>
        ```
        
        while the 1080 Ti can have 10de:1b06 pci devid with 10de:120f subsystem id for example, so the new vcfgclone line would have the first two numbers from the xml, the third number pci dev id of the card to be added and the fourth number can be zero or subsystem id (0x120f):
        ```shell
        vcfgclone ${TARGET}/vgpuConfig.xml 0x1B38 0x0 0x1B06 0x0000
        ```
        
   - when adding new vcfgclone lines, always refer into the xml to get the first two parameters and be sure to copy them case sensitively as the script searches for them as they are provided

3. Run one of these commands, depending on what you need:
      ```shell
      # a driver merged from vgpu-kvm with consumer driver (cuda and opengl for host too)
      ./patch.sh general-merge
      # display output on host not needed (proxmox) or you have secondary gpu
      ./patch.sh vgpu-kvm
      # driver for linux vm
      ./patch.sh grid
      # driver for linux vm functionally similar to grid one but using consumer .run as input
      ./patch.sh general
      # stuff for windows vm
      ./patch.sh wsys
      ```

## Changelog

- added `--docker-hack` option to fix `nvidia-docker-toolkit` not being able to detect the correct libraries
- `cudahost=1` nvidia module option of merged driver now works with all versions, enables also raytracing on host
- multiple fixes and tuning in the default profiles for rtx 2070+
- simplified patching focusing only on vgpu kvm blob, split the patch for merged driver extension
- supports setup of general-merge converting grid variant to general in case general run file is not available
- multiple versions in branches: 460.107, 470.141, 510.73, 510.85
- with 460.107 version ray tracing works on host with general-merge driver even with windows VMs

### Other Options 
`--lk6-patches` include compat patches for kernel versions >= 6.1
`--repack` option that can be used to create unlocked/patched `.run` file (usually not necessary as you can simply start nvidia-installer from the directory).
`--docker-hack` option to patch the driver version to make it compatible with the `nvidia-docker-toolkit` so vGPU functionality and Docker can be used simultaneously with a merged driver

### Credits
- Thanks to the discord user @mbuchel for the experimental patches
- Thanks to the discord user @LIL'pingu for the extended 43 crash fix
- Special thanks to @DualCoder without his work (vGPU_Unlock) we would not be here
- and thanks to the discord user @snowman for creating this patcher
