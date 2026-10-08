# Hybrid Identity Architecture

## Overview

I designed and implemented a hybrid Microsoft identity environment connecting my on-premises Active Directory infrastructure with Microsoft Entra ID through a personally managed Microsoft cloud tenant.

The environment combines traditional Windows domain services with Microsoft cloud identity and security technologies, providing a controlled platform for evaluating enterprise identity architecture.

The implementation spans multiple infrastructure layers:

- **Proxmox VE** — virtualization infrastructure hosting the on-premises workloads
- **Active Directory Domain Services** — on-premises directory and authentication services
- **Microsoft Entra Connect** — identity synchronization between on-premises AD and Microsoft Entra ID
- **Microsoft Entra ID** — cloud identity and access management
- **Microsoft Defender for Endpoint** — endpoint security and device telemetry
- **Microsoft Intune** — endpoint management and security policy testing

A key architectural objective is to validate how identities, devices, authentication mechanisms, and security controls operate across on-premises and cloud boundaries.

---

## Architecture Overview

The hybrid environment consists of two primary identity platforms connected through a synchronization layer.

```text
                   MICROSOFT CLOUD
            +----------------------------+
            |                            |
            |     Microsoft Entra ID     |
            |                            |
            |   Personally Managed       |
            |       Microsoft Tenant     |
            |                            |
            +-------------+--------------+
                          ^
                          |
                Identity Synchronization
                          |
                Microsoft Entra Connect
                          ^
                          |
            +-------------+--------------+
            |                            |
            |    Active Directory DS     |
            |                            |
            |           DC01             |
            |                            |
            +-------------+--------------+
                          |
                  Proxmox Network
                          |
              +-----------+-----------+
              |                       |
         Windows VMs              Lab Servers
```

Active Directory provides the on-premises identity source for synchronized objects, while Microsoft Entra ID provides the cloud identity platform.

The synchronization service bridges these environments without eliminating their separate administrative and security responsibilities.

---

## Personally Managed Microsoft Tenant

An important component of this architecture is that I maintain and administer my own Microsoft tenant.

This provides direct control over the cloud identity environment, including the ability to configure identity settings, evaluate security features, create test accounts, and validate changes independently.

The tenant is integrated with my local infrastructure rather than operating as a standalone cloud environment.

This architecture allows me to evaluate identity scenarios across both sides of the hybrid boundary.

### Engineering Capabilities

The environment supports testing involving:

- Cloud-only and synchronized identities
- Microsoft Entra administrative roles
- User and group management
- Identity synchronization
- Authentication configuration
- Device identity
- Microsoft security integration
- Access-control policies
- Identity lifecycle operations

Availability of specific features depends on the licenses and services enabled within the tenant.

---

## On-Premises Identity Foundation

The on-premises identity infrastructure is hosted within Proxmox.

The domain controller, DC01, provides Active Directory Domain Services for the lab.

Active Directory is responsible for maintaining the on-premises directory and supporting domain-integrated systems.

### Core Components

| Component | Responsibility |
|---|---|
| Active Directory DS | Directory and domain services |
| DC01 | Domain controller |
| DNS | Domain service discovery and name resolution |
| Kerberos | Domain authentication |
| Group Policy | Centralized Windows configuration |
| Security Groups | Group-based authorization |
| Organizational Units | Logical organization and policy scope |

This provides the traditional identity foundation required for hybrid synchronization and domain-based security testing.

---

## Identity Synchronization Architecture

Microsoft Entra Connect provides the synchronization layer between Active Directory and Microsoft Entra ID.

For synchronized identities, the general lifecycle is:

```text
       Active Directory
              |
              v
       User / Group Object
              |
              v
       Synchronization Scope
              |
              v
       Microsoft Entra Connect
              |
              v
       Attribute Processing
              |
              v
       Microsoft Entra ID
              |
              v
       Synchronized Identity
```

Synchronization allows selected on-premises identity information to be represented in Microsoft Entra ID.

The synchronization process must account for object scope, attribute consistency, matching behavior, and source-of-authority considerations.

**Identity synchronization is not equivalent to authentication.** Synchronization determines how directory information is represented across environments, while authentication depends on the configured sign-in method.

---

## Identity Source of Authority

One of the most important design considerations in hybrid identity is understanding which system controls an identity and its attributes.

For an identity originating in Active Directory, many synchronized attributes are managed through the on-premises directory.

Other properties and security settings may be managed in Microsoft Entra ID.

The precise management boundary depends on the attribute, synchronization configuration, and enabled cloud capabilities.

### Example

```text
     On-Premises AD User
              |
              | Directory Attributes
              v
         Entra Connect
              |
              v
       Microsoft Entra ID
              |
              +---- Cloud Authentication
              |
              +---- Application Access
              |
              +---- Cloud Security Policies
```

Understanding these boundaries is essential when troubleshooting attribute changes, authentication behavior, and identity administration.

---

## Hybrid Authentication

Hybrid identity supports multiple authentication architectures.

Common Microsoft authentication approaches include:

- Password Hash Synchronization (PHS)
- Pass-through Authentication (PTA)
- Federation

These are alternative architectural approaches, not interchangeable components that must all be deployed together.

### Password Hash Synchronization

With PHS, supported password hash material is synchronized to Microsoft Entra ID, allowing cloud authentication to occur without contacting the on-premises domain controller for every sign-in.

### Pass-through Authentication

PTA validates applicable sign-in requests against on-premises Active Directory through authentication agents.

### Federation

Federation delegates authentication to a configured identity provider.

Each approach has different availability, operational dependency, and security considerations.

The authentication method used in a deployment should be explicitly documented and validated rather than inferred from the presence of Entra Connect.

---

## User Identity Lifecycle

Hybrid identity introduces a lifecycle that spans both on-premises and cloud directories.

```text
       Create AD User
             |
             v
       Assign Groups
             |
             v
       Synchronize
             |
             v
       Microsoft Entra ID
             |
             v
       Assign Cloud Access
             |
             v
       Modify / Disable User
             |
             v
       Synchronize Changes
             |
             v
       Validate Access State
```

Identity lifecycle validation should consider:

- User provisioning
- Attribute changes
- Group membership
- Authentication
- Access assignment
- Account disablement
- Deprovisioning
- Removal of unnecessary access

The objective is to verify that identity changes produce the intended behavior across both environments.

---

## Device Identity Architecture

Hybrid identity also introduces device identity considerations.

A Windows system can have different identity and management states.

| Device state | Description |
|---|---|
| AD domain joined | Joined to on-premises Active Directory |
| Microsoft Entra joined | Joined directly to Microsoft Entra ID |
| Microsoft Entra hybrid joined | Joined to AD DS and registered as hybrid joined in Entra ID |
| Microsoft Entra registered | Registered with Entra ID, commonly for workplace or personal device scenarios |

These states are distinct and should not be treated as interchangeable.

Device join state also does not automatically establish whether a device is enrolled in Microsoft Intune.

### Device Validation

A Windows endpoint can be inspected using:

```powershell
dsregcmd /status
```

Relevant values include:

```text
AzureAdJoined
DomainJoined
WorkplaceJoined
```

These values help identify the device's current registration and join state.

Additional checks are required to validate MDM enrollment, compliance, and endpoint security.

---

## Microsoft Intune Integration

The personally managed Microsoft tenant provides a platform for evaluating Microsoft Intune and endpoint-management scenarios.

Identity and device management intersect in areas such as:

- Device enrollment
- Configuration profiles
- Endpoint security policies
- Device compliance
- Device restrictions
- Security baselines
- Windows configuration management

A simplified management model is:

