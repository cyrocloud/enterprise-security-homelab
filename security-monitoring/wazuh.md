# Wazuh Security Monitoring

## Overview

Wazuh is deployed within my Proxmox-based security lab as a centralized platform for **endpoint telemetry, security monitoring, detection, and investigation**.

The implementation provides host-level visibility across selected Windows and Linux systems and complements the network-level visibility provided by Security Onion.

Rather than treating the deployment as simply a SIEM installation, I use the environment to evaluate the complete monitoring workflow:

**endpoint activity → telemetry collection → detection → investigation → validation**

Because the environment also contains isolated security-testing workloads, I can intentionally generate known activity and compare that activity against the telemetry and detections produced by Wazuh.

---

# Architecture

At a high level, the Wazuh implementation follows this model:

```text
                         Proxmox VE
                             |
          +------------------+------------------+
          |                  |                  |
          v                  v                  v
     Windows VM          Linux VM         Security/Test VM
          |                  |                  |
          |                  |                  |
          +------------- Wazuh Agents ----------+
                             |
                             v
                      Wazuh Platform
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
           Events         Alerts        Investigation
```

Selected workloads report endpoint security telemetry to the centralized Wazuh environment.

This provides visibility into activity occurring directly within the operating systems rather than relying exclusively on network traffic.

---

# Role Within the Security Architecture

Wazuh is one component of a broader layered security architecture.

The different technologies within the environment perform separate functions.

```text
                        Security Activity
                               |
              +----------------+----------------+
              |                                 |
              v                                 v
       Endpoint Activity                 Network Activity
              |                                 |
              v                                 v
            Wazuh                        Security Onion
              |                                 |
              +----------------+----------------+
                               |
                               v
                       Security Analysis
```

This separation is intentional.

A network-monitoring platform may observe communication between systems, while endpoint telemetry may provide additional information about what occurred inside the host.

---

# Endpoint Monitoring

Wazuh agents can be deployed to selected endpoints that require host-level monitoring.

Systems within the lab may include:

- Windows Server
- Windows 10
- Windows 11
- Ubuntu Linux
- Infrastructure workloads
- Security-testing systems

The exact systems monitored can change as workloads are added or removed from the environment.

---

# Windows Monitoring

Windows systems provide an important telemetry source because many of the lab scenarios involve Windows infrastructure, identity, endpoint security, and attack simulation.

Depending on the configuration and testing scenario, useful Windows telemetry can include:

- Authentication activity
- Security events
- Account activity
- System events
- Service activity
- File changes
- Security configuration changes
- Endpoint-security events

Centralizing this information makes it easier to analyze activity across multiple Windows systems rather than reviewing each endpoint independently.

---

# Linux Monitoring

Linux systems are also present throughout the environment and support infrastructure, security tooling, and testing workloads.

Wazuh provides centralized visibility into selected Linux systems for activity such as:

- Authentication events
- System events
- Service activity
- Configuration changes
- File activity
- Security events
- Operating-system activity

This provides a consistent monitoring layer across heterogeneous operating systems.

---

# Detection Validation

One of the primary objectives of the Wazuh deployment is **detection validation**.

Installing an agent and receiving events confirms connectivity, but it does not necessarily prove that the security-monitoring implementation provides useful detection coverage.

I use controlled activity within the lab to evaluate whether expected telemetry and alerts are generated.

The workflow is:

```text
Generate Known Activity
          |
          v
Endpoint Generates Events
          |
          v
Wazuh Agent Collects Telemetry
          |
          v
Wazuh Processes Activity
          |
          v
Detection / Alert
          |
          v
Review Evidence
          |
          v
Validate Expected Behavior
```

This allows the monitoring configuration to be tested rather than assumed to function correctly.

---

# Controlled Security Testing

The isolated security-testing portion of the lab provides a controlled environment for generating activity that would be inappropriate to perform against production systems.

Examples of activity that may be used for validation include:

