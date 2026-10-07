# pfSense Network Segmentation

## Overview

pfSense provides the primary **routing, segmentation, and policy-enforcement layer** for the security-testing portion of my Proxmox environment.

The lab contains workloads used for penetration testing, vulnerability assessment, honeypots, exploit validation, malware-related testing, and security monitoring. Because some of these systems intentionally generate or receive potentially hostile traffic, they are separated from trusted infrastructure rather than operating directly on the primary network.

The design establishes a dedicated security boundary where pfSense controls which networks and services security-testing workloads are permitted to reach.

The objective is to maintain a functional security research environment while minimizing the possibility of unintended communication with trusted systems.

---

# Architecture

At a high level, the environment follows this model:

```text
                         Internet
                            |
                            |
                    Upstream Network
                            |
                            |
                     Proxmox VE Host
                            |
                     +------+------+
                     |             |
                WAN / Uplink     pfSense
                                   |
                                   |
                           Security Interface
                                   |
                         Isolated Network
                                   |
             +---------------------+---------------------+
             |                     |                     |
             |                     |                     |
          Kali Linux          Windows Test VMs      Linux Workloads
             |                     |                     |
             +---------------------+---------------------+
                                   |
                            Security Testing
                                   |
                    +--------------+--------------+
                    |                             |
              Vulnerability                 Attack / Exploit
                 Testing                       Validation
```

The actual Proxmox environment uses multiple physical interfaces and virtual bridges to support separate traffic paths.

Sensitive addressing and unnecessary infrastructure identifiers are intentionally excluded from the public documentation.

---

# Why I Segmented the Security Environment

Some workloads within the lab are intentionally designed to behave differently from normal infrastructure.

Examples include:

- Kali Linux
- Vulnerability scanners
- Penetration-testing platforms
- Honeypots
- Test Windows endpoints
- Malware-related testing systems
- Exploit-validation systems
- Security research workloads

Allowing these systems unrestricted access to the same network as trusted infrastructure would create unnecessary risk.

Instead, I treat the security-testing environment as a **lower-trust security zone**.

The architecture assumes that a security-testing workload could become compromised or generate traffic that should not reach other portions of the environment.

---

# Trust Boundaries

The network is designed around explicit trust boundaries.

Conceptually:

```text
+------------------------------------------------------+
|                  TRUSTED ENVIRONMENT                 |
|                                                      |
|   Management     Infrastructure     Trusted Systems  |
|                                                      |
+--------------------------+---------------------------+
                           |
                       BLOCK / CONTROL
                           |
                    +------v------+
                    |   pfSense   |
                    |  Firewall   |
                    +------+------+
                           |
                    CONTROLLED ACCESS
                           |
+--------------------------v---------------------------+
|                SECURITY TEST ENVIRONMENT             |
|                                                      |
|   Kali     Test Windows     Linux     Security Tools |
|                                                      |
+------------------------------------------------------+
```

Traffic crossing these boundaries must be explicitly permitted when communication is required.

---

# Firewall Policy Strategy

The firewall policy follows a **deny-by-default approach between trust zones**.

Security workloads are permitted to communicate only where required for the scenario being tested.

A simplified policy model is:

| Source | Destination | Action | Purpose |
|---|---|---:|---|
| Security Network | Internet | Controlled Allow | Updates, testing, required external connectivity |
| Security Network | Security Network | Allow as required | Communication between authorized test workloads |
| Security Network | Approved Lab Service | Explicit Allow | Required infrastructure dependency |
| Security Network | Trusted Network | Block | Prevent lateral movement |
| Security Network | Management Network | Block | Protect management interfaces |
| Security Network | Other VLANs | Block by default | Maintain segmentation |

The exact rules may change depending on the testing scenario, but the trust model remains consistent.

---

# Rule Ordering

Firewall rule order is important because pfSense evaluates interface rules according to its processing behavior.

More specific requirements are evaluated before broader policies.

Conceptually:

```text
1. Allow explicitly required services
2. Allow explicitly required destinations
3. Block protected internal networks
4. Block management networks
5. Block other restricted network segments
6. Permit controlled Internet access where required
7. Deny traffic that does not match an authorized flow
```

This prevents a broad allow rule from unintentionally bypassing more restrictive security controls.

---

# Protecting the Management Plane

One of the most important boundaries in the environment is access to infrastructure management.

Security-testing systems should not require unrestricted access to management interfaces such as:

```text
Proxmox Management
pfSense Management
Switch Management
Storage Management
Hypervisor Services
Administrative Interfaces
```

Access from the security-testing network to management services is therefore restricted unless a specific administrative workflow requires it.

This helps prevent a compromised test workload from pivoting directly into the infrastructure used to control the lab itself.

---

# Preventing Inter-Network Leakage

A primary design requirement is preventing the isolated security environment from unintentionally communicating with other networks.

This includes preventing unnecessary access to:

- Trusted endpoints
- Administrative systems
- Infrastructure services
- Storage systems
- Other VLANs
- Hypervisor management
- Unrelated devices

Segmentation is enforced at the firewall rather than relying solely on endpoint firewalls.

This provides a network-level control that remains in place even if a guest operating system firewall is disabled, misconfigured, or intentionally bypassed during testing.

---

# Internet Access

Some security workloads require Internet connectivity for legitimate purposes such as:

- Operating system updates
- Security-tool updates
- Package repositories
- Vulnerability feeds
- Testing approved Internet-facing resources
- Downloading required software

Internet access can therefore be permitted while access to trusted internal networks remains restricted.

Conceptually:

```text
Security VM
    |
    v
 pfSense
    |
    +----------------X----------------> Trusted Networks
    |
    +----------------X----------------> Management
    |
    +----------------X----------------> Restricted VLANs
    |
    +---------------------------------> Internet
```

This creates an environment that remains usable for research while maintaining internal segmentation.

---

# Controlled Malware and Exploit Testing

The segmented environment has also been used for controlled malware-related and exploit testing.

These workloads are placed behind the pfSense security boundary so that potentially unsafe traffic is separated from trusted systems.

The isolation model is particularly important during these scenarios because endpoint-level controls alone are not considered sufficient containment.

Multiple layers are used:

```text
             Potentially Unsafe Workload
                       |
                       v
               Guest OS Controls
                       |
                       v
                Virtual Network
                       |
                       v
                   pfSense
                       |
                 Firewall Policy
                       |
             +---------+---------+
             |                   |
          BLOCKED             ALLOWED
             |                   |
       Trusted Networks     Required Traffic
```

This provides defense in depth if one individual security control does not behave as expected.

---

# Isolation Validation

Configuring firewall rules is only part of the implementation.

I also validate that the expected network boundaries are actually enforced.

A simplified validation matrix looks like:

| Test | Expected Result |
|---|---:|
| Security VM → Internet | PASS when required |
| Security VM → Authorized security service | PASS |
| Security VM → Trusted workstation | BLOCK |
| Security VM → Management network | BLOCK |
| Security VM → Restricted VLAN | BLOCK |
| Security VM → Proxmox management | BLOCK unless explicitly required |

Testing can include:

```text
ICMP connectivity testing
TCP connection testing
Port scanning
Firewall log review
Packet inspection
Application-level connectivity
Security monitoring telemetry
```

Testing both **allowed and denied traffic** is important.

Successfully reaching an authorized destination proves connectivity.

Failing to reach a protected destination provides evidence that the security boundary is functioning as intended.

---

# Proxmox Integration

pfSense operates as a virtualized firewall within the Proxmox environment.

Proxmox virtual network interfaces and bridges provide connectivity between pfSense and the appropriate network segments.

Conceptually:

```text
                  Physical Network
                         |
                         v
                    Proxmox NIC
                         |
                         v
                  Proxmox Bridge
                         |
                         v
                  pfSense Interface
                         |
                         v
                   Firewall Policy
                         |
                         v
                 Security Network
                         |
            +------------+------------+
            |            |            |
           Kali       Windows       Linux
```

Using separate virtual network paths allows workloads to be placed into the appropriate security zone without requiring every virtual machine to connect directly to the physical network.

---

# Relationship to Security Monitoring

Segmentation is also integrated with the broader monitoring architecture.

Security testing generates useful telemetry that can be analyzed by defensive security platforms.

The environment includes technologies such as:

- Security Onion
- Wazuh
- Qualys
- Horizon3.ai NodeZero

This creates a useful relationship between offensive activity and defensive visibility.

Conceptually:

```text
                   Security Testing
                         |
                         v
                       pfSense
                         |
              +----------+----------+
              |                     |
              v                     v
        Network Traffic        Firewall Logs
              |
              v
        Traffic Mirroring
              |
              v
        Security Monitoring
```

This allows security controls to be evaluated from both the **attack perspective and the detection perspective**.

---

# Security Onion and Traffic Visibility

The Proxmox environment also includes an **Open vSwitch (OVS)** implementation used for network traffic mirroring.

Selected network traffic can be copied to Security Onion for network security monitoring without requiring Security Onion to operate inline with the original traffic path.

This architecture separates two different responsibilities:

**pfSense**

```text
Routing
Segmentation
Firewall enforcement
Traffic control
```

**Security Onion**

```text
Network visibility
Traffic analysis
Detection
Investigation
```

This provides enforcement and monitoring as separate security layers.

---

# Security Design Principles

The pfSense implementation follows several principles.

## Default Deny Between Trust Zones

Communication should not exist simply because routing is technically possible.

Access between trust zones must have a defined requirement.

## Least-Required Connectivity

Security workloads receive the network access necessary for their function without automatically receiving access to unrelated systems.

## Protect the Management Plane

Infrastructure management interfaces are treated as higher-trust resources and separated from security-testing workloads.

## Defense in Depth

pfSense segmentation operates alongside endpoint controls, security monitoring, vulnerability management, and identity security.

## Validate Controls

A firewall rule being present does not prove that segmentation works.

Both permitted and denied communication paths should be tested.

## Assume Test Systems Can Become Untrusted

Security-testing workloads are designed with the assumption that they may intentionally or unintentionally execute hostile activity.

The surrounding architecture should therefore provide containment independently of the workload itself.

---

# Lessons Learned

One of the most important lessons from building the environment is that **network segmentation is not simply the creation of VLANs or additional interfaces**.

A meaningful security boundary requires several components working together:

```text
Network Separation
       +
Routing Architecture
       +
Firewall Policy
       +
Trust Classification
       +
Traffic Monitoring
       +
Validation Testing
       =
Effective Segmentation
```

A VLAN can separate broadcast domains, but the firewall policy determines whether systems in those networks are actually prevented from communicating.

For security-testing environments, validating the firewall behavior is therefore just as important as creating the network itself.

---

# Future Improvements

The architecture will continue to evolve as additional technologies and testing scenarios are introduced.

Potential improvements include:

- Additional segmentation between security workloads
- More granular egress filtering
- Expanded firewall logging
- Additional IDS/IPS integration
- Automated firewall-rule validation
- Additional network telemetry
- Expanded Security Onion monitoring
- Centralized firewall-log analysis
- Additional detection-validation scenarios

Changes will be evaluated based on the security requirements of the environment rather than adding complexity solely for the sake of complexity.

---

## Related Documentation

Additional documentation in this repository covers:

- Proxmox VE infrastructure
- Proxmox virtual networking
- Open vSwitch traffic mirroring
- Security Onion
- Wazuh
- Vulnerability management
- VM deployment and migration
- Microsoft security testing
- Hybrid identity