```text
       Microsoft Entra ID
               |
               v
         Device Identity
               |
               v
         Microsoft Intune
               |
               v
        Configuration Policy
               |
               v
          Windows Device
               |
               v
        Policy Validation
```

Device enrollment, licensing, policy assignment, and management authority must be validated separately.

---

## Microsoft Defender for Endpoint Integration

Microsoft Defender for Endpoint is implemented within the broader lab to evaluate endpoint protection and security controls.

Selected Windows systems have been onboarded for endpoint-security testing.

The environment has also been used to validate Device Control policies, including removable-storage restrictions.

This connects identity architecture with endpoint-security enforcement.

```text
          Microsoft Tenant
                 |
        +--------+--------+
        |                 |
        v                 v
   Microsoft Entra ID    Defender for Endpoint
        |                 |
        v                 v
   Identity Context    Device Security
        |                 |
        +--------+--------+
                 |
                 v
           Windows Endpoint
```

The platforms provide complementary capabilities.

Entra ID handles identity and access management, while Defender for Endpoint provides endpoint protection, detection, and response capabilities.

---

## Conditional Access and Zero Trust

Conditional Access is an important component of modern Microsoft identity security.

It provides a policy engine that can evaluate applicable signals when users access protected resources.

Depending on licensing and configuration, these signals can include:

- User or group
- Application
- Device state
- Location
- Sign-in risk
- User risk
- Authentication strength

A conceptual access flow is:

```text
          User Sign-In
               |
               v
         Microsoft Entra ID
               |
               v
       Conditional Access
               |
       +-------+-------+
       |               |
       v               v
   Requirements     Block Policy
       |               |
       v               v
   Evaluate Access    Deny
       |
       v
   Grant / Deny
```

Conditional Access is a potential enforcement layer within the architecture. Individual policy implementations and results should be documented separately once validated.

---

## Privileged Identity Security

Privileged identities represent a high-impact security boundary across both Active Directory and Microsoft Entra ID.

A compromised privileged identity may allow an attacker to modify directory configuration, access cloud services, or weaken security controls.

Architectural considerations include:

- Separate administrative identities
- Least-privilege role assignments
- Strong authentication
- Protected administrative access
- Privileged Identity Management where licensed
- Monitoring administrative activity
- Reviewing privileged group membership

Hybrid environments require careful consideration of both on-premises and cloud privilege boundaries.

---

## Identity Security and Network Segmentation

Hybrid identity depends on communication across multiple infrastructure layers.

These dependencies should be considered when designing pfSense firewall policies and Proxmox networking.

```text
        Windows Endpoint
               |
               v
          Active Directory
               |
               v
          Entra Connect
               |
               v
         Microsoft Cloud
```

Each connection has distinct network and authentication requirements.

Security controls should permit necessary communication without unnecessarily exposing management systems or unrelated security zones.

---

## Security Monitoring

Identity activity can generate valuable telemetry across both environments.

### On-Premises

Examples include:

- Successful and failed authentication
- Account lockouts
- Security group changes
- Account creation and modification
- Privileged activity
- Directory service events

### Microsoft Cloud

Examples include:

- Microsoft Entra sign-in logs
- Audit logs
- Authentication events
- Administrative changes
- Conditional Access results, where applicable

These telemetry sources provide different perspectives into identity activity.

Centralized analysis may require additional integrations, configuration, and licensing.

---

## Hybrid Identity Validation

I use a layered validation methodology to evaluate the architecture.

### 1. Validate Active Directory

Confirm that the expected user or computer object exists and has the correct attributes.

### 2. Validate Synchronization Scope

Confirm that the object is included in the intended synchronization configuration.

### 3. Validate Entra Connect

Review synchronization health, processing, and errors.

### 4. Validate Microsoft Entra ID

Confirm that the expected cloud object exists and reflects the intended synchronized attributes.

### 5. Validate Authentication

Test the configured authentication method using an authorized test identity.

### 6. Validate Device State

