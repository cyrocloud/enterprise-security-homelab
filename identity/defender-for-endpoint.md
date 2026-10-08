# Microsoft Defender for Endpoint — Deployment, Onboarding, and Device Control

## Overview

I implemented Microsoft Defender for Endpoint (MDE) within my Enterprise Security Home Lab to evaluate endpoint security architecture, device onboarding, centralized management, and security-control enforcement.

The implementation integrates Windows endpoints hosted in Proxmox with my independently managed Microsoft tenant.

The environment supports evaluating multiple onboarding and policy-delivery methods, including:

- Microsoft Intune
- Microsoft Defender for Endpoint onboarding packages
- Group Policy
- Microsoft Defender security settings management, where applicable

A primary validation scenario involved implementing **Microsoft Defender Device Control** to restrict removable USB storage access on Windows endpoints.

The objective is to validate the complete security-control lifecycle:

**Onboarding → Device Visibility → Policy Assignment → Policy Enforcement → Security Telemetry → Troubleshooting**

---

## Architecture Overview

```text
                     Microsoft Cloud
                            |
              +-------------+-------------+
              |                           |
              v                           v
       Microsoft Intune          Microsoft Defender XDR
              |                           |
              |                     Defender for Endpoint
              |                           |
              +-------------+-------------+
                            |
                     Windows Endpoints
                            |
                    Proxmox VE Lab
                            |
                +-----------+-----------+
                |                       |
                v                       v
           Windows 11               Windows 10
                |                       |
                +-----------+-----------+
                            |
                            v
                   Security Validation
```

Microsoft Intune provides device-management and policy-delivery capabilities.

Microsoft Defender for Endpoint provides endpoint protection, security telemetry, detection, and response capabilities.

These services are complementary but perform different functions.

---

## Microsoft Defender for Endpoint Onboarding

Endpoint onboarding establishes the connection between a supported Windows device and the Microsoft Defender for Endpoint service.

A successfully onboarded device should become visible within the Microsoft Defender portal and begin reporting supported endpoint telemetry.

The onboarding method depends on the device's operating system, management state, and deployment requirements.

### Method 1 — Microsoft Intune Onboarding

For devices managed through Microsoft Intune, MDE onboarding can be configured through an Endpoint detection and response (EDR) policy.

General implementation workflow:

1. Verify Microsoft Defender for Endpoint licensing and tenant configuration.
2. Confirm the device is enrolled in Intune.
3. Open the Microsoft Intune admin center.
4. Navigate to **Endpoint security → Endpoint detection and response**.
5. Create an appropriate EDR policy for the Windows platform.
6. Configure the onboarding settings.
7. Assign the policy to the intended device group.
8. Synchronize the endpoint and validate policy application.
9. Confirm the endpoint appears in the Microsoft Defender device inventory.

Depending on tenant configuration, the onboarding package may be automatically supplied through the Defender integration.

### Architecture

```text
Microsoft Intune
       |
       v
EDR Onboarding Policy
       |
       v
Assigned Device Group
       |
       v
Windows Endpoint
       |
       v
MDE Sensor Onboarding
       |
       v
Microsoft Defender Portal
```

**Important:** Intune enrollment and MDE onboarding are separate states. A device can be enrolled in Intune without being successfully onboarded to MDE.

---

## Method 2 — Local Onboarding Package

Microsoft Defender for Endpoint also supports onboarding through packages downloaded from the Microsoft Defender portal.

This method is useful for controlled testing, troubleshooting, or supported deployment scenarios where Intune is not the onboarding mechanism.

### General Workflow

1. Open the Microsoft Defender portal.
2. Navigate to **Settings → Endpoints → Device management → Onboarding**.
3. Select the supported operating system.
4. Choose the appropriate deployment method.
5. Download the onboarding package.
6. Extract the package where required.
7. Execute or deploy the package using the prescribed method.
8. Verify sensor status.
9. Confirm device visibility in Microsoft Defender.

For Windows client devices, local script onboarding is primarily intended for testing or limited deployment scenarios rather than large-scale production deployment.

### Validation

```powershell
Get-Service Sense
```

Review the sensor service and its operational state.

Additional registry validation:

```powershell
Get-ItemProperty `
  -Path "HKLM:\SOFTWARE\Microsoft\Windows Advanced Threat Protection" `
  -Name OnboardingState `
  -ErrorAction SilentlyContinue
