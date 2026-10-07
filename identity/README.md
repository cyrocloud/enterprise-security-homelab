# Identity and Access Architecture

## Overview

The identity portion of my Enterprise Security Home Lab extends beyond the local Proxmox environment into a **Microsoft cloud tenant that I maintain and administer specifically for lab, engineering, and security testing**.

This allows me to design and validate identity architectures across both on-premises and cloud environments using systems that I control directly.

The environment includes:

- Microsoft Active Directory Domain Services
- Windows Server Domain Controller
- Microsoft Entra ID
- Microsoft Entra Connect
- Hybrid identities
- Windows domain-joined systems
- Microsoft Entra-integrated systems
- Microsoft Defender for Endpoint
- Microsoft Intune and endpoint-management testing
- Identity and endpoint security controls

The objective is to maintain a realistic hybrid Microsoft environment where identity architecture, synchronization, authentication, endpoint security, and access controls can be implemented and validated without depending on a production or customer environment.

---

# Personally Managed Microsoft Tenant

A key component of the lab is my own Microsoft tenant.

I maintain administrative control of the tenant and use it as an independent engineering environment for Microsoft cloud and security technologies.

This allows configurations to be implemented directly rather than limiting the lab to theoretical exercises or preconfigured training environments.

Conceptually:

```text id="xg4h8k"
              Microsoft Cloud
                     |
                     v
             Microsoft Entra ID
                     |
          Personally Managed Tenant
                     |
         +-----------+-----------+
         |                       |
         v                       v
   Identity Services       Security Services
         |                       |
         +-----------+-----------+
                     |
                     v
                Home Lab
```

Having a dedicated tenant provides an environment where identity and security changes can be tested independently before applying similar architectural concepts elsewhere.

---

# Hybrid Identity Architecture

The lab includes a hybrid identity architecture connecting local Active Directory infrastructure with Microsoft Entra ID.

At a high level:

```text id="m13f0s"
                   Microsoft Entra ID
                          |
                          |
                    Entra Connect
                          |
                          |
                 Active Directory
                          |
                         DC01
                          |
              +-----------+-----------+
              |                       |
              v                       v
       Windows Clients          Lab Servers
```

The environment provides both traditional Active Directory and cloud identity components within the same architecture.

This makes it possible to evaluate how identities move between the two environments and how Microsoft cloud security technologies interact with hybrid identities.

---

# Active Directory Domain Services

Active Directory Domain Services provides the on-premises identity foundation for the hybrid environment.

A Windows Server domain controller operates within the Proxmox infrastructure and provides traditional Active Directory functionality for selected lab systems.

The environment can be used to evaluate areas such as:

- User accounts
- Computer accounts
- Organizational Units
- Security groups
- Group Policy
- Domain authentication
- Administrative permissions
- Windows security policies
- Hybrid identity synchronization

Active Directory remains intentionally included because many enterprise environments continue to operate hybrid identity architectures rather than exclusively cloud-native environments.

---

# Microsoft Entra ID

Microsoft Entra ID provides the cloud identity layer of the environment.

Because the tenant is independently managed, I can configure and evaluate Microsoft identity functionality directly within my own environment.

The tenant provides a platform for working with concepts such as:

- Cloud identities
- Hybrid identities
- Device identities
- Authentication
- Administrative roles
- Security groups
- Application access
- Identity security
- Endpoint integration

This extends the home lab beyond the local hypervisor and creates a hybrid infrastructure spanning both local and Microsoft cloud services.

---

# Microsoft Entra Connect

Microsoft Entra Connect provides the synchronization layer between Active Directory and Microsoft Entra ID.

Conceptually:

```text id="40l4qs"
          On-Premises Identity
                   |
                   v
         Active Directory DS
                   |
                   v
          Microsoft Entra Connect
                   |
                   v
          Identity Synchronization
                   |
                   v
          Microsoft Entra ID
                   |
                   v
             Cloud Services
```

This architecture allows selected identities created within Active Directory to be represented within the Microsoft cloud environment.

It also provides an environment for understanding and validating the dependencies involved in hybrid identity synchronization.

---

# Identity Synchronization

Hybrid identity introduces an additional lifecycle that does not exist in a standalone Active Directory environment.

```text id="3wg2y7"
Create Identity
      |
      v
Active Directory
      |
      v
Synchronization
      |
      v
Microsoft Entra ID
      |
      v
Cloud Identity
      |
      v
Microsoft Services
```

Changes to identity attributes can then be evaluated across the synchronization boundary.

