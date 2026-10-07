# Security Monitoring Architecture

## Overview

Security monitoring within my Enterprise Security Home Lab is designed to provide visibility across both **endpoint and network activity**.

Rather than relying on a single security platform, the environment uses multiple telemetry sources and monitoring technologies to evaluate activity from different perspectives.

The primary platforms currently used for security monitoring include:

- **Wazuh** for host-based security monitoring and endpoint telemetry
- **Security Onion** for network security monitoring and traffic analysis

These platforms complement other security technologies within the environment, including pfSense, vulnerability scanners, penetration-testing platforms, and Microsoft Defender for Endpoint.

The objective is to create an environment where security activity can be **generated, observed, correlated, investigated, and validated** across multiple security layers.

---

## Monitoring Architecture

At a high level, the monitoring architecture follows this model:

```text
                    Security Activity
                           |
             +-------------+-------------+
             |                           |
             v                           v
       Endpoint Activity           Network Activity
             |                           |
             v                           v
           Wazuh                   Network Traffic
                                         |
                                         v
                                  OVS Traffic Mirror
                                         |
                                         v
                                  Security Onion
```

This provides visibility into the same environment from different perspectives.

Wazuh focuses primarily on activity occurring **within monitored systems**, while Security Onion provides visibility into activity occurring **across the network**.

---

## Defense-in-Depth Visibility

The monitoring architecture is based on the principle that no single telemetry source provides complete visibility.

For example, an activity may generate:

```text
Endpoint Event
      |
      +----> Process activity
      +----> Authentication event
      +----> File modification
      +----> Security alert

Network Event
      |
      +----> Connection attempt
      +----> DNS request
      +----> Port scan
      +----> Suspicious protocol activity
```

Combining endpoint and network monitoring provides additional context during investigation and security-control validation.

---

# Wazuh

Wazuh is deployed within the Proxmox environment as a centralized security-monitoring platform.

Selected endpoints and systems report security telemetry to Wazuh for analysis.

The platform provides visibility into host-level activity and allows security events from multiple systems to be reviewed centrally.

Wazuh is used within the lab for scenarios including:

- Endpoint monitoring
- Security-event collection
- Authentication monitoring
- File and system activity
- Security configuration visibility
- Detection testing
- Alert investigation
- Security-control validation

---

## Endpoint Monitoring Model

Conceptually:

```text
Windows Endpoint --------+
                         |
Linux Endpoint ----------+----> Wazuh
                         |
Server ------------------+
                         |
Security Test VM --------+
```

This centralizes security telemetry that would otherwise remain distributed across individual systems.

---

# Security Onion

Security Onion provides the **network-security monitoring layer** within the environment.

Rather than installing an agent on every system to obtain network visibility, Security Onion analyzes selected network traffic presented to its monitoring interface.

The Proxmox environment uses **Open vSwitch (OVS)** to support traffic mirroring.

Conceptually:

```text
                  Original Network Traffic
                           |
                           v
                         OVS
                        /   \
                       /     \
                      v       v
               Destination   Mirrored Copy
                                  |
                                  v
                           Security Onion
```

The original traffic continues toward its intended destination while a copy is provided to the monitoring environment.

This allows Security Onion to observe selected network activity without operating directly inline with the original communication path.

---

## Monitoring vs Enforcement

The environment separates **security enforcement** from **security monitoring**.

For example:

### pfSense

Responsible for:

- Routing
- Network segmentation
- Firewall policy
- Traffic enforcement
- Trust-boundary control

### Security Onion

Responsible for:

- Network visibility
- Traffic analysis
- Network detection
- Investigation
- Security telemetry

### Wazuh

Responsible for:

- Host visibility
- Endpoint telemetry
- Security events
- Host-level detection
- Centralized event analysis

This separation allows individual security controls to perform specialized functions while contributing to the broader defensive architecture.

---

# Controlled Detection Testing

One advantage of maintaining an isolated security-testing environment is the ability to intentionally generate activity and observe how monitoring technologies respond.

Testing may include controlled activities such as:

- Network reconnaissance
- Port scanning
- Authentication testing
- Vulnerability scanning
- Exploit validation
- Suspicious network traffic
- Endpoint security testing
- Honeypot activity
- Security configuration changes

These activities provide known events that can be compared against the telemetry produced by monitoring platforms.

---

## Detection Validation

The objective is not simply to install a monitoring platform and assume that it provides visibility.

Security controls should be validated.

A simplified workflow is:

```text
Generate Controlled Activity
           |
           v
   Capture Telemetry
           |
           v
    Generate Detection
           |
           v
     Review Alert
           |
           v
  Validate Information
           |
           v
Identify Visibility Gaps
           |
           v
   Adjust Configuration
```

This approach helps determine whether monitoring systems provide the expected visibility during a security event.

---

# Offensive and Defensive Integration

The lab allows offensive and defensive security technologies to operate within the same controlled environment.

For example:

```text
               Kali / Security Tool
                       |
                       |
                Generate Activity
                       |
          +------------+------------+
          |                         |
          v                         v
     Target Endpoint             Network
          |                         |
          v                         v
        Wazuh                  Security Onion
          |                         |
          +------------+------------+
                       |
                       v
                Detection Review
```

This makes it possible to evaluate an activity from both the attack and defensive perspectives.

---

# Vulnerability Management Integration

Security monitoring is also complemented by vulnerability-management and security-validation technologies.

The environment has included:

- Qualys
- Horizon3.ai NodeZero

These platforms answer different questions than Wazuh or Security Onion.

Conceptually:

```text
Qualys
   |
   +----> What vulnerabilities exist?

NodeZero
   |
   +----> Can identified weaknesses contribute to an attack path?

Wazuh
   |
   +----> What occurred on the endpoint?

Security Onion
   |
   +----> What occurred across the network?
```

Using different security technologies provides a broader understanding of the environment than relying on a single platform.

---

# Monitoring the Security Testing Environment

The isolated security environment provides a controlled source of potentially suspicious activity.

Security tools can intentionally generate activity while monitoring platforms observe the results.

For example:

```text
Kali Linux
    |
    | Network Scan
    v
Test Endpoint
    |
    +------------------> Endpoint Telemetry ----> Wazuh
    |
    +------------------> Network Traffic -------> Security Onion
```

This creates a repeatable method for validating monitoring coverage.

---

# Security Monitoring Principles

The monitoring architecture follows several principles.

## Visibility Before Assumption

A security tool being installed does not automatically mean that useful telemetry is being collected.

Telemetry and detections should be validated.

## Multiple Data Sources

Endpoint and network telemetry provide different perspectives and should complement each other.

## Controlled Testing

Known activity provides a baseline for determining whether security controls respond as expected.

## Segmentation

Monitoring infrastructure should be designed without weakening the security boundaries established elsewhere in the environment.

## Detection Validation

Alerts should be tested against expected behavior rather than assumed to function based solely on configuration.

## Continuous Improvement

Monitoring configurations evolve as new systems, attack scenarios, and security technologies are introduced.

---

# Current Security Monitoring Technologies

| Technology | Primary Function |
|---|---|
| Wazuh | Endpoint and host security monitoring |
| Security Onion | Network security monitoring |
| Open vSwitch | Traffic mirroring |
| pfSense | Firewalling and network policy enforcement |
| Qualys | Vulnerability assessment |
| NodeZero | Security and attack-path validation |
| Microsoft Defender for Endpoint | Endpoint protection and detection |

Each technology addresses a different component of the broader security architecture.

---

# Planned Documentation

Additional documentation in this section will cover:

- Wazuh architecture and deployment
- Wazuh endpoint onboarding
- Wazuh monitoring and detection testing
- Security Onion deployment
- Open vSwitch traffic mirroring
- Security Onion traffic validation
- Controlled detection testing
- Security monitoring lessons learned

Detailed documentation will focus on **architecture, implementation, validation, and troubleshooting** rather than simply documenting product installation.

---

## Security Note

All security testing documented in this repository is performed within systems and networks that I own or am explicitly authorized to test.

Sensitive information, credentials, public addressing, and unnecessary infrastructure identifiers are excluded or sanitized from public documentation.