- Authentication attempts
- Network reconnaissance
- Port scanning
- Vulnerability scanning
- Security configuration changes
- Endpoint policy testing
- Exploit-related testing
- Suspicious process activity
- File modifications
- Administrative activity

Testing is performed only against systems within the controlled lab environment.

---

# Attack and Detection Workflow

The environment allows security testing and defensive monitoring to operate together.

A simplified example:

```text
                      Kali Linux
                          |
                          |
                  Controlled Activity
                          |
                          v
                   Windows Test VM
                          |
                 +--------+--------+
                 |                 |
                 v                 v
          Endpoint Events     Network Traffic
                 |                 |
                 v                 v
               Wazuh         Security Onion
                 |                 |
                 +--------+--------+
                          |
                          v
                   Detection Review
```

This provides two different perspectives into the same activity.

---

# Endpoint vs. Network Visibility

Wazuh and Security Onion are not intended to provide identical telemetry.

Their value comes from observing different layers.

### Wazuh

Provides visibility from the endpoint perspective:

```text
User Activity
Processes
Authentication
Operating System Events
Configuration Changes
File Activity
Host Security Events
```

### Security Onion

Provides visibility from the network perspective:

```text
Connections
Network Protocols
Traffic Patterns
Network Reconnaissance
Suspicious Communications
Network Detection
```

An investigation can therefore contain information that would not necessarily be visible from only one monitoring layer.

---

# Example Validation Scenario

A simplified validation scenario might involve performing controlled reconnaissance against a monitored system.

```text
Kali Linux
    |
    | Controlled Scan
    v
Windows / Linux Target
    |
    +----------> Endpoint Telemetry
    |                    |
    |                    v
    |                  Wazuh
    |
    +----------> Network Traffic
                         |
                         v
                   Security Onion
```

The resulting telemetry can then be reviewed to determine:

1. What activity was visible on the endpoint?
2. What activity was visible on the network?
3. Was an alert generated?
4. Did the alert provide enough context?
5. Was additional telemetry required?
6. Did the network segmentation behave as expected?

This turns the lab into a security-control validation environment rather than simply a collection of installed tools.

---

# Wazuh and Vulnerability Management

Wazuh exists alongside dedicated vulnerability-management technologies within the environment.

These technologies serve different purposes.

```text
Qualys
   |
   +----> Identify vulnerabilities and exposure

NodeZero
   |
   +----> Validate attack paths and exploitable conditions

Wazuh
   |
   +----> Observe endpoint activity and security events

Security Onion
   |
   +----> Observe network activity
```

The combination provides multiple perspectives into security posture and activity.

---

# Wazuh and Microsoft Defender for Endpoint

Microsoft Defender for Endpoint is also used with selected Windows workloads within the broader lab.

Wazuh and Defender for Endpoint provide an opportunity to compare security visibility across different endpoint-security technologies.

The objective is not necessarily to make the platforms perform identical functions.

Instead, the environment can be used to evaluate:

- Endpoint telemetry
- Security detections
- Device visibility
- Security-control behavior
- Investigation context
- Differences in detection coverage

This becomes particularly useful when generating controlled security events against monitored endpoints.

---

# Network Segmentation

Wazuh monitoring operates within the security boundaries established elsewhere in the lab.

Endpoints should not receive additional network privileges simply because they require security monitoring.

Required communication between monitored endpoints and Wazuh should be permitted deliberately while unrelated communication remains restricted.

Conceptually:

```text
Monitored Endpoint
       |
       | Required Monitoring Traffic
       v
     Wazuh

Monitored Endpoint
       |
       X
       |
Unauthorized Network
```

Monitoring requirements should not weaken existing network segmentation.

---

# Monitoring Security Infrastructure

Security platforms themselves should also be treated as infrastructure requiring protection.

Administrative access to Wazuh should be separated from potentially hostile workloads wherever practical.

This is particularly important in a lab where some systems may intentionally execute malicious or exploit-related activity.

The monitoring platform should remain available even when the systems being observed are considered untrusted.

---