Where applicable, verify the device's join and registration status.

### 7. Validate Security Controls

Confirm that assigned security and access policies produce the expected results.

### 8. Review Telemetry

Review the relevant logs to verify the observed behavior.

---

## Troubleshooting Methodology

Hybrid identity troubleshooting requires understanding multiple dependencies.

```text
       Identity Issue
             |
             v
       Active Directory
             |
             v
       Object Attributes
             |
             v
       Synchronization Scope
             |
             v
       Entra Connect
             |
             v
       Microsoft Entra ID
             |
             v
       Authentication
             |
             v
       Device State
             |
             v
       Access Policy
             |
             v
       Resource Access
```

This approach helps distinguish synchronization issues from authentication failures, device-registration problems, and access-policy behavior.

### Useful Validation Tools

For Windows and Active Directory:

```powershell
Get-ADUser -Identity "testuser" -Properties *

Get-ADComputer -Identity "TEST-PC01" -Properties *

gpresult /r

dsregcmd /status
```

For Microsoft Entra Connect Sync installations, synchronization diagnostics may include:

```powershell
Get-ADSyncScheduler
```

And, when a manual delta synchronization is appropriate:

```powershell
Start-ADSyncSyncCycle -PolicyType Delta
```

These synchronization commands are applicable to Microsoft Entra Connect Sync environments with the relevant components and permissions; they are not general-purpose commands for Microsoft Entra Cloud Sync.

---

## Architecture Decisions and Tradeoffs

| Design area | Architectural consideration |
|---|---|
| On-premises AD | Supports traditional domain-based identity and Group Policy |
| Entra Connect | Introduces synchronization capability and operational dependencies |
| Microsoft Entra ID | Provides cloud identity and access management |
| Authentication | Method selection affects resilience and on-premises dependencies |
| Device identity | Join state affects available management and access scenarios |
| Intune | Provides centralized endpoint management when appropriately configured |
| Defender for Endpoint | Adds endpoint protection and security visibility |
| Conditional Access | Adds identity-aware access enforcement where configured |
| Segmentation | Limits unnecessary network exposure |
| Monitoring | Supports investigation and operational validation |

The architecture is intentionally hybrid because it allows evaluation of both traditional enterprise identity and modern Microsoft cloud security technologies.

---

## Lessons Learned

Hybrid identity is more than synchronizing users between two directories.

It introduces several interconnected engineering considerations:

```text
       Directory Architecture
                +
       Identity Synchronization
                +
          Authentication
                +
          Device Identity
                +
         Access Management
                +
         Endpoint Security
                +
         Security Monitoring
                +
        Operational Validation
                =
       Hybrid Identity Architecture
```

Maintaining administrative control over my own Active Directory infrastructure and Microsoft tenant allows me to evaluate the complete architecture independently.

It also provides a controlled environment for troubleshooting synchronization behavior, testing security controls, validating endpoint integration, and understanding the operational dependencies of hybrid identity.

---

## Future Improvements

Areas for continued development include:

- Additional Conditional Access scenarios
- Authentication strengths
- Phishing-resistant MFA
- Privileged Identity Management
- Microsoft Entra Identity Protection
- Hybrid device registration testing
- Cloud-native device deployment
- Expanded Intune compliance policies
- Identity governance
- Application registrations and enterprise applications
- Workload identity security
- Identity-based attack-path validation
- Additional identity monitoring
- Zero Trust architecture validation

These capabilities will be documented as they are implemented and tested.

---

## Related Documentation

- Identity and Access Architecture — `README.md`
- Active Directory Domain Services — `active-directory.md`
- Microsoft Entra Connect — `entra-connect.md`
- Microsoft Defender for Endpoint — `defender-for-endpoint.md`
- Proxmox VE Infrastructure — `../proxmox/README.md`
- Network Security Architecture — `../network-security/README.md`
- Security Monitoring Architecture — `../security-monitoring/README.md`
