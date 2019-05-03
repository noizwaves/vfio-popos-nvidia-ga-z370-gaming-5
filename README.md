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

1. Slow boot when `VT-d` is enabled
1. Slow shut down when VM has been started
1. High CPU usage of `system-udev?` when VM is running
1. High CPU usage of `system-udev?` when VM has been shut down

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

## Step X. Disable Nvidia drivers

1. Create file at `/etc/modprobe.d/blacklist-nvidia.conf` with contents:
    ```
    blacklist nvidia
    ```

## Step X. Add kernel parameters

1. `$ sudo kernelstub -a 'intel_iommu=on'` to do X
1. `$ sudo kernelstub -a 'iommu=pt'` to do X
1. `$ sudo kernelstub -a 'iommu=1'` to do X

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