```

An `OnboardingState` value of `1` is an expected indicator of successful onboarding, but should be evaluated alongside sensor health and portal visibility.

---

## Method 3 — Group Policy Onboarding

Group Policy provides another deployment option for domain-joined Windows endpoints.

This method is particularly relevant within my lab because the environment includes an Active Directory domain controller, DC01.

### Architecture

```text
                    DC01
                     |
              Active Directory
                     |
                Group Policy
                     |
                     v
              Windows Endpoint
                     |
                     v
               MDE Onboarding
                     |
                     v
             Microsoft Defender
```

### General Workflow

1. Download the appropriate Group Policy onboarding package from Microsoft Defender.
2. Extract the package.
3. Configure the deployment using the Microsoft-supported Group Policy procedure.
4. Link the GPO to the intended OU.
5. Verify security filtering and policy scope.
6. Allow Group Policy processing or initiate an update.
7. Validate the onboarding state on the endpoint.
8. Confirm device visibility within Microsoft Defender.

The deployment should use the supported package and procedure for the selected operating system.

---

## Onboarding Validation

Successful deployment of an onboarding package does not automatically prove the endpoint is reporting correctly.

Validation should occur at multiple layers.

| Validation | Expected result |
|---|---|
| Device management | Device is enrolled or managed as intended |
| Onboarding configuration | Correct MDE tenant and deployment method |
| SENSE service | Service is present and operating appropriately |
| Onboarding registry state | Expected onboarding state |
| Network connectivity | Required Microsoft Defender service endpoints reachable |
| Defender portal | Device appears in inventory |
| Sensor health | Device reports expected health |
| Security telemetry | Supported endpoint events are received |

### Endpoint Diagnostics

```powershell
Get-Service Sense, WinDefend -ErrorAction SilentlyContinue
```

The services have different responsibilities:

- **Sense:** Microsoft Defender for Endpoint sensor service.
- **WinDefend:** Microsoft Defender Antivirus service.

MDE onboarding and Microsoft Defender Antivirus protection are related but distinct components.

---

# Microsoft Defender Device Control

## Overview

Microsoft Defender Device Control provides capabilities for managing access to supported removable storage and other device categories.

Within my lab, I evaluated Device Control to restrict USB removable-storage access on Windows endpoints.

The security objective was to prevent unauthorized removable-storage activity and validate enforcement at the endpoint.

### Security Objective

```text
USB Storage Connected
         |
         v
Device Control Policy
         |
         v
Evaluate Matching Rules
         |
    +----+----+
    |         |
    v         v
  Allowed   Denied
    |         |
    v         v
  Access    Blocked
```

The implementation is intended to demonstrate actual enforcement rather than simply configuring a policy in the management portal.

---

## Important: Two Configuration Layers

One of the most important findings during implementation is that Device Control requires the appropriate **MDE feature configuration and enforcement policy**.

These should not be confused.

### Layer 1 — Enable MDE Device Control

In Microsoft Intune, the relevant policy settings can include:

**Endpoint security → Attack surface reduction → Device Control**

Under the Defender configuration category:

| Setting | Configuration |
|---|---|
| Device Control Enabled | Device Control is enabled |
| Allow Full Scan Removable Drive Scanning | Configure according to security requirements |
| Default Enforcement | Configure deliberately based on the intended behavior |
| Secured Devices Configuration | Select the applicable device categories |

In my lab, I configured the Device Control capability and evaluated removable-storage restrictions.

**Important:** Setting *Device Control Enabled* does not, by itself, guarantee that USB storage is blocked.

The effective enforcement depends on the selected device categories, default enforcement behavior, applicable rules, exclusions, and endpoint policy state.

---

## Layer 2 — Configure Device Control Enforcement

After enabling Device Control, the required access restrictions must be configured.

A policy may evaluate properties such as:

- Device class or category
- Removable storage type
- Hardware identifiers
- Device identifiers
- User or group scope
- Read access
- Write access
- Execute access

The specific policy model determines whether access is allowed, denied, or audited.

### Example: Block Removable Storage

```text
Windows Endpoint
       |
       v
USB Storage Inserted
       |
       v
MDE Device Control
       |
       v
Matching Policy / Default Enforcement
       |
       v
Deny Applicable Access
       |
       v
