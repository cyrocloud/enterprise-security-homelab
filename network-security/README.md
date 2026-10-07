# Network Security Architecture

## Overview

Network security within my Enterprise Security Home Lab is designed around **segmentation, isolation, controlled communication, and visibility**.

The environment includes systems used for vulnerability scanning, penetration testing, exploit validation, honeypots, malware-related testing, security monitoring, and other potentially untrusted workloads.

Rather than placing these systems directly on trusted network segments, I designed dedicated network paths and security boundaries using **pfSense, Proxmox virtual networking, VLANs, Linux bridges, and Open vSwitch (OVS)**.

The primary objective is to provide a controlled environment where security testing can be performed while reducing the possibility of unintended communication with trusted systems.

---

## Architecture Objectives

The network architecture was designed around several security requirements:

- Isolate higher-risk security-testing workloads
- Prevent unnecessary communication between network segments
- Control traffic through explicit firewall policy
- Provide Internet connectivity only where required
- Maintain a separate management path for infrastructure
- Capture selected network traffic for security monitoring
- Support vulnerability and penetration-testing platforms
- Validate firewall controls through testing
- Maintain flexibility for introducing additional security tools

Network placement is treated as part of the security architecture rather than simply a method of providing connectivity to a virtual machine.

---

## Network Security Architecture

At a high level, the environment follows the following model:

```text
                         Internet
                            |
                            |
                         pfSense
                            |
              +-------------+-------------+
              |                           |
        Trusted Networks            Security Network
                                          |
                                  Security Workloads
                                          |
                     +--------------------+--------------------+
                     |                    |                    |
                   Kali               Test VMs          Security Tools
                                                              |
                                                    Vulnerability /
                                                    Attack Testing
```

The exact implementation includes multiple Proxmox bridges and physical network interfaces supporting different traffic paths.

Sensitive addressing information and unnecessary infrastructure identifiers are intentionally excluded from the public documentation.

---

## pfSense Security Boundary

pfSense provides a primary security boundary for isolated testing workloads.

The firewall is responsible for controlling communication between the security-testing environment and other network segments.

Rather than allowing unrestricted routing between networks, firewall policies are designed around explicitly required communication.

Conceptually:

```text
Security Network
      |
      +------> Internet                Controlled / Allowed
      |
      +------> Required Lab Services   Explicitly Allowed
      |
      +------> Trusted Networks        Blocked
      |
      +------> Management Network      Blocked
      |
      +------> Other VLANs             Blocked unless required
```

This approach reduces the possibility of a compromised or intentionally malicious test system communicating with unrelated infrastructure.

---

## Isolated Security Testing

A dedicated portion of the environment is used for controlled security testing.

Workloads placed within this security boundary may include:

- Kali Linux
- Windows security-testing systems
- Ubuntu security workloads
- Vulnerability scanners
- Penetration-testing platforms
- Honeypots
- Malware-related test workloads
- Exploit-validation systems

Because these workloads may intentionally generate suspicious or malicious traffic, they are not treated with the same trust level as general infrastructure systems.

---

## Firewall Policy Design

Firewall policy is based on **least-required network access**.

The objective is not simply to determine what traffic should be blocked, but to identify what communication a workload actually requires and permit only those flows.

A simplified policy model looks like:

| Source | Destination | Policy |
|---|---|---|
| Security Network | Internet | Controlled Allow |
| Security Network | Required Security Services | Explicit Allow |
| Security Network | Trusted Network | Block |
| Security Network | Infrastructure Management | Block |
| Security Network | Other VLANs | Block by Default |
| Management Network | Infrastructure | Controlled Allow |

The actual rule set may vary depending on the security scenario being tested.

---

## Isolation Validation

Firewall configuration alone is not considered sufficient evidence that segmentation is functioning correctly.

After implementing firewall policy, I validate the expected communication paths.

Testing may include:

