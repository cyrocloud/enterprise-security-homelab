# Enterprise Security Home Lab

This repository documents my enterprise cybersecurity home lab built on **Proxmox VE**. I use this environment to expand on technologies I work with professionally, experiment with security and infrastructure concepts in my own environment, and document practical lab configurations that others can use as ideas for building and learning in their own home labs.

The goal of this lab is to build and test technologies commonly used in enterprise environments while maintaining a segmented and isolated environment for security testing.

## Lab Infrastructure

My primary virtualization environment is built on a **Dell PowerEdge R630** running Proxmox VE.

The lab currently includes:

- Windows Server 2022 Domain Controller
- Windows 10 and Windows 11 endpoints
- pfSense Firewall
- Kali Linux
- Ubuntu Linux
- Wazuh
- Security Onion
- Qualys Vulnerability Scanner
- Horizon3.ai NodeZero
- TrueNAS SCALE
- Pi-hole
- Microsoft Defender for Endpoint
- Microsoft Entra Connect / Hybrid Identity

## Network Security & Segmentation

I configured pfSense to provide network segmentation for the lab, including a dedicated security testing VLAN.

Firewall rules prevent the security testing environment from communicating with trusted networks and other VLANs unless explicitly permitted.

This allows me to safely perform:

- Vulnerability scanning
- Penetration testing
- Malware testing
- Exploit testing
- Network traffic analysis
- Detection and response testing

I also configured an **Open vSwitch (OVS) bridge** within Proxmox for mirrored network traffic used by Security Onion for network security monitoring.

## Security Monitoring

The lab includes multiple security platforms for testing different detection and vulnerability-management capabilities.

**Wazuh**
- Endpoint monitoring
- Security event collection
- Detection testing

**Security Onion**
- Network security monitoring
- Mirrored traffic analysis
- Network-based threat detection

**Qualys**
- Vulnerability scanning
- Asset discovery
- Vulnerability assessment

**Horizon3.ai NodeZero**
- Autonomous penetration testing
- Attack-path discovery
- Network security validation

## Microsoft Security & Hybrid Identity

I also use the environment to test Microsoft enterprise security technologies.

This includes:

- Active Directory Domain Services
- Group Policy
- Microsoft Entra Connect
- Hybrid identity
- Microsoft Defender for Endpoint onboarding
- Defender for Endpoint Device Control
- USB device control policies
- Windows security configuration

## Storage

**TrueNAS SCALE** is deployed within the lab for network storage and SMB testing.

This allows me to gain hands-on experience with:

- SMB shares
- Permissions
- Network storage
- Storage access between systems

## VM Migration

I have also migrated virtual machines from other virtualization platforms into Proxmox.

This includes working with:

- OVA
- VMDK
- QCOW2
- VirtualBox
- Virt-Manager

Detailed migration procedures and lessons learned will be documented in this repository.

## Purpose

This lab is continuously evolving as I learn and test additional cybersecurity, cloud, networking, and infrastructure technologies.

The repository documents not only the final configurations, but also the troubleshooting, testing, validation, and lessons learned while building the environment.
