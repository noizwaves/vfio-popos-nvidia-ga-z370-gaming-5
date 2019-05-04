# VFIO w/ Nvidia, Pop!_OS and Gigabyte Z370 Gaming 5

Instructions for VFIO based Nvidia GPU passthrough on Pop!_OS and Gigabyte Z370 Gaming 5 motherboard.

VFIO will be loaded from kernel modules, and configured through configuration files.

## Goals

Passthrough to a Pop!_OS guest of:
- [x] Nvidia GPU
- [x] USB controller (for audio, keyboard, mouse)
- [ ] SATA controller (for a SSD)

VM optimisations of:
- [ ] Pinned CPU cores
- [ ] Static memory allocation

## Known Issues

## Slow boot when `VT-d` is enabled

## Slow shut down when VM has been started

## High CPU usage of `system-udev?` when VM is running

## High CPU usage of `system-udev?` when VM has been shut down

### Current Fix 

run `$ sudo service udev restart`

## Excessive logging to `dmesg`

From `nvidia-nvlink` and `NVRM`, example:

```
[  530.504375] nvidia-nvlink: Nvlink Core is being initialized, major device number 234
[  530.504692] NVRM: The NVIDIA probe routine was not called for 1 device(s).
[  530.504693] NVRM: This can occur when a driver such as: 
            NVRM: nouveau, rivafb, nvidiafb or rivatv 
            NVRM: was loaded and obtained ownership of the NVIDIA device(s).
[  530.504693] NVRM: Try unloading the conflicting kernel module (and/or
            NVRM: reconfigure your kernel without the conflicting
            NVRM: driver(s)), then try loading the NVIDIA kernel module
            NVRM: again.
[  530.504694] NVRM: No NVIDIA graphics adapter probed!
[  530.525816] nvidia-nvlink: Unregistered the Nvlink Core, major device number 234
[  530.630306] nvidia-nvlink: Nvlink Core is being initialized, major device number 234
[  530.630470] NVRM: The NVIDIA probe routine was not called for 1 device(s).
[  530.630470] NVRM: This can occur when a driver such as: 
            NVRM: nouveau, rivafb, nvidiafb or rivatv 
            NVRM: was loaded and obtained ownership of the NVIDIA device(s).
[  530.630470] NVRM: Try unloading the conflicting kernel module (and/or
            NVRM: reconfigure your kernel without the conflicting
            NVRM: driver(s)), then try loading the NVIDIA kernel module
            NVRM: again.
[  530.630471] NVRM: No NVIDIA graphics adapter probed!
[  530.649892] nvidia-nvlink: Unregistered the Nvlink Core, major device number 234
[  530.748590] nvidia-nvlink: Nvlink Core is being initialized, major device number 234
[  530.748905] NVRM: The NVIDIA probe routine was not called for 1 device(s).
[  530.748905] NVRM: This can occur when a driver such as: 
            NVRM: nouveau, rivafb, nvidiafb or rivatv 
            NVRM: was loaded and obtained ownership of the NVIDIA device(s).
[  530.748906] NVRM: Try unloading the conflicting kernel module (and/or
            NVRM: reconfigure your kernel without the conflicting
            NVRM: driver(s)), then try loading the NVIDIA kernel module
            NVRM: again.
[  530.748906] NVRM: No NVIDIA graphics adapter probed!
[  530.777793] nvidia-nvlink: Unregistered the Nvlink Core, major device number 234
[  530.864642] nvidia-nvlink: Nvlink Core is being initialized, major device number 234
[  530.864955] NVRM: The NVIDIA probe routine was not called for 1 device(s).
[  530.864956] NVRM: This can occur when a driver such as: 
            NVRM: nouveau, rivafb, nvidiafb or rivatv 
            NVRM: was loaded and obtained ownership of the NVIDIA device(s).
[  530.864956] NVRM: Try unloading the conflicting kernel module (and/or
            NVRM: reconfigure your kernel without the conflicting
            NVRM: driver(s)), then try loading the NVIDIA kernel module
            NVRM: again.
[  530.864957] NVRM: No NVIDIA graphics adapter probed!
```

## Useful resources

The following guides were helpful in getting this all to work:
- https://blog.zerosector.io/2018/07/28/kvm-qemu-windows-10-gpu-passthrough/

## Specifications

### Software

- Pop_OS! 19.04 (w/ Nvidia drivers)

### Hardware

