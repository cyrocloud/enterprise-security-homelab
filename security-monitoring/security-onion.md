# Security Onion Network Security Monitoring

## Overview

Security Onion is deployed within my Proxmox-based security lab to provide **network security monitoring, traffic analysis, detection, and investigation capabilities**.

The implementation complements host-based monitoring technologies such as Wazuh and Microsoft Defender for Endpoint by providing visibility from the network perspective.

A key component of the design is the use of **Open vSwitch (OVS)** within Proxmox to support traffic mirroring. Selected network traffic can be copied to the Security Onion monitoring interface while the original traffic continues toward its intended destination.

This allows Security Onion to observe network activity without becoming the primary routing or firewall enforcement point.

The architecture separates three important functions:

- **pfSense** — routing, segmentation, and firewall enforcement
- **Open vSwitch** — virtual switching and traffic mirroring
- **Security Onion** — network monitoring, detection, and analysis

---

# Architecture

At a high level, the monitoring path follows this model:

```text id="d9t4sr"
                    Network Activity
                           |
                           v
                    Proxmox Networking
                           |
                           v
                     Open vSwitch
                      /         \
                     /           \
                    v             v
           Original Traffic    Mirrored Copy
                    |             |
                    v             v
              Destination    Security Onion
                                  |
                                  v
                         Network Monitoring
                                  |
                    +-------------+-------------+
                    |             |             |
                    v             v             v
                 Detection     Analysis     Investigation
```

The original communication path remains operational while Security Onion receives a copy of selected traffic for analysis.

---

# Why Network Security Monitoring

Endpoint telemetry provides detailed visibility into activity occurring inside a host, but it does not provide a complete picture of network communication.

Network security monitoring provides another perspective.

Examples of activity that may be visible at the network layer include:

- Network reconnaissance
- Port scanning
- Connection attempts
- Protocol activity
- DNS communication
- Suspicious traffic patterns
- Communication between systems
- Potential command-and-control behavior
- Exploit-related traffic
- Unexpected services
- Lateral-movement activity

Combining network and endpoint visibility provides additional context during security investigations.

---

# Separation of Responsibilities

An important design decision is keeping **network enforcement and network monitoring as separate responsibilities**.

## pfSense

pfSense is responsible for:

- Routing
- Firewall policy
- Network segmentation
- Trust-boundary enforcement
- Controlling permitted communication

## Open vSwitch

OVS provides:

- Virtual switching
- Connectivity within the Proxmox networking architecture
- Traffic-mirroring capabilities
- Delivery of selected traffic to monitoring interfaces

## Security Onion

Security Onion provides:

- Network visibility
- Network security monitoring
- Detection
- Traffic analysis
- Investigation capabilities

Conceptually:

```text id="d32lvd"
                    NETWORK TRAFFIC
                          |
                          v
                      [pfSense]
                          |
                   Policy Enforcement
                          |
                          v
                         OVS
                      /       \
                     /         \
                    v           v
             Destination    Mirror Traffic
                                |
                                v
                         [Security Onion]
                                |
                                v
                        Detection / Analysis
```

This separation allows the firewall to enforce policy independently of the monitoring platform.

---

# Proxmox Integration

Security Onion operates as a virtualized workload within the Proxmox environment.

The Proxmox host uses multiple networking constructs to support different traffic requirements, including:

- Physical network interfaces
- Linux bridges
- Open vSwitch
- VLAN-aware networking
- Dedicated virtual network interfaces

Security Onion requires a network path that allows it to receive the traffic selected for monitoring.

Rather than placing all workloads directly onto the same virtual bridge, network interfaces are assigned according to their function and trust requirements.

---

# Open vSwitch Traffic Mirroring

One of the key components of the implementation is **Open vSwitch traffic mirroring**.

Without traffic mirroring, a Security Onion VM connected to a normal virtual network interface would not automatically receive traffic exchanged between unrelated systems.

The monitoring architecture therefore creates a copy of selected network traffic and delivers that copy to the Security Onion monitoring interface.

Conceptually:

```text id="l2j2lo"
             VM-A -----------------------> VM-B
               \                           /
                \                         /
                 +------ OVS ------------+
                          |
                          |
                     Mirror Copy
                          |
                          v
                  Security Onion
```

Security Onion receives the mirrored copy without changing the original destination of the communication.

---

# Monitoring Interface

The Security Onion monitoring interface serves a different purpose from a normal management interface.

Conceptually:

```text id="wrx9dc"
Security Onion
│
├── Management Interface
│      |
│      +---- Administrative access
│      +---- Updates
│      +---- Management traffic
│
└── Monitoring Interface
       |
       +---- Mirrored traffic
       +---- Packet observation
       +---- Network telemetry
```

Separating management traffic from monitoring traffic provides a cleaner architecture and reduces unnecessary interaction between the monitoring plane and administrative plane.

The monitoring interface is intended primarily to **observe traffic**, not act as the default gateway for monitored workloads.

---

# Passive Monitoring

The architecture allows Security Onion to function primarily as a **passive monitoring platform**.

Conceptually:

```text id="j9qrki"
Attacker/Test VM
       |
       |
       v
Target System
       |
       +-----------------------> Normal Communication
       |
       +-----------------------> Mirrored Copy
                                      |
                                      v
                               Security Onion
```

Security Onion does not need to sit directly between the systems to observe selected communication.

This reduces the risk of the monitoring platform becoming an unnecessary dependency in the primary communication path.

---

# Controlled Detection Testing

The isolated security-testing environment allows known network activity to be generated intentionally.

Examples may include:

- Port scanning
- Service enumeration
- Vulnerability scanning
- Network reconnaissance
- Exploit-related testing
- Authentication testing
- Suspicious protocol activity
- Honeypot interaction
- Attack simulation

Because the activity is intentionally generated, the expected behavior is known before reviewing the resulting telemetry.

---

# Validation Workflow

The monitoring configuration is validated using a repeatable workflow.

```text id="9t5qya"
Generate Controlled Traffic
           |
           v
Traffic Crosses Monitored Path
           |
           v
OVS Mirrors Traffic
           |
           v
Security Onion Receives Traffic
           |
           v
Telemetry / Detection Generated
           |
           v
Review Evidence
           |
           v
Validate Expected Visibility
```

This provides a way to verify the complete monitoring path rather than simply confirming that Security Onion is powered on.

---

# Validating Traffic Visibility

One of the most important implementation checks is determining whether the monitoring interface is actually receiving the expected traffic.

The validation process can be considered in layers:

```text id="2r5qk5"
Traffic Generated
      |
      v
Correct Network Path
      |
      v
OVS Receives Traffic
      |
      v
Mirror Configuration
      |
      v
Security Onion Interface
      |
      v
Packet Visibility
      |
      v
Security Telemetry
```

If expected telemetry is missing, each layer can be validated independently.

This makes troubleshooting significantly more effective than immediately changing detection rules.

---

# Detection vs. Visibility

An important distinction is that **seeing traffic and generating an alert are not the same thing**.

A successful monitoring implementation should first establish whether the traffic is visible.

The workflow is:

```text id="npk6yd"
Did the activity occur?
        |
        v
Did Security Onion receive the traffic?
        |
        v
Was useful telemetry generated?
        |
        v
Was a detection expected?
        |
        v
Did the detection trigger?
```

If the traffic never reaches the monitoring interface, tuning a detection rule will not solve the underlying problem.

This distinction is particularly useful when troubleshooting network-monitoring systems.

---

# Security Onion and Wazuh

Security Onion and Wazuh provide different perspectives into the same security activity.

For example:

```text id="4i3avc"
                      Kali Linux
                          |
                    Port Scan
                          |
                          v
                   Windows Target
                     /         \
                    /           \
                   v             v
            Endpoint Events   Network Traffic
                   |             |
                   v             v
                 Wazuh     OVS Mirror
                                 |
                                 v
                          Security Onion
```

Wazuh may provide information about what occurred on the endpoint.

Security Onion may provide information about what occurred across the network.

Together, they provide broader visibility than either platform alone.

---

# Security Onion and pfSense

Security Onion also complements pfSense.

A firewall answers questions such as:

- Was the traffic permitted?
- Was the traffic denied?
- Which security boundary did the communication attempt to cross?

Security Onion addresses different questions:

- What network activity occurred?
- What protocols were involved?
- What systems communicated?
- Was the behavior suspicious?
- Does the traffic warrant investigation?

Conceptually:

```text id="p48fdk"
                    Network Activity
                           |
              +------------+------------+
              |                         |
              v                         v
           pfSense                Security Onion
              |                         |
              v                         v
        Enforcement                  Visibility
              |                         |
              v                         v
      Allow / Deny Decision      Detect / Investigate
```

These capabilities complement rather than replace one another.

---

# Security Onion and Vulnerability Testing

Vulnerability scanners and penetration-testing platforms generate significant network activity.

The environment includes technologies such as:

- Qualys
- Horizon3.ai NodeZero
- Kali Linux

Running these tools within a monitored environment provides an opportunity to observe how legitimate security-testing activity appears from a network-defense perspective.

For example:

```text id="45f2s7"
Qualys / NodeZero / Kali
          |
          | Scan / Test
          v
      Target System
          |
          +--------------> Endpoint Evidence
          |
          +--------------> Network Evidence
                                  |
                                  v
                           Security Onion
```

This helps connect vulnerability-management and offensive-security activities with detection engineering and defensive monitoring.

---

# Monitoring Honeypot Activity

Honeypot workloads can also provide useful telemetry for network-monitoring platforms.

A honeypot is intentionally exposed within a controlled environment to observe interaction with services designed to attract or record suspicious behavior.

Conceptually:

```text id="x0ksba"
External / Test Activity
          |
          v
       Honeypot
          |
          +------------> Honeypot Telemetry
          |
          +------------> Network Traffic
                               |
                               v
                        Security Onion
```

The honeypot remains subject to the same segmentation principles applied to other potentially hostile workloads.

---

# Isolation Considerations

A monitoring platform should not weaken the security boundaries it is intended to observe.

Security Onion's network interfaces should therefore be designed according to their specific purpose.

The management interface requires controlled administrative connectivity.

The monitoring interface requires access to mirrored traffic.

Neither requirement should automatically result in unrestricted access between otherwise isolated networks.

This distinction is especially important when monitoring malware-related or exploit-testing environments.

---

# Troubleshooting Methodology

When Security Onion does not display expected activity, I troubleshoot the architecture from the traffic source toward the monitoring platform.

```text id="qgd0g8"
1. Confirm traffic is actually generated
            |
            v
2. Confirm source and destination path
            |
            v
3. Confirm correct Proxmox network
            |
            v
4. Confirm OVS receives the traffic
            |
            v
5. Confirm mirror configuration
            |
            v
6. Confirm Security Onion interface
            |
            v
7. Confirm packet visibility
            |
            v
8. Review Security Onion telemetry
            |
            v
9. Review detection behavior
```

This layered approach helps distinguish a **network visibility problem** from a **detection problem**.

---

# Monitoring Architecture Principles

The Security Onion implementation follows several design principles.

## Passive Visibility Where Appropriate

Monitoring should not unnecessarily become part of the production traffic path.

## Separate Management and Monitoring Functions

Administrative access and packet monitoring serve different purposes and should be architected accordingly.

## Validate the Traffic Path

A configured mirror is not sufficient evidence that Security Onion receives the expected traffic.

Traffic visibility should be tested.

## Detection Follows Visibility

Before troubleshooting detection logic, verify that the underlying traffic reaches the monitoring platform.

## Maintain Segmentation

Monitoring requirements should not create unintended communication paths between trust zones.

## Use Multiple Security Perspectives

Network telemetry should complement endpoint, firewall, vulnerability, and identity telemetry.

---

# Lessons Learned

One of the most important lessons from implementing Security Onion within a virtualized environment is that **deploying the monitoring platform is only one component of network security monitoring**.

The complete architecture requires:

```text id="wd3u8o"
Traffic Source
      +
Correct Network Path
      +
Virtual Switching
      +
Traffic Mirroring
      +
Monitoring Interface
      +
Packet Visibility
      +
Detection Logic
      =
Network Security Monitoring
```

If any component in that chain is incorrectly configured, visibility can be incomplete even though Security Onion itself is operating normally.

Building the traffic-mirroring architecture therefore became as important as deploying the monitoring platform.

---

# Future Improvements

The Security Onion implementation can continue to evolve as additional monitoring requirements are introduced.

Potential improvements include:

- Expanding monitored network segments
- Additional traffic-mirroring scenarios
- Additional detection-validation exercises
- Detection tuning
- Expanded protocol analysis
- Additional honeypot monitoring
- Correlation with endpoint telemetry
- Expanded firewall telemetry
- Additional attack-simulation scenarios
- Automated validation of monitoring coverage

Changes will be introduced based on visibility requirements and identified monitoring gaps.

---

## Related Documentation

- Security Monitoring Architecture
- Wazuh Security Monitoring
- Network Security Architecture
- pfSense Network Segmentation
- Proxmox VE Infrastructure
- Vulnerability Management
- Security Testing

> **Security Note:** Network addressing, credentials, sensitive identifiers, management endpoints, and unnecessary infrastructure details are intentionally excluded or sanitized from public documentation.