This makes it possible to observe how local identity configuration affects the corresponding cloud identity.

---

# Hybrid Identity Validation

A synchronization service reporting success is not enough to validate the entire identity architecture.

I validate the environment across multiple layers:

```text id="pm2d2z"
Active Directory
      |
      v
Identity Exists?
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
Expected Cloud Identity?
      |
      v
Authentication / Access
      |
      v
Expected Behavior?
```

This provides a more complete validation process than checking only synchronization status.

---

# Identity Lifecycle

The environment can also be used to evaluate identity lifecycle operations.

Examples include:

```text id="80i71y"
Create User
    |
    v
Assign Groups
    |
    v
Synchronize Identity
    |
    v
Assign Access
    |
    v
Modify Identity
    |
    v
Disable Identity
    |
    v
Remove Access
```

Understanding this lifecycle is important because identity security extends beyond authentication.

Access must also change appropriately as identities change.

---

# Device Identity

The lab also contains Windows systems that can be used to evaluate device identity alongside user identity.

This creates an architecture where access decisions can involve both:

```text id="hmbcgw"
             User Identity
                   +
             Device Identity
                   |
                   v
          Access / Security
                   |
                   v
           Microsoft Cloud
```

Device identity becomes increasingly important when integrating Microsoft endpoint-management and security technologies.

---

# Microsoft Intune

The Microsoft tenant can also be used to evaluate endpoint-management concepts through Microsoft Intune.

This extends identity testing into device configuration and management.

Depending on the scenario being evaluated, the environment can be used for areas such as:

- Device enrollment
- Configuration profiles
- Security policies
- Endpoint security
- Device compliance
- Windows configuration
- Security-control validation

This connects identity architecture with endpoint-management architecture.

---

# Microsoft Defender for Endpoint

Microsoft Defender for Endpoint is also used within the environment to evaluate endpoint-security capabilities.

Selected Windows systems have been onboarded to Microsoft Defender for Endpoint, providing another security layer within the lab.

Conceptually:

```text id="y16qch"
              Windows Endpoint
                     |
          +----------+----------+
          |                     |
          v                     v
    Identity / Device      Endpoint Security
          |                     |
          v                     v
      Entra ID                  MDE
          |                     |
          +----------+----------+
                     |
                     v
             Microsoft Tenant
```

This allows endpoint-security controls to be evaluated alongside identity and device configuration.

---

# Device Control Testing

The lab has also been used to evaluate Microsoft Defender for Endpoint **Device Control** policies.

One validation scenario involved controlling USB/removable-storage access on selected Windows systems.

The purpose of this testing is not simply to deploy a policy.

The complete validation process is:

```text id="z48kdt"
Define Security Requirement
          |
          v
Configure Policy
          |
          v
Apply to Test Endpoint
          |
          v
Generate Test Activity
          |
          v
Validate Enforcement
          |
          v
Review Security Behavior
```

This allows Microsoft endpoint-security controls to be evaluated against known test conditions.

---

# Identity and Endpoint Security

Identity and endpoint security are closely related.

A valid username and password should not automatically be treated as sufficient evidence that access is trustworthy.

A modern security architecture can consider multiple signals:

```text id="4sjz3h"
User Identity
      +
Authentication
      +
Device Identity
      +
Device Security
      +
Access Policy
      =
Access Decision
```

Maintaining both Microsoft identity and endpoint technologies within the same tenant provides an environment for evaluating these relationships.

---

# Hybrid Environment

The resulting architecture spans multiple layers:

```text id="e98d4x"
                    Microsoft Cloud
                          |
        +-----------------+-----------------+
        |                                   |
        v                                   v
 Microsoft Entra ID              Microsoft Security
        |                                   |
        |                              MDE / Intune
        |                                   |
        +-----------------+-----------------+
                          |
                    Entra Connect
                          |
                          v
                  Active Directory
                          |
                          v
                       DC01
                          |
            +-------------+-------------+
            |                           |
            v                           v
      Windows Clients              Lab Servers
```

This provides a realistic platform for testing interactions between traditional enterprise identity and modern cloud security services.

---

# Relationship with the Broader Security Lab

Identity does not operate independently from the rest of the security architecture.

The identity environment exists alongside:

- pfSense network segmentation
- Wazuh endpoint monitoring
- Security Onion network monitoring
- Qualys vulnerability assessment
- NodeZero attack-path validation
- Microsoft Defender for Endpoint
- Proxmox virtualization

This creates opportunities to evaluate security scenarios across multiple layers.