USB Storage Restricted
```

The effective configuration must be tested using the actual removable-storage devices and Windows systems included in the lab.

---

## Policy Delivery Methods

MDE Device Control can be configured through supported management methods, depending on the operating system and policy type.

Within the lab, the relevant approaches include:

### Microsoft Intune

Cloud-managed policy delivery through Intune.

### Group Policy

Domain-based configuration for supported Microsoft Defender Device Control settings.

### Security Settings Management

For eligible MDE-onboarded devices, Microsoft Defender security settings management may provide an additional supported management path.

These management approaches should not be assumed to be interchangeable for every setting.

Policy ownership and overlapping configurations must be reviewed to avoid conflicting enforcement.

---

## Device Control Validation

The most important part of the implementation was validating actual USB restriction behavior.

The lab included a block-all removable-storage testing scenario, with enforcement confirmed on the Windows test endpoint.

### Validation Workflow

1. Confirm the Windows endpoint is onboarded to MDE.
2. Verify that the appropriate Device Control settings are configured.
3. Confirm the enforcement policy is assigned to the intended device.
4. Validate policy delivery.
5. Insert a test USB removable-storage device.
6. Attempt the access operations covered by the policy.
7. Confirm that prohibited operations are denied.
8. Review available policy and security telemetry.
9. Retest after configuration changes.

### Test Matrix

| Test scenario | Expected behavior |
|---|---|
| USB device inserted | Device is recognized according to Windows and policy behavior |
| Read restricted USB storage | Access denied when covered by the deny policy |
| Write restricted USB storage | Access denied when covered by the deny policy |
| Execute restricted USB content | Execution denied when covered by the deny policy |
| Approved exception | Access permitted only where an explicit applicable exception exists |
| Policy removed in controlled test | Behavior changes according to remaining effective policies |

This matrix represents a repeatable validation methodology. Individual outcomes should be recorded against the actual test endpoint.

---

# Troubleshooting MDE Onboarding

## Scenario 1 — Device Does Not Appear in Defender

Potential causes include:

- Incorrect onboarding package
- Wrong tenant configuration
- Sensor service issue
- Required service endpoints unreachable
- Proxy or firewall restrictions
- Incomplete policy assignment
- Unsupported operating-system configuration
- Delayed reporting

### Troubleshooting Sequence

```text
Onboarding Deployment
         |
         v
Confirm Package / Policy
         |
         v
Check SENSE Service
         |
         v
Check Onboarding State
         |
         v
Check Network Connectivity
         |
         v
Review Sensor Health
         |
         v
Confirm Portal Visibility
```

Do not repeatedly deploy onboarding packages without first determining which layer is failing.

---

## Scenario 2 — Device Is in Intune but Not MDE

Intune enrollment does not guarantee Defender for Endpoint onboarding.

Validate:

- EDR policy assignment
- Policy deployment status
- MDE onboarding configuration
- Sensor state
- Device eligibility
- Required connectivity

Also confirm that the device is reporting to the expected tenant.

---

## Scenario 3 — Device Is in MDE but Not Intune

MDE onboarding and Intune enrollment are separate processes.

A device may report to Microsoft Defender without being enrolled in Intune.

If Intune management is required, investigate the appropriate enrollment path independently.

MDE security settings management may support certain security configurations for eligible devices, but it does not make the device equivalent to a fully Intune-enrolled endpoint.

---

# Troubleshooting Device Control

## Scenario 1 — USB Policy Exists but Storage Is Not Blocked

This is an important troubleshooting scenario.

A policy can exist in Intune while the endpoint continues to permit USB storage access.

Potential causes include:

- MDE Device Control is not enabled.
- The applicable device category is not configured.
- Default enforcement permits access.
- No matching deny rule applies.
- A higher-priority or otherwise applicable allow rule affects the result.
- Policy assignment does not include the endpoint.
- The endpoint has not received the configuration.
- The device type does not match the intended restriction.
- Another management policy conflicts with the configuration.
- The Windows platform or Defender components do not meet the relevant requirements.

### Troubleshooting Workflow

```text
USB Storage Not Blocked
          |
          v
Is MDE Device Control Enabled?
          |
          v
Is Device Category Configured?
          |
          v
Is Deny Enforcement Configured?
          |
          v
Is Policy Assigned?
          |
          v
Did Endpoint Receive Policy?
          |
          v
Does USB Match the Rule?
          |
          v
Is Another Policy Affecting Access?
          |
          v
