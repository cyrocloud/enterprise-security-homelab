# Active Directory Domain Services

## Overview

I maintain a dedicated **Active Directory Domain Services (AD DS)** environment within my Proxmox infrastructure to provide the on-premises identity foundation for the broader Enterprise Security Home Lab.

A Windows Server virtual machine, **DC01**, operates as the domain controller for the lab and provides centralized identity, authentication, policy, and directory services for selected Windows systems.

The Active Directory environment is also integrated with my independently managed Microsoft tenant through Microsoft Entra Connect, allowing the lab to support both traditional Active Directory and hybrid Microsoft identity scenarios.

The objective is to maintain an Active Directory environment that can be used for architecture testing, security validation, troubleshooting, Group Policy, endpoint integration, and hybrid identity engineering.

---

# Architecture

The Active Directory environment operates within the Proxmox infrastructure.

At a high level:

```text id="39txs4"
                     Proxmox VE
                         |
                         v
                       DC01
                         |
             Active Directory DS
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
      Users            Groups        Computers
        |                |                |
        +----------------+----------------+
                         |
                         v
                Domain Authentication
                         |
              +----------+----------+
              |                     |
              v                     v
       Windows Clients         Lab Servers
```

DC01 provides centralized directory and authentication services to domain-integrated systems within the lab.

---

# Domain Controller

DC01 is deployed as a Windows Server virtual machine within Proxmox.

Its primary role within the identity architecture is to provide Active Directory Domain Services.

Conceptually:

```text id="baf0f1"
                        DC01
                         |
              +----------+----------+
              |          |          |
              v          v          v
             AD DS      DNS     Group Policy
              |          |          |
              +----------+----------+
                         |
                         v
                  Domain Services
```

The domain controller acts as a central component for identity and policy within the on-premises portion of the lab.

---

# Active Directory Objects

The directory contains the primary object types required to support domain-based identity.

Examples include:

- User accounts
- Computer accounts
- Security groups
- Organizational Units
- Group Policy Objects
- Administrative identities

These objects allow identity and access to be managed centrally instead of configuring each Windows system independently.

---

# Organizational Units

Organizational Units provide logical organization within Active Directory and allow policy and administrative boundaries to be applied more deliberately.

Conceptually:

```text id="jrz0ec"
                  Active Directory
                         |
              +----------+----------+
              |          |          |
              v          v          v
            Users     Computers   Servers
                         |
                  +------+------+
                  |             |
                  v             v
              Windows 10    Windows 11
```

The exact structure can evolve as additional systems and testing scenarios are introduced.

The objective is to organize systems according to their purpose rather than maintaining every object in default containers.

---

# Security Groups

Security groups provide a centralized method for assigning access and policy based on membership.

Instead of assigning permissions individually:

```text id="uv98xk"
User A -----\
User B ------> Security Group ------> Resource / Policy
User C -----/
```

This provides a more scalable access model and mirrors the group-based authorization commonly used within enterprise Active Directory environments.

Groups can also be used when testing:

- Resource access
- Group Policy targeting
- Administrative delegation
- VPN access
- Application access
- Hybrid identity synchronization

---

# DNS

DNS is a critical dependency for Active Directory.

Domain-joined systems rely on DNS to locate Active Directory services and domain controllers.

The relationship can be simplified as:

```text id="yqt0d4"
Windows Client
      |
      | DNS Query
      v
   AD DNS
      |
      v
Locate Domain Controller
      |
      v
Authentication / Domain Services
```

This makes DNS troubleshooting an important part of Active Directory troubleshooting.

A system may have general IP connectivity while still experiencing domain failures if DNS is incorrectly configured.

---

# Domain Join

Selected Windows systems can be joined to the Active Directory domain.

The basic relationship becomes:

```text id="22yud3"
Windows Endpoint
      |
      v
Domain Join
      |
      v
Computer Object
      |
      v
Active Directory
      |
      v
Central Authentication
      +
Group Policy
```

After joining the domain, the endpoint becomes part of the centralized identity and policy architecture.

---

# Authentication

Active Directory provides centralized authentication for domain-integrated Windows systems.

Instead of maintaining unrelated local credentials on every endpoint:

```text id="zh1r1n"
             User
              |
              v
       Domain Credential
              |
              v
      Active Directory
              |
              v
       Authentication
              |
              v
       Windows System
```

This provides an environment where authentication behavior and security controls can be evaluated across multiple systems.

---

# Kerberos and NTLM

Windows domain environments can involve multiple authentication protocols.

**Kerberos** is the primary authentication protocol used in modern Active Directory environments when the necessary domain conditions are met.

NTLM may still appear in scenarios involving compatibility requirements, configuration issues, legacy systems, or situations where Kerberos cannot be used.

Conceptually:

```text id="n6ny2s"
                   Authentication
                         |
               +---------+---------+
               |                   |
               v                   v
            Kerberos              NTLM
            Preferred          Compatibility /
                               Fallback Cases
```

