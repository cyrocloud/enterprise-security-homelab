# Proxmox VE Infrastructure

## Overview

The foundation of my home lab is a Dell PowerEdge R630 running Proxmox VE.

I use Proxmox as the primary virtualization platform for hosting Windows, Linux, networking, storage, identity, vulnerability management, and security monitoring workloads.

The environment is designed to support both general infrastructure testing and isolated cybersecurity testing while keeping potentially unsafe workloads separated from trusted networks.

## Current Virtual Machines

The environment currently includes virtual machines for:

- Windows Server / Active Directory
- Windows 10
- Windows 11
- Kali Linux
- Ubuntu Linux
- pfSense
- Wazuh
- Security testing
- Qualys vulnerability scanning
- TrueNAS SCALE

Additional systems and security tools are added or removed as the lab evolves.

## Network Architecture

Proxmox uses multiple physical network interfaces and virtual bridges to separate different types of network traffic.

The environment uses both:

- Linux Bridges
- Open vSwitch (OVS)

pfSense provides firewalling and network segmentation for security testing workloads.

A dedicated isolated network is used for activities such as vulnerability scanning, exploit testing, malware testing, honeypots, and other security research.

Firewall rules are configured to prevent this environment from communicating with trusted networks unless communication is explicitly required.

Open vSwitch is also used for traffic mirroring to support network monitoring and analysis with Security Onion.

## Design Goals

The environment is designed around several principles:

- Network segmentation
- Workload isolation
- Least-privilege network access
- Security monitoring
- Safe security testing
- Flexibility for adding new technologies
- Reproducible lab configurations

## Documentation

Additional documentation in this section will cover:

- Proxmox networking and virtual bridges
- pfSense integration
- VLAN isolation
- Open vSwitch traffic mirroring
- Virtual machine deployment
- VM migration from OVA, VMDK, and QCOW2
- Security testing architecture
