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

Other features
- [ ] Remove `Tablet` input from VM

## Known Issues

## Slow boot when `VT-d` is enabled

## Slow shut down when VM has been started

## High CPU usage of `system-udev?` when VM is running

## High CPU usage of `system-udev?` when VM has been shut down

### Current Fix 

Run `$ sudo service udev restart`, or if that does not work, then `$ sudo service udev stop` and `$ sudo service udev start`.

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

Force VFIO drivers to load before the USB drivers by
```
sudo echo 'softdep xhci_hcd pre: vfio vfio_pci' >> /etc/modprobe.d/vfio.conf
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

## Step X. Create initial VM

1. Navigate to https://system76.com/pop and download the `Pop!_OS 19.04 (with Nvidia)`
1. Open Virtual Machine Manager
1. Create new VM by clicking the `New VM` icon or via `File > New Virtual Machine`

### Step 1 of 5

1. Ensure `Local instlal media (ISO image or CDROM)` is selected
1. Click `Forward`

### Step 2 of 5

1. Click `Browse` button, then click `Browse Local`
1. Navigate to `~/Downloads` (or wherever you downloaded the Pop!_OS iso to)
1. Select the file (at time of writing this was `pop-os_19.04_amd64_nvidia_4.iso`) and click `Open`
1. Uncheck `Automatically detect from the installation media / source`
1. In the text input above (placeholder text of  `Type to start searching...`) type `ubuntu`
1. Select `Ubuntu 19.04 (ubuntu19.04)` from the list that appears
1. Click `Forward`

### Step 3 of 5

1. Increase memory to `8192`
1. Increase CPUs to `2`
1. Click `Forward`

### Step 4 of 5

1. Increase disk image size from `15.0` to `50.0`
1. Click `Forward`

### Step 5 of 5

1. Check `Customize configuration before install`
1. Click `Finish`

## Step X. Customize the VM in GUI

### Overview

1. The `Overview` tab should be selected (if not, select `Overview` from the side bar)
1. Ensure the `Chipset` is set to Q35
1. Change the `Firmware` to `UEFI x86_64: /usr/share/OVMF/OVMF_CODE.fd`
  - CAUTION: take care not to select the `OVMF_CODE.secboot.fd` or `OVMF_CODE.ms.fd` options

### Disk type

1. In the side bar, select `VirtIO Disk 1`
1. Ensure the `Advanced options` is expanded
1. Change the `Disk bus` to `SATA`
1. Click `Apply`

### Boot Options

1. In the side bar, select `Boot Options`
1. Ensure `SATA CDROM 1` is checked
1. With `SATA CDROM 1` selected, press the up button to the right, this will move `SATA CDROM 1` above `DATA Disk 1`
1. Click `Apply`

### Display Spice

1. In the side bar, select `Display Spice`
1. Click `Remove`
1. Click `Yes`

### Video QXL

1. In the side bar, select `Video QXL`
1. Click `Remove`
1. Click `Yes`

### Channel spice

1. In the side bar, select `Channel spice`
1. Click `Remove`
1. Click `Yes`

### Sound ich9

1. In the side bar, select `Sound ich9`
1. Click `Remove`
1. Click `Yes`

### Add GPU (VGA)

1. Below the side bar, click `Add Hardware`
1. Select `PCI Host Device`
1. Select the Nvidia GPU `0000:01:00:0 NVIDIA Corporation GP104 [GeForce GTX 1080]`
1. Click `Finish`

### Add GPU (Audio)

1. Below the side bar, click `Add Hardware`
1. Select `PCI Host Device`
1. Select the Nvidia GPU `0000:01:00:1 NVIDIA Corporation GP104 High Definition Audio Controller`
1. Click `Finish`

### Add USB Controller

1. Below the side bar, click `Add Hardware`
1. Select `PCI Host Device`
1. Select the Nvidia GPU `0000:07:00:0 ASMedia Technology Inc.`
1. Click `Finish`

### Finish the installation (it will fail, that's OK)

1. Click `Begin Installation` to start the VM
1. Focus on the `ubuntu19.04 on QEMU/KVM` window
1. Power off the VM via `Virtual Machine > Force Off`

## Step X. Customize the VM XML

1. In a new terminal, run `$ sudo virsh edit ubuntu19.04` to open a VI session to edit the VM's XML
1. Scroll down to the `domain.features` block (approximately line 23 at time of writing)
1. Insert the following lines to configure KVM and Hyperv:
    ```
    <hyperv>
      <vendor_id state='on' value='whatever'/>
    </hyperv>
    <kvm>
      <hidden state='on'/>
    </kvm>
    ```
1. Save the changes

## Step X. Restore initial boot settings

1. Navigate to the VM details by via `View > Details`
1. Click on `SATA CDROM 1` and ensure the Pop!_OS iso is still selected
1. Click on `Boot Options` and ensure `SATA CDROM 1` is checked and above `SATA Disk 1`

## Step X. Booting the VM and install OS

1. Plug a monitor into the GPU (either HDMI or DP ports)
1. Plug a keyboard and mouse into the Red USB 3.1 DAC port or the USB Type C port (your choice of connections and USB hubs)
1. Power on the VM via `Virtual Machine > Run`
1. Install Pop!_OS onto the VM
1. When asked if you want to shutdown or restart, choose `Shutdown`

## Step X. Change boot settings to disk

1. On the host, navigate to the VM details via `View > Details`
1. On the `Boot Options` tab, uncheck `SATA CDROM 1`, click `Apply`
1. On the `SATA CDROM 1` tab, clear the `Source Path`, click `Apply`

## Step X. Booting the VM for final guest configuration

1. Power on the VM via `Virtual Machine > Run`
1. Complete the installation

## Step X. Use the guest VM

Install Steam, play games, etc etc :)

## Troubleshooting

If your host fails to boot, try:
1. disable `VT-d` in the bios and reboot
