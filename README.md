# Enterprise Security Home Lab

## Overview

This repository documents my **Enterprise Security Home Lab**, a continuously evolving environment built to design, implement, test, and validate infrastructure and cybersecurity solutions outside of production environments.

The lab extends technologies and architectural concepts I work with professionally while providing an independent environment for deeper testing, security validation, integration, troubleshooting, and research.

The environment is built primarily on **Proxmox VE** and incorporates network segmentation, hybrid identity, endpoint security, vulnerability management, network security monitoring, offensive security testing, DNS services, and network-attached storage.

A key objective of this repository is to document not only *what* was implemented, but also the **architecture, design decisions, security considerations, implementation methods, validation procedures, and lessons learned**. The documentation may also serve as a reference for others designing similar lab environments.

---

## Core Technologies

The environment currently incorporates technologies including:

- Proxmox VE
- pfSense
- Microsoft Active Directory Domain Services
- Microsoft Entra ID
- Microsoft Entra Connect
- Microsoft Defender for Endpoint
- Windows 10 / Windows 11
- Windows Server
- Ubuntu Linux
- Kali Linux
- Wazuh
- Security Onion
- Qualys
- Horizon3.ai NodeZero
- TrueNAS SCALE
- Pi-hole
- Open vSwitch (OVS)
- SMB

The environment continues to change as technologies, architectures, and security scenarios are evaluated.

---

## Infrastructure & Virtualization

The core virtualization platform is **Proxmox VE**, hosted on a Dell PowerEdge server.

Proxmox provides the compute, storage, and virtual networking foundation for the environment and hosts workloads supporting:

- Identity and directory services
- Windows endpoint testing
- Linux infrastructure
- Network security
- Security monitoring
- Vulnerability management
- Offensive security testing
- Network storage
- DNS services

Multiple virtual networking technologies are used to support segmentation and specialized security-monitoring requirements.

---

## Network Architecture & Segmentation

Network segmentation is a fundamental component of the environment.

**pfSense** provides firewalling, routing, and policy enforcement between selected network segments. Dedicated security-testing networks are isolated from trusted workloads through explicit firewall policies.

This architecture allows potentially hostile or untrusted activity to be generated and analyzed while limiting unnecessary communication with other network segments.

The isolated environment supports controlled activities such as:

- Vulnerability assessment
- Penetration testing
- Exploit validation
- Malware-related security testing
- Honeypot deployments
- Network traffic analysis
- Detection validation

Firewall policies are used to control communication between security-testing workloads, trusted systems, management networks, and other segments.

---

## Network Security Monitoring

The environment incorporates multiple layers of security visibility.

### Security Onion

Security Onion is used for network security monitoring and analysis of selected network traffic.

An **Open vSwitch (OVS)** implementation within the Proxmox networking architecture supports mirrored traffic delivery to monitoring workloads.

This provides visibility into network activity generated during controlled testing and enables analysis from a network-detection perspective.

### Wazuh

Wazuh provides centralized security monitoring and endpoint telemetry across selected systems.

The platform is used to evaluate host-level activity and detection capabilities alongside network-level monitoring.

Together, these technologies provide multiple perspectives into activity occurring throughout the environment.

---

## Vulnerability Management & Security Validation

The lab incorporates vulnerability assessment and autonomous security validation technologies.

### Qualys

Qualys is used for vulnerability scanning and assessment of selected systems within the environment.

This provides visibility into vulnerabilities, exposed services, configuration weaknesses, and remediation opportunities.

### Horizon3.ai NodeZero

NodeZero has been used to perform autonomous penetration testing and attack-path validation within the controlled lab environment.

This provides an additional method of evaluating whether identified weaknesses can be practically leveraged and how security controls respond to simulated attack activity.

---

## Hybrid Identity

The environment includes an Active Directory infrastructure integrated with Microsoft cloud identity services.

The architecture includes:

- Active Directory Domain Services
- DNS
- Group Policy
- Microsoft Entra Connect
- Microsoft Entra ID
- Hybrid identity scenarios

This environment provides a platform for validating identity architecture, authentication behavior, directory synchronization, endpoint integration, and security controls across on-premises and cloud identity systems.

---

## Microsoft Endpoint Security

Microsoft Defender for Endpoint is integrated with selected Windows systems within the environment.

Testing and validation have included:

- Defender for Endpoint onboarding
- Endpoint security configuration
- Microsoft Defender policy deployment
- Group Policy integration
- Device Control
- USB/removable-storage policy enforcement
- Device identification and policy validation

These scenarios provide an environment for evaluating endpoint controls and validating expected behavior without introducing changes into production systems.

---

## Security Testing Environment

Dedicated virtual machines are maintained for controlled security testing.

These workloads include platforms such as:

- Kali Linux
- Windows test endpoints
- Ubuntu Linux
- Security monitoring systems
- Vulnerability scanners
- Honeypot/security research workloads

Potentially unsafe testing is performed within segmented networks protected by pfSense policies designed to prevent unintended communication with trusted systems.

The objective is to maintain a controlled environment where security controls can be tested against realistic traffic and attack scenarios while preserving network isolation.

---

## Storage Architecture

**TrueNAS SCALE** provides network storage services within the lab.

The implementation includes SMB-based file services and provides an environment for testing:

- Network file sharing
- SMB
- Permissions and access controls
- Storage connectivity
- Cross-platform file access
- Network segmentation involving storage services

---

## DNS Services

**Pi-hole** is deployed as part of the lab's DNS architecture.

The service provides DNS-based filtering and an additional source of visibility into DNS activity generated by systems within selected portions of the environment.

---

## Virtual Machine Migration

The environment also includes virtual machines migrated from other virtualization platforms into Proxmox.

Migration work has included:

- OVA
- VMDK
- QCOW2
- VirtualBox
- Virt-Manager

The repository will document the migration process, disk import procedures, storage considerations, boot configuration, troubleshooting, and post-migration validation.

---

## Architecture Principles

The environment is designed around several core principles:

- **Segmentation** — Separate workloads based on function and risk.
- **Isolation** — Restrict security-testing workloads from trusted environments.
- **Least Privilege** — Permit network communication only where required.
- **Defense in Depth** — Combine endpoint, network, identity, and vulnerability-management controls.
- **Visibility** — Collect telemetry from multiple layers of the environment.
- **Validation** — Verify that security controls behave as designed.
- **Repeatability** — Document configurations and procedures so implementations can be reproduced.
- **Controlled Testing** — Evaluate security technologies without introducing unnecessary risk to production environments.

---

## Repository Documentation

This repository will contain detailed documentation covering areas such as:

- Proxmox architecture and configuration
- Virtual networking and bridges
- pfSense network segmentation
- Firewall policy design
- Security VLAN architecture
- Open vSwitch traffic mirroring
- Security Onion
- Wazuh
- Vulnerability management
- Qualys
- NodeZero security validation
- Active Directory
- Microsoft Entra hybrid identity
- Microsoft Defender for Endpoint
- Device Control and USB security
- TrueNAS SCALE and SMB
- Pi-hole
- VM migrations
- Troubleshooting and validation

Each section focuses on the **architecture, implementation, security considerations, testing methodology, and technical decisions** behind the configuration.

---

## Purpose

This lab serves as an independent engineering environment for designing, testing, and validating cybersecurity, cloud, networking, identity, and infrastructure solutions outside of production environments.

The repository documents architecture decisions, configurations, security controls, troubleshooting, testing methodologies, and lessons learned while implementing and evaluating technologies across the environment.

It is also intended to provide practical examples and architectural ideas for engineers and security professionals building their own lab environments.

---