Maintaining an Active Directory environment provides a platform for understanding how these authentication methods appear in actual Windows infrastructure rather than only studying them conceptually.

---

# Group Policy

Group Policy is used to centrally configure Windows systems and security settings within the Active Directory environment.

The policy path can be represented as:

```text id="eczh9e"
Group Policy Object
        |
        v
Domain / OU
        |
        v
Computer / User
        |
        v
Policy Processing
        |
        v
Configured Setting
```

The lab provides a controlled environment where policies can be deployed, tested, modified, and removed without affecting production systems.

---

# Security Policy Testing

Group Policy can be used to evaluate security controls across domain-joined systems.

Examples of areas that can be tested include:

- Windows security settings
- Microsoft Defender configuration
- Firewall settings
- Authentication policies
- Audit policies
- Device restrictions
- BitLocker configuration
- Administrative settings
- Remote Desktop configuration
- Endpoint hardening

The validation workflow is:

```text id="nknu40"
Define Requirement
       |
       v
Create / Modify GPO
       |
       v
Link Policy
       |
       v
Update Endpoint Policy
       |
       v
Validate Result
       |
       v
Review Logs / Behavior
```

The important step is validation.

A configured GPO does not automatically prove that the intended endpoint received or successfully applied the setting.

---

# Group Policy Troubleshooting

When a policy does not behave as expected, I evaluate the policy-processing chain rather than immediately recreating the GPO.

```text id="4e3lwb"
GPO Exists?
    |
    v
Correct Link?
    |
    v
Correct OU?
    |
    v
Security Filtering?
    |
    v
Endpoint Connectivity?
    |
    v
Policy Processed?
    |
    v
Setting Applied?
```

Tools and techniques available for validation can include:

```text id="3wqrhh"
gpupdate /force

gpresult /r

gpresult /h report.html

rsop.msc
```

Event logs and the Group Policy Management Console can provide additional context when troubleshooting policy-processing issues.

---

# Active Directory Security

Because Active Directory provides centralized identity, compromise of the directory can affect a significant portion of the environment.

Security considerations include:

- Protecting privileged accounts
- Limiting administrative membership
- Restricting domain-controller access
- Maintaining endpoint security
- Monitoring authentication
- Applying security updates
- Reducing unnecessary services
- Reviewing group membership
- Maintaining appropriate firewall policy
- Monitoring configuration changes

The domain controller should be treated as critical security infrastructure rather than simply another Windows server.

---

# Privileged Access

Administrative identities require additional protection because they can modify directory configuration and potentially control domain-integrated systems.

Conceptually:

```text id="l42ezg"
Standard User
     |
     +----> Normal Resources


Administrative Identity
     |
     +----> Directory Administration
     +----> Server Administration
     +----> Security Configuration
     +----> High-Impact Changes
```

The additional capability means administrative access should be granted deliberately and kept separate from unnecessary day-to-day activity where practical.

---

# Active Directory and Network Segmentation

Identity requirements must also be considered when designing network segmentation.

Domain-integrated systems require access to specific Active Directory services.

Conceptually:

```text id="xmyj24"
Domain-Joined Endpoint
        |
        | Required AD Communication
        v
       DC01
        |
        v
Authentication / DNS / Policy
```

Firewall policy therefore needs to balance two objectives:

1. Permit the communication required for Active Directory.
2. Prevent unnecessary communication between security zones.

This is an important example of the relationship between identity architecture and network architecture.

---

# Security Monitoring

Active Directory also generates valuable security telemetry.

Relevant activity can include:

- Authentication attempts
- Account changes
- Group membership changes
- Administrative activity
- Security events
- Policy-related activity
- Account lockouts
- Failed authentication
- Successful authentication

Depending on the monitoring scenario, endpoint telemetry can be evaluated through security technologies deployed elsewhere in the lab.

Conceptually:

```text id="frvxcp"
                     DC01
                      |
               Security Events
                      |
            +---------+---------+
            |                   |
            v                   v
      Host Monitoring      Security Analysis
```

This makes Active Directory useful not only for identity testing but also for detection and investigation exercises.

---

# Active Directory and Attack Paths

Active Directory is also an important component when evaluating attack paths.

A security incident may begin with an endpoint but eventually involve identity.

For example:

```text id="slq7fy"
Compromised Endpoint
        |
        v
Credential Exposure
        |
        v
User Account
        |
        v
Additional Access
        |
        v
Privileged Identity
        |
        v
Domain Impact
```

This demonstrates why vulnerability management, endpoint security, network segmentation, and identity security should not be treated as completely independent disciplines.

---

# Hybrid Identity

The local Active Directory environment also serves as the foundation for the hybrid Microsoft identity architecture.

Selected directory identities can be synchronized to my Microsoft tenant using Microsoft Entra Connect.

Conceptually:

```text id="syu14b"
                Active Directory
                       |
                       v
                Entra Connect
                       |
                       v
              Synchronization
                       |
                       v
              Microsoft Entra ID
                       |
                       v
              Microsoft Cloud
```

This extends the local directory into a hybrid identity environment.

---

# Identity Source and Synchronization

Hybrid identity introduces an important distinction between the local directory object and its synchronized representation in Microsoft Entra ID.

Conceptually:

```text id="pjgr9z"
On-Premises Identity
        |
        v
Active Directory
        |
        v
Entra Connect
        |
        v
Synchronized Identity
        |
        v
Microsoft Entra ID
```

Understanding where an identity originates and which system controls particular attributes is important when troubleshooting hybrid environments.

---

# Hybrid Identity Troubleshooting

When a synchronized identity does not appear or behave as expected, the troubleshooting path can span multiple systems.

```text id="snv7w3"
AD Object
    |
    v
Correct Attributes?
    |
    v
Synchronization Scope?
    |
    v
Entra Connect
    |
    v
Synchronization Successful?
    |
    v
Microsoft Entra ID
    |
    v
Expected Cloud Object?
```

This layered approach helps isolate whether the problem originates in Active Directory, synchronization configuration, or the cloud identity layer.

---

# Integration with Microsoft Security

The hybrid identity environment also connects with other Microsoft security technologies used in the lab.

Conceptually:

```text id="3ctiwj"
                    Microsoft Tenant
                          |
            +-------------+-------------+
            |                           |
            v                           v
     Microsoft Entra ID       Microsoft Security
            |                           |
            |                      MDE / Intune
            |                           |
            +-------------+-------------+
                          |
                    Entra Connect
                          |
                          v
                 Active Directory
                          |
                          v
                       DC01
                          |
                          v
                 Windows Endpoints
```

This provides a platform for evaluating identity, endpoint management, endpoint security, and hybrid infrastructure together.

---

# Validation Methodology

Changes within the Active Directory environment are validated across the complete path whenever possible.

```text id="thxjs4"
Configure
    |
    v
Apply
    |
    v
Force / Wait for Processing
    |
    v
Validate Endpoint
    |
    v
Review Logs
    |
    v
Confirm Expected Behavior
```

This approach is particularly useful when working with Group Policy and hybrid identity because configuration success and operational success are not always the same thing.

---

# Troubleshooting Methodology

When troubleshooting Active Directory, I generally work from foundational dependencies upward.

```text id="2ahq3r"
Network Connectivity
        |
        v
DNS
        |
        v
Domain Controller Reachability
        |
        v
Authentication
        |
        v
Directory Object
        |
        v
Group / OU
        |
        v
Group Policy
        |
        v
Application / Service
```

This helps avoid troubleshooting higher-level services when a lower-level dependency is responsible for the failure.

---

# Architecture Principles

The Active Directory implementation follows several principles.

## Centralize Identity

User and computer identities should be managed centrally where appropriate.

## Organize Intentionally

OUs and groups should reflect administrative, policy, and access requirements.

## Protect Privileged Access

Administrative identities and domain controllers require stronger protection than standard endpoints.

## Treat DNS as Core Infrastructure

Active Directory depends heavily on correct DNS configuration.

## Validate Group Policy

A configured policy should be verified on the endpoint.

## Monitor Authentication

Authentication activity provides important security telemetry.

## Design for Hybrid Identity

The directory should be understood not only as a local identity platform but also as a component of the broader Microsoft identity architecture.

---

# Lessons Learned

Building and maintaining Active Directory within the lab reinforces that AD DS is not simply a user database.

It combines multiple infrastructure and security functions:

```text id="76t05x"
Identity
   +
Authentication
   +
DNS
   +
Group Policy
   +
Authorization
   +
Device Management
   +
Security Monitoring
   +
Hybrid Synchronization
   =
Active Directory Infrastructure
```

Because many of these components depend on one another, troubleshooting requires understanding the complete architecture rather than focusing on a single service.

Integrating the local directory with my Microsoft tenant further extends the environment into a realistic hybrid identity architecture where on-premises identity decisions can be evaluated against cloud identity and endpoint-security technologies.

---

# Future Improvements

Areas for continued development include:

- Additional Group Policy security baselines
- Expanded audit policies
- Additional administrative separation
- Additional privileged-access testing
- Windows LAPS
- Additional authentication testing
- Active Directory security hardening
- Additional attack-path validation
- Expanded identity monitoring
- Additional Windows Server workloads
- Hybrid identity security testing
- Identity-focused detection scenarios

These improvements will continue to strengthen the relationship between Active Directory, endpoint security, network security, and Microsoft cloud identity within the lab.

---

## Related Documentation

- Identity and Access Architecture
- Microsoft Entra Connect
- Hybrid Identity
- Microsoft Entra ID
- Microsoft Defender for Endpoint
- pfSense Network Segmentation
- Wazuh Security Monitoring
- Vulnerability Management
