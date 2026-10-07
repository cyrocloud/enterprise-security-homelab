# Deploying Windows and Ubuntu Virtual Machines on Proxmox VE

## Overview

This document outlines the process and design considerations I use when deploying new **Windows and Ubuntu virtual machines on Proxmox VE**.

The objective is to create virtual machines using appropriate virtual hardware, firmware, storage, networking, guest drivers, and management components rather than relying exclusively on default settings.

The general deployment workflow is:

```text
Obtain Installation Media
        |
        v
Upload ISO to Proxmox
        |
        v
Create Virtual Machine
        |
        v
Configure Virtual Hardware
        |
        v
Attach Installation Media
        |
        v
Install Operating System
        |
        v
Install Guest Drivers / Agent
        |
        v
Patch and Secure
        |
        v
Validate VM
```

---

# Installation Media

## Uploading ISO Files to Proxmox

Operating system installation media can be uploaded directly through the Proxmox web interface.

Navigate to the Proxmox storage location configured to store ISO images.

For example:

```text
Datacenter
└── Proxmox Node
    └── local
        └── ISO Images
```

Select:

```text
Upload
```

Then select the appropriate ISO.

Examples include:

```text
ubuntu-24.04.x-live-server-amd64.iso
Windows11.iso
virtio-win.iso
```

ISOs can then be attached to virtual machines as virtual CD/DVD devices.

---

# Ubuntu VM Deployment

## Create the VM

Select:

```text
Create VM
```

Assign an appropriate:

- VM ID
- VM name
- Resource configuration
- Network
- Storage location

Attach the Ubuntu ISO during creation.

---

## Firmware

Ubuntu supports both traditional BIOS and UEFI configurations.

For modern deployments, UEFI can be used with:

```text
BIOS: OVMF (UEFI)
Machine: q35
```

An EFI disk should also be configured when OVMF is selected.

The firmware configuration should ultimately match the requirements of the workload rather than being selected solely because it is newer.

---

## CPU

CPU allocation should be based on the expected workload.

A reasonable starting point for a general-purpose Ubuntu server may be:

```text
Sockets: 1
Cores: 2
```

Additional resources can be allocated as workload requirements increase.

For many lab workloads, CPU type can also be configured as:

```text
host
```

This allows the VM to use CPU features exposed by the physical Proxmox host.

Compatibility requirements should be considered if VM migration between hosts is expected.

---

## Memory

Memory should similarly be sized according to workload requirements.

For a lightweight Ubuntu infrastructure server:

```text
2-4 GB RAM
```

may be sufficient.

Security applications, databases, scanners, SIEM components, containers, or other resource-intensive applications may require substantially more memory.

---

## Storage

For modern Linux guests, I generally prefer VirtIO-based storage where appropriate.

A typical configuration may use:

```text
SCSI Controller: VirtIO SCSI
Disk Bus: SCSI
```

This provides efficient virtualized storage while maintaining flexibility for Proxmox features.

Disk size should be determined by the workload rather than using the same size for every VM.

---

## Networking

For network interfaces, VirtIO is generally preferred:

```text
Model: VirtIO (paravirtualized)
```

The VM should then be connected to the appropriate Proxmox bridge.

For example:

```text
vmbr0
vmbr1
vmbr2
```

The selected bridge depends on the security zone and purpose of the workload.

A security-testing VM should not automatically be placed on the same network as trusted infrastructure.

---

# Ubuntu Installation

Start the VM and open the console.

Complete the Ubuntu installation normally.

Depending on the server's purpose, configuration may include:

- Hostname
- User account
- Storage layout
- Network configuration
- OpenSSH Server
- Package selection

After installation completes, remove or detach the installation ISO if necessary and confirm the VM boots from its virtual disk.

---

# Ubuntu Post-Installation

Update the operating system:

```bash
sudo apt update
sudo apt upgrade -y
```

Reboot if required:

```bash
sudo reboot
```