```text
Security VM -> Internet               EXPECTED: Allowed
Security VM -> Approved Lab Service   EXPECTED: Allowed

Security VM -> Trusted Endpoint       EXPECTED: Blocked
Security VM -> Management Interface   EXPECTED: Blocked
Security VM -> Restricted VLAN        EXPECTED: Blocked
```

Validation can include connectivity testing, packet inspection, firewall logging, port testing, and review of security-monitoring telemetry.

This provides evidence that the intended trust boundaries are being enforced.

---

## Proxmox Virtual Networking

Proxmox provides the virtualization layer supporting the network architecture.

The environment uses multiple virtual bridges rather than connecting every workload to a single virtual network.

Technologies used include:

- Linux Bridges
- Open vSwitch
- Physical NIC mappings
- VLAN-aware networking
- Dedicated virtual interfaces
- pfSense virtual interfaces

This allows virtual workloads to be connected to network segments according to their function and risk profile.

---

## Open vSwitch

**Open vSwitch (OVS)** is used where additional virtual networking capabilities are required.

One of the primary use cases within the environment is supporting network traffic visibility for security monitoring.

Selected traffic can be mirrored toward monitoring infrastructure without requiring the monitoring system to sit directly inline with the traffic path.

Conceptually:

```text
               Network Traffic
                      |
                      v
                OVS Switching
                 /          \
                /            \
               v              v
        Destination       Mirrored Copy
                               |
                               v
                        Security Monitoring
```

This provides a method of analyzing network activity without changing the original destination of the traffic.

---

## Security Onion Integration

Security Onion is used for network-focused security monitoring within the environment.

Mirrored traffic from selected portions of the lab can be presented to Security Onion through the virtual networking architecture.

This allows network activity generated during controlled testing to be analyzed from a defensive perspective.

The design provides an environment where I can generate activity from one portion of the lab while independently observing that activity through network security monitoring.

---

## Layered Security Visibility

The lab is designed to provide visibility from multiple security layers.

For example:

```text
                    Security Test
                         |
                         v
                       Network
                         |
              +----------+----------+
              |                     |
              v                     v
        Network Telemetry      Endpoint Telemetry
              |                     |
              v                     v
        Security Onion             Wazuh
```

Additional tools such as vulnerability scanners and security-validation platforms provide other perspectives into the same environment.

This allows findings to be compared across network, endpoint, vulnerability, and attack-validation technologies.

---

## Security Validation Scenarios

The isolated network architecture supports controlled testing scenarios such as:

- Vulnerability scanning
- Network reconnaissance
- Port scanning
- Exploit validation
- Detection testing
- Endpoint security testing
- Network intrusion detection
- Honeypot monitoring
- Attack-path analysis
- Security-control validation

Testing is performed within the boundaries established for the lab environment.

---

## Design Philosophy

The architecture follows several core principles.

### Segmentation

Systems with different purposes and risk profiles should not automatically share the same trust boundary.

### Least Privilege

Network access should be granted based on actual communication requirements.

### Isolation

Potentially hostile workloads should be separated from trusted infrastructure.

### Visibility

Security controls should provide sufficient telemetry to understand what occurs during testing.

### Validation

Security controls should be tested rather than assumed to function because they have been configured.

### Defense in Depth

Network controls operate alongside endpoint monitoring, vulnerability management, identity security, and other defensive technologies.

---

## Planned Documentation

Additional documentation within this section will provide deeper implementation details for:

- pfSense network segmentation
- Firewall policy architecture
- Security VLAN isolation
- Proxmox bridge architecture
- Open vSwitch configuration
- Traffic mirroring
- Security Onion integration
- Network isolation validation
- Controlled security-testing workflows

Where appropriate, diagrams and sanitized screenshots will be included to demonstrate configurations without exposing sensitive infrastructure information.

---

## Security Note

This repository documents a controlled personal lab environment.

IP addressing, credentials, authentication material, public endpoints, sensitive identifiers, and other unnecessary infrastructure details are intentionally excluded or sanitized.

Security-testing techniques documented within this repository are performed against systems and networks specifically configured and authorized for testing.
