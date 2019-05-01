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

## Step X. Disable `nouveau` drivers

1. Create file at `/etc/modprobe.d/blacklist-nouveau.conf` with contents:
    ```
    blacklist nouveau
    options nouveau modeset=0
    ```

## Step X. Add kernel parameters

1. `$ sudo kernelstub -a 'intel_iommu=on'` to do X
1. `$ sudo kernelstub -a 'iommu=pt'` to do X
1. `$ sudo kernelstub -a 'iommu=1'` to do X

## Step X. Look up device PCI IDs

### Nvidia VGA and Audio devices

Look up GPU IDs by running `$ lspci -nn | grep -i nvidia`. The output will look something like:

    ```
    TODO: Sample output here.
    ```

The IDs are `X` and `Y`.

### USB Controller

Look up ID by running `$ lspci -nn | grep -i nvidia`. The output will look something like:

    ```
    TODO: Sample output here.
    ```

The ID is `Z`.

## Step X. Enable the VFIO kernel module

`$ sudo echo 'vfio-pci' > /etc/modules-load.d/vfio-pci.conf`

## Step X. Configure VFIO

`$ sudo echo 'options vfio-pci ids=X,Y,Z' > /etc/modprobe.d/vfio.conf`

## Step X. Regenerate initramfs

`$ sudo update-initramfs -u`

*Note*: this should be run after changing any configuration (`/etc/modprobe.d/`, `/etc/modules-load/`) or kernel parameters.

## Step X. Reboot

`$ sudo reboot -h now`