# Validation Methodology

When evaluating monitoring coverage, I use a simple methodology:

## 1. Establish Expected Behavior

Determine what telemetry or detection should be produced by the activity.

## 2. Generate Controlled Activity

Perform the test against an authorized lab system.

## 3. Review Endpoint Telemetry

Determine what Wazuh observed.

## 4. Review Network Telemetry

Where applicable, compare the activity against Security Onion or other network telemetry.

## 5. Review Security Controls

Determine whether endpoint protection, firewalling, or other controls affected the test.

## 6. Identify Visibility Gaps

Determine whether important information was missing.

## 7. Adjust and Retest

Modify monitoring or security configuration where appropriate and repeat the test.

The process can be represented as:

```text
Design Test
    |
    v
Generate Activity
    |
    v
Collect Evidence
    |
    v
Analyze Detection
    |
    v
Identify Gap
    |
    v
Adjust Configuration
    |
    v
Retest
```

---

# Operational Validation

Beyond generating security events, I also validate the health of the monitoring implementation itself.

Areas to verify include:

- Wazuh platform availability
- Agent connectivity
- Expected endpoints reporting
- Event ingestion
- Alert generation
- Time synchronization
- DNS resolution
- Network connectivity
- Required firewall communication
- Storage utilization
- Service health

A security-monitoring platform that is installed but not reliably receiving telemetry creates a false sense of visibility.

Operational health is therefore part of the security validation process.

---

# Troubleshooting Approach

When expected telemetry is missing, I troubleshoot the monitoring path in layers.

```text
Endpoint
   |
   v
Agent
   |
   v
Network Connectivity
   |
   v
Firewall / Segmentation
   |
   v
Wazuh Platform
   |
   v
Event Processing
   |
   v
Detection / Alert
```

This helps determine whether the issue exists at the endpoint, agent, network, ingestion, or detection layer.

Breaking the workflow into layers also avoids changing multiple components simultaneously during troubleshooting.

---

# Security Considerations

Several security principles apply to the Wazuh implementation.

### Least Privilege

Monitoring requirements should not automatically provide broad network access.

### Segmentation

Security-monitoring infrastructure should remain appropriately separated from potentially hostile workloads.

### Administrative Protection

Access to the monitoring platform should be limited to authorized administrative systems and users.

### Telemetry Integrity

The monitoring architecture should provide sufficient confidence that expected endpoints are reporting and that missing telemetry can be identified.

### Time Synchronization

Accurate timestamps are important when comparing endpoint and network activity during an investigation.

### Defense in Depth

Wazuh complements rather than replaces firewalling, endpoint protection, vulnerability management, identity security, and network monitoring.

---

# Lessons Learned

A monitoring platform becomes significantly more useful when it is tested against known activity.

Simply seeing endpoints appear in a dashboard proves that the agents can communicate with the platform.

It does not necessarily prove that the environment can detect or provide sufficient visibility into a security event.

The more meaningful validation cycle is:

```text
Known Activity
      +
Expected Telemetry
      +
Actual Telemetry
      +
Detection Review
      +
Cross-Platform Validation
      =
Monitoring Confidence
```

Maintaining both offensive/security-testing systems and defensive monitoring systems within the same isolated lab makes it possible to repeatedly perform this validation.

---

# Future Improvements

The Wazuh implementation will continue to evolve alongside the broader security environment.

Areas for additional development include:

- Additional endpoint coverage
- Expanded detection testing
- Custom detection rules
- Additional log sources
- Detection tuning
- Automated testing
- Alert enrichment
- Additional correlation between endpoint and network activity
- Expanded security-control validation
- Additional attack simulation scenarios

Changes will be introduced based on monitoring requirements and identified visibility gaps rather than simply enabling additional features.

---

## Related Documentation

- Security Monitoring Architecture
- Network Security Architecture
- pfSense Network Segmentation
- Security Onion
- Open vSwitch Traffic Mirroring
- Vulnerability Management
- Microsoft Defender for Endpoint