For example:

```text id="1c0c6m"
Identity Weakness
      |
      v
Endpoint Access
      |
      v
Network Movement
      |
      v
Security Monitoring
      |
      v
Detection / Investigation
```

This is particularly useful when evaluating how identity weaknesses can contribute to broader attack paths.

---

# Identity as a Security Boundary

Identity is treated as a security boundary rather than simply a directory containing user accounts.

A compromised identity can potentially provide access to:

- Endpoints
- Applications
- Administrative interfaces
- Cloud resources
- Security platforms
- Additional identities

Identity architecture therefore requires the same level of security consideration as network architecture.

---

# Administrative Separation

Administrative access within the Microsoft tenant should be treated differently from standard user access.

Important design principles include:

- Least privilege
- Limited administrative roles
- Strong authentication
- Separation of administrative and standard activities
- Controlled privileged access
- Regular review of permissions

The lab provides a controlled environment where these concepts can be implemented and evaluated without affecting a production tenant.

---

# Security Validation

Because I control both the local infrastructure and Microsoft tenant, security changes can be validated across the entire path.

For example:

```text id="0kvtwr"
Configure Identity Control
          |
          v
Apply to Test User / Device
          |
          v
Attempt Expected Activity
          |
          v
Observe Authentication
          |
          v
Validate Enforcement
          |
          v
Review Logs / Telemetry
          |
          v
Adjust Configuration
```

This is particularly valuable for understanding not just how a setting is configured, but how it behaves when applied to an actual user or endpoint.

---

# Troubleshooting Methodology

Hybrid identity introduces multiple components that may contribute to an issue.

When troubleshooting, I approach the architecture in layers.

```text id="nuf7cu"
Active Directory
      |
      v
User / Device Object
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
Authentication
      |
      v
Device / Security Policy
      |
      v
Application / Resource
```

This helps isolate whether a problem originates from the local directory, synchronization layer, cloud identity, endpoint configuration, or access policy.

---

# Architecture Principles

The identity environment follows several core principles.

## Maintain an Independent Test Environment

Identity and security changes can be implemented within systems and a Microsoft tenant under my own administrative control.

## Validate Rather Than Assume

Successful configuration does not automatically mean expected behavior.

Authentication, synchronization, and security controls should be tested.

## Protect Privileged Access

Administrative identities represent a higher security risk and should receive stronger protection.

## Connect Identity and Endpoint Security

User identity, device identity, and endpoint security should be considered together.

## Maintain Hybrid Identity Knowledge

Modern enterprise environments frequently contain both Active Directory and cloud identity services.

Understanding the interaction between both architectures remains important.

## Treat Identity as a Security Control

Identity configuration directly affects the attack surface and should be included in broader security architecture decisions.

---

# Lessons Learned

Maintaining my own Microsoft tenant alongside an on-premises Active Directory lab has made it possible to evaluate Microsoft identity architecture across the complete hybrid path.

Instead of evaluating each technology independently, the environment can be viewed as:

```text id="ejapw1"
Active Directory
       +
Entra Connect
       +
Microsoft Entra ID
       +
Device Identity
       +
Endpoint Management
       +
Endpoint Security
       +
Security Validation
       =
Hybrid Identity Architecture
```

The ability to implement and test these configurations independently provides a practical environment for evaluating architecture decisions, security controls, synchronization behavior, endpoint integration, and troubleshooting scenarios.

---

# Future Improvements

The identity environment will continue to evolve as additional scenarios are evaluated.

Potential areas include:

- Additional hybrid identity scenarios
- Cloud-native identity testing
- Conditional Access
- Multi-factor authentication
- Authentication strengths
- Privileged Identity Management
- Role-based access control
- Identity governance
- Application registrations
- Enterprise applications
- Workload identities
- Additional Intune integration
- Additional Defender for Endpoint policies
- Identity-focused attack-path validation
- Expanded Zero Trust testing

The objective is to continue developing the environment as an integrated identity and security architecture rather than treating each Microsoft technology as an isolated product.

---

## Planned Documentation

Additional documentation within this section will cover:

- Active Directory Domain Services
- Microsoft Entra Connect
- Hybrid identity
- Microsoft Entra ID
- Microsoft Defender for Endpoint
- Device Control
- Microsoft Intune
- Additional identity-security controls

> **Security Note:** Tenant identifiers, domain names, usernames, credentials, authentication details, tokens, application secrets, internal IP addresses, and other sensitive identity information are intentionally excluded or sanitized from public documentation.