---

# QEMU Guest Agent

The QEMU Guest Agent improves communication between Proxmox and the guest operating system.

For Ubuntu:

```bash
sudo apt install qemu-guest-agent -y
```

Enable and start the service:

```bash
sudo systemctl enable --now qemu-guest-agent
```

The guest agent must also be enabled within the Proxmox VM configuration.

Navigate to:

```text
VM
└── Options
    └── QEMU Guest Agent
```

Enable the feature.

The agent can provide Proxmox with additional guest information and improve VM management operations.

---

# Windows 11 VM Deployment

Windows 11 requires additional consideration because Proxmox commonly uses paravirtualized devices that are not included in standard Windows installation media.

A typical Windows 11 deployment uses:

```text
Machine: q35
BIOS: OVMF (UEFI)
EFI Disk: Enabled
TPM: 2.0
Disk Controller: VirtIO SCSI
Network Adapter: VirtIO
```

---

# Windows 11 Installation Media

For Windows deployment, I attach two ISO images to the VM.

### Windows ISO

The first CD/DVD device contains:

```text
Windows11.iso
```

### VirtIO ISO

The second contains:

```text
virtio-win.iso
```

The VirtIO ISO provides Windows drivers for Proxmox/KVM virtual hardware.

The resulting VM configuration conceptually looks like:

```text
Windows 11 VM
│
├── Hard Disk
│   └── VirtIO/SCSI
│
├── CD/DVD Drive 1
│   └── Windows 11 ISO
│
├── CD/DVD Drive 2
│   └── VirtIO Driver ISO
│
├── Network Device
│   └── VirtIO
│
├── EFI Disk
│
└── TPM State
    └── TPM 2.0
```

---

# Why Windows Setup May Not See the Disk

A common issue occurs when Windows Setup reaches:

```text
Where do you want to install Windows?
```

and no disks appear.

This does **not necessarily mean that the virtual disk is missing or incorrectly configured**.

If the VM is using a VirtIO-based storage controller, Windows Setup may simply not have the required storage driver.

The VirtIO driver must therefore be provided to Windows Setup.

---

# Loading the VirtIO Storage Driver

At the Windows disk selection screen, select:

```text
Load driver
```

Browse the VirtIO CD/DVD drive.

The exact directory depends on the VirtIO configuration and driver version.

For a VirtIO SCSI configuration, the required storage driver is commonly located under a directory corresponding to the VirtIO SCSI driver and the Windows version/architecture.

For example, the VirtIO ISO may contain a path similar to:

```text
vioscsi
└── w11
    └── amd64
```

Load the appropriate driver.

After the driver loads, return to the disk selection screen.

The Proxmox virtual disk should now become visible.

Select the disk and continue the Windows installation.

---

# Alternative Driver Strategy

Another approach is to initially deploy Windows using virtual hardware for which Windows already contains native drivers and transition to VirtIO afterward.

This can be useful in certain migration or troubleshooting scenarios.

However, Windows must have the appropriate VirtIO storage driver installed **before changing the boot disk to a VirtIO controller**.

Otherwise, Windows may be unable to access its boot disk after the controller is changed.

For clean Windows installations, attaching the VirtIO ISO during Windows Setup is generally a straightforward approach.

---

# Installing Remaining VirtIO Drivers

Loading the storage driver during Windows Setup only solves the immediate storage-controller requirement.

After Windows boots, mount/open the VirtIO ISO from inside Windows.

The VirtIO guest tools installer can then be used to install the remaining applicable drivers and guest components.

This may include drivers for:

- Network interfaces
- Storage devices
- Memory ballooning
- Serial devices
- Additional VirtIO devices
- QEMU guest functionality

After installation, reboot Windows if required.

---

# Network Driver

Another common Windows installation issue is the network adapter not appearing.

If the VM uses:

```text
VirtIO Network Device
```

Windows may require the VirtIO network driver.

The driver can be loaded from the VirtIO ISO or installed after Windows installation as part of the VirtIO guest tools package.

After installation, verify that Windows recognizes the network adapter.

---

# Windows 11 TPM Configuration

Windows 11 expects TPM 2.0 for supported deployments.

Within Proxmox, add:

```text
Hardware
└── Add
    └── TPM State
```

Configure:

```text
Version: 2.0
```

The VM should also use UEFI where appropriate:

```text
BIOS: OVMF
Machine: q35
EFI Disk: Enabled
```

This creates a virtual hardware configuration that more closely aligns with modern Windows 11 platform requirements.

---

# Windows QEMU Guest Agent

After Windows installation, install the appropriate QEMU guest components through the VirtIO guest tools package.

Then enable the QEMU Guest Agent within Proxmox:

```text
VM
└── Options
    └── QEMU Guest Agent
```

This improves integration between Proxmox and Windows.

---

# Windows Post-Deployment

After the initial installation, I validate and configure the operating system before using the VM for its intended workload.

Typical tasks include:

- Install Windows updates
- Install VirtIO drivers
- Install/validate QEMU Guest Agent
- Verify Device Manager
- Verify storage
- Verify network connectivity
- Validate DNS
- Confirm system time
- Configure Windows Firewall
- Apply required security controls
- Rename the system where required
- Join Active Directory or Microsoft Entra ID where applicable
- Onboard to Microsoft Defender for Endpoint where applicable

Device Manager should also be reviewed for unknown devices or missing drivers.

---

# VM Network Placement

Network placement is treated as a security decision rather than simply a connectivity decision.

Before attaching a VM to a bridge or VLAN, I determine what the system needs to communicate with and what systems should be prevented from reaching it.

For example:

```text
Trusted Infrastructure VM
        |
        +---- Trusted/Internal Network

Security Testing VM
        |
        +---- Isolated Security Network
                   |
                pfSense
                   |
             Firewall Policy
```

A newly deployed security-testing workload does not automatically receive access to trusted infrastructure.

---

# Deployment Validation

Before considering a VM deployment complete, I validate:

- VM boots correctly
- CPU and memory allocation are appropriate
- Operating system recognizes the virtual disk
- Network adapter is operational
- Correct bridge/VLAN is assigned
- DNS works
- Required services start
- Operating system is patched
- Required VirtIO drivers are installed
- QEMU Guest Agent is operational
- No unexpected devices appear in Device Manager
- Firewall configuration is appropriate
- Security tooling is operational where required
- The VM cannot communicate with networks that it should not be able to access

---

# Design Considerations

The VM creation wizard is only the beginning of a deployment.

The virtual hardware configuration should reflect the purpose and security requirements of the workload.

Key design decisions include:

**Firmware**

UEFI/OVMF versus legacy BIOS.

**Machine Type**

Modern workloads may benefit from Q35 depending on compatibility requirements.

**Storage**

The disk bus and storage controller affect performance, compatibility, and driver requirements.

**Networking**

Bridge and VLAN selection determine the VM's network trust boundary.

**Drivers**

Windows requires additional consideration when VirtIO devices are used.

**Management**

QEMU Guest Agent improves guest/hypervisor integration.

**Security**

TPM, Secure Boot requirements, firewall policy, network segmentation, patching, and endpoint controls should be considered as part of the VM deployment rather than afterthoughts.

---

## Summary

A properly deployed Proxmox VM is more than an operating system installed on a virtual disk.

The deployment includes the complete virtual hardware and security architecture:

```text
Operating System
      +
Virtual CPU / Memory
      +
Firmware
      +
Storage Controller
      +
VirtIO Drivers
      +
Virtual Networking
      +
Network Segmentation
      +
Guest Agent
      +
Security Controls
      =
Validated Virtual Machine
```

Standardizing these decisions makes VM deployments more predictable, easier to troubleshoot, and easier to integrate into the broader lab architecture.
