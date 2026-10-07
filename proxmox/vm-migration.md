# Migrating OVA, VMDK, and QCOW2 Virtual Machines to Proxmox VE

## Overview

This document covers the process I have used to migrate existing virtual machines from other virtualization platforms into **Proxmox VE**.

Rather than rebuilding an existing workload, virtual disks can be transferred to the Proxmox host, imported into Proxmox storage, attached to a destination VM, and validated before the migrated workload is placed into service.

The process is applicable to common virtualization formats such as:

- OVA
- VMDK
- QCOW2
- VirtualBox exports
- Virt-Manager / KVM virtual disks

> **Important:** VM configuration varies between operating systems and source hypervisors. CPU type, firmware, storage controller, networking, and boot configuration should be reviewed during each migration.

---

## Migration Workflow

At a high level, the migration process is:

```text
Source Hypervisor
      |
      v
Export VM / Virtual Disk
      |
      v
Transfer to Proxmox
      |
      v
Extract OVA (if applicable)
      |
      v
Import Virtual Disk
      |
      v
Attach Disk to Destination VM
      |
      v
Configure Boot Order
      |
      v
Boot and Validate
```

---

## 1. Create the Destination VM

Create a new virtual machine in Proxmox that will act as the destination for the imported workload.

Configure the appropriate:

- VM ID
- CPU
- Memory
- Network interface
- BIOS/UEFI configuration
- Machine type

An operating system installation ISO is not required because the existing virtual disk will be imported.

If a temporary disk is created during the VM creation process, it can be detached and removed before attaching the imported disk.

---

## 2. Transfer the VM Files to Proxmox

Transfer the exported VM or virtual disk to the Proxmox host.

I have used an SFTP client such as **FileZilla** to transfer exported VM files to the Proxmox server.

After transferring the files, connect to the Proxmox shell and verify that the files are present.

```bash
ls -lah
```

---

## 3. Extract an OVA

An OVA is an archive that typically contains an OVF configuration file, one or more virtual disks, and a manifest.

Extract the OVA using:

```bash
tar -xvf ubuntu-vm.ova
```

Then inspect the extracted files:

```bash
ls -lah
```

An extracted OVA may contain files similar to:

```text
ubuntu-vm.ovf
ubuntu-vm.mf
ubuntu-vm-disk001.vmdk
```

The `.vmdk` file is the virtual disk that will typically be imported into Proxmox.

The `.mf` file is a manifest containing file integrity/checksum information and is **not** the virtual disk.

---

## 4. Import the Virtual Disk

Proxmox provides the `qm importdisk` command for importing an existing virtual disk into a VM.

General syntax:

```bash
qm importdisk <VMID> <DISK_FILE> <STORAGE>
```

Example:

```bash
qm importdisk 300 ubuntu-vm-disk001.vmdk local-lvm
```

Where:

```text
300                         Destination Proxmox VM ID
ubuntu-vm-disk001.vmdk      Source virtual disk
local-lvm                   Destination Proxmox storage
```

The VM ID, disk filename, and destination storage should be adjusted for the environment.

---

## 5. Attach the Imported Disk

After the import completes, return to the Proxmox web interface.

Navigate to:

```text
VM
└── Hardware
    └── Unused Disk
```

The imported disk should appear as an **Unused Disk**.

Select the disk and attach it to the VM.

Review the disk configuration and storage controller before completing the attachment.

---

## 6. Configure the Boot Order

Attaching the disk does not necessarily make it the primary boot device.

Navigate to:

```text
VM
└── Options
    └── Boot Order
```

Enable the imported disk as a bootable device and place it in the appropriate position in the boot sequence.

---

## 7. Start the VM

Start the virtual machine and open the Proxmox console.

Confirm that the operating system successfully boots from the imported disk.

A successful disk import does not guarantee that the guest operating system will boot correctly. Differences between the source and destination hypervisors can require additional configuration.

---

## 8. Post-Migration Validation

After the VM boots, validate the workload before considering the migration complete.

Validation should include:

- Operating system boots successfully
- Expected disks are available
- Network connectivity functions correctly
- IP configuration is correct
- DNS resolution works
- Required services start successfully
- Applications operate as expected
- System time is correct
- No unexpected driver errors are present
- Proxmox/QEMU Guest Agent functionality is reviewed
- Security controls remain operational

---

## Common Migration Considerations

### BIOS vs. UEFI

The destination VM firmware should be compatible with how the source operating system was installed.

A VM originally installed using UEFI may not boot correctly if the destination VM is configured for legacy BIOS, and vice versa.

### Storage Controller

The source operating system may not contain drivers for the storage controller selected in Proxmox.

Controller selection should therefore be validated before changing to an optimized configuration such as VirtIO SCSI.

### Network Interfaces

The migrated operating system may detect the Proxmox virtual NIC as a new network adapter.

Static IP configurations may need to be reviewed or reapplied.

### Windows Guests

Windows workloads may require VirtIO drivers depending on the selected virtual hardware.

### Linux Guests

Linux workloads may require interface or network configuration changes if the virtual NIC name changes after migration.

---

## Security Considerations

Imported virtual machines should be treated as untrusted until their configuration and security posture have been validated.

Depending on the workload, validation may include:

- Patching the operating system
- Reviewing local accounts
- Rotating credentials
- Reviewing SSH keys
- Validating firewall configuration
- Reviewing exposed services
- Running vulnerability scans
- Confirming endpoint security controls
- Placing the VM into an isolated network during initial testing

In my environment, workloads that require additional validation can be placed behind the segmented pfSense security environment before being allowed to communicate with other lab systems.

---

## Lessons Learned

A successful VM migration involves more than importing a disk.

The virtual disk contains the workload, but the destination hypervisor still needs to provide compatible virtual hardware, firmware, networking, storage controllers, and boot configuration.

Separating the process into **transfer, import, attachment, boot configuration, and validation** makes migrations easier to troubleshoot and reduces the risk of changing multiple variables simultaneously.