- CPU: Intel 8700K
- Motherboard: Gigabyte Z370 Gaming 5
- GPU: MSI GeForce GTX 1080 8GB

## Step X. BIOS Configurations

1. Start computer and press DEL at BIOS screen to enter BIOS settings
1. Enable `VT-d Virtualization` under Chipset settings
1. Save and exit

## Step X. Install VFIO / Qemu / KVM / Virtual Machine Manager

`$ sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager ovmf`

## Step X. Add kernel parameters

1. `$ sudo kernelstub -a 'intel_iommu=on'` to do ?
1. `$ sudo kernelstub -a 'iommu=pt'` to do ?
1. `$ sudo kernelstub -a 'iommu=1'` to do ?
1. `$ sudo kernelstub -a "pci=noaer"` to fix an issue that prevented the VM from starting when the USB Controller was attached

Notes:
- to view current kernel boot options, run `$ sudo kernelstub -p`
- to remove a kernel boot option, run `$ sudo kernelstub -d 'key=value'`

## Step X. Look up device PCI IDs

### Nvidia VGA and Audio devices

Look up GPU IDs by running `$ lspci -nn | grep -i nvidia`. The output will look something like:

    ```
    01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP104 [GeForce GTX 1080] [10de:1b80] (rev a1)
    01:00.1 Audio device [0403]: NVIDIA Corporation GP104 High Definition Audio Controller [10de:10f0] (rev a1)
    ```

The IDs are `10de:1b80` and `10de:10f0`.

### USB Controller

Look up ID by running `$ lspci -nn | grep -i nvidia`. The output will look something like:

    ```
    07:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM2142 USB 3.1 Host Controller [1b21:2142]
    ```

The ID is `1b21:2142`.

## Step X. Enable the VFIO kernel module

`$ sudo echo 'vfio-pci' > /etc/modules-load.d/vfio-pci.conf`

## Step X. Configure VFIO

Tell VFIO which devices to run for

`$ sudo echo 'options vfio-pci ids=10de:1b80,10de:10f0,1b21:2142' > /etc/modprobe.d/vfio.conf`

Force VFIO drivers to load before the Nvidia drivers by

```
sudo echo 'softdep nvidia pre: vfio vfio_pci' >> /etc/modprobe.d/vfio.conf
sudo echo 'softdep nvidia_418 pre: vfio vfio_pci' >> /etc/modprobe.d/vfio.conf
sudo echo 'softdep nvidia_drm pre: vfio vfio_pci' >> /etc/modprobe.d/vfio.conf
```

## Step X. Regenerate initramfs

`$ sudo update-initramfs -u`

*Note*: this should be run after changing any configuration (`/etc/modprobe.d/`, `/etc/modules-load/`) or kernel parameters.

## Step X. Reboot

`$ sudo reboot -h now`

Plug in a display into the onboard HDMI port.

Wait several minutes with a black boot screen... and then log into Pop!_OS.

## Step X. Confirm IOMMU is working

QUICK! After logging in run `$ dmesg | grep -e DMAR -e IOMMU` and see if there is any messages.

If this is run too late, the `dmesg` will be flooded with `nvidia` log junk.

## Step X. Confirm vfio-pci drivers are being used

On the GPU VGA:
```
$ lspci -nnk -d 10de:1b80
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP104 [GeForce GTX 1080] [10de:1b80] (rev a1)
	Subsystem: Micro-Star International Co., Ltd. [MSI] GP104 [GeForce GTX 1080] [1462:3363]
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau, nvidia_drm, nvidia
```

On the GPU audio:
```
$ lspci -nnk -d 10de:10f0
01:00.1 Audio device [0403]: NVIDIA Corporation GP104 High Definition Audio Controller [10de:10f0] (rev a1)
	Subsystem: Micro-Star International Co., Ltd. [MSI] GP104 High Definition Audio Controller [1462:3363]
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```

On the USB Controller, it doesn't seem to show the `vfio-pci` driver, but this output results in a successful passthrough:
```
$ lspci -nnk -d 1b21:2142
07:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM2142 USB 3.1 Host Controller [1b21:2142]
	Subsystem: Gigabyte Technology Co., Ltd ASM2142 USB 3.1 Host Controller [1458:5007]
	Kernel driver in use: xhci_hcd
```

## Troubleshooting

If your host fails to boot, try:
1. disable `VT-d` in the bios and reboot