Retest Enforcement
```

The key distinction is between **policy creation, policy delivery, and actual policy enforcement**.

---

## Scenario 2 — Policy Reports Success but USB Still Works

A successful management status does not necessarily prove that the expected restriction is effective.

Troubleshooting should include:

1. Review the exact settings configured.
2. Confirm the effective default enforcement.
3. Review device groups and rule definitions.
4. Check for exclusions or conflicting policies.
5. Validate the endpoint's Defender platform and security intelligence components as relevant.
6. Generate a controlled USB access test.
7. Review the resulting endpoint behavior and available events.

This prevents confusing successful policy deployment with successful security enforcement.

---

## Scenario 3 — Group Policy and Intune Conflict

In a hybrid environment, endpoints may receive settings through multiple management channels.

For example:

```text
               Windows Endpoint
                      ^
             +--------+--------+
             |                 |
             v                 v
       Group Policy       Microsoft Intune
             |                 |
             +--------+--------+
                      |
                      v
             Effective Settings
```

When troubleshooting, determine which management mechanism owns the setting and whether another policy configures the same control.

Do not assume that Intune always overrides Group Policy or that Group Policy always takes precedence. Precedence and conflict behavior depend on the specific setting and policy mechanism.

---

# Device Control Policy Verification

When validating Device Control, I separate the process into four stages.

### 1. Configuration

Confirm the feature and enforcement settings exist.

### 2. Assignment

Confirm the intended device is targeted.

### 3. Delivery

Confirm the endpoint receives the configuration.

### 4. Enforcement

Confirm the endpoint actually blocks the prohibited action.

```text
Configuration
      |
      v
Assignment
      |
      v
Delivery
      |
      v
Enforcement
      |
      v
Security Validation
```

This model is useful for troubleshooting endpoint-security policies beyond USB restrictions.

---

# Security Monitoring and Investigation

Microsoft Defender for Endpoint provides endpoint telemetry that can be used to investigate security activity.

Depending on the configured capabilities, supported telemetry may include:

- Device inventory
- Security alerts
- Endpoint events
- Process activity
- Security configuration
- Device Control activity
- Investigation evidence

The lab also includes Wazuh and Security Onion, which provide additional host and network visibility.

These platforms serve complementary purposes and are not assumed to be directly integrated.

---

# Security Architecture Considerations

## Device Onboarding Is Not Device Management

MDE onboarding and Intune enrollment must be validated independently.

## Policy Assignment Is Not Enforcement

A policy reporting successful deployment does not prove that a prohibited action is blocked.

## Protect Management Access

Administrative access to Defender and Intune should follow least-privilege principles.

## Maintain Clear Policy Ownership

Avoid unnecessary overlap between Group Policy, Intune, and other security-management mechanisms.

## Validate Using Real Activity

Security controls should be tested using actual endpoint actions rather than relying only on portal status.

## Document Exceptions

Approved removable-storage exceptions should be narrowly scoped and validated separately.

---

# Lessons Learned

Implementing Microsoft Defender for Endpoint within a hybrid lab reinforces the difference between enabling a security capability and demonstrating effective protection.

The complete implementation requires:

```text
Endpoint Provisioning
        +
MDE Onboarding
        +
Device Management
        +
Security Policy Configuration
        +
Policy Assignment
        +
Endpoint Enforcement
        +
Security Telemetry
        +
Troubleshooting
        =
Validated Endpoint Security
```

The Device Control implementation particularly demonstrates why endpoint-security engineering requires more than configuring settings within a management console.

The meaningful outcome is whether the intended restriction is consistently enforced on the target device.

---

# Future Improvements

Potential improvements include:

- Expanded MDE onboarding validation
- Additional Windows Server onboarding
- Microsoft Defender security settings management
- Additional Device Control scenarios
- Approved USB device exceptions
- Device Control auditing
- Endpoint detection testing
- Attack surface reduction rules
- Tamper protection validation
- Endpoint security baselines
- Defender advanced hunting
- Additional Intune policy testing
- Automated policy validation
- Expanded integration with hybrid identity

These capabilities will be documented as they are implemented and tested.

---

## Related Documentation

- Identity and Access Architecture — `README.md`
- Active Directory — `active-directory.md`
- Microsoft Entra Connect — `entra-connect.md`
- Hybrid Identity — `hybrid-identity.md`
- Security Monitoring — `../security-monitoring/README.md`
- Wazuh — `../security-monitoring/wazuh.md`
- Proxmox VE — `../proxmox/README.md`
- 
