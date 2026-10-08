# Storage Architecture — TrueNAS SCALE

## Overview

The storage component of my Enterprise Security Home Lab is built around **TrueNAS SCALE**, deployed as a virtual machine within my Dell PowerEdge R630 running Proxmox VE.

TrueNAS provides a dedicated network-attached storage (NAS) platform for evaluating centralized file storage, SMB file sharing, storage permissions, and integration with Windows-based infrastructure.

The implementation extends the lab beyond virtualization and security tooling into storage architecture, where data availability, access control, network connectivity, and operational reliability become important design considerations.

The environment supports evaluating:

- TrueNAS SCALE virtualization
- Network-attached storage architecture
- SMB file sharing
- Windows client connectivity
- Dataset and file permissions
- Network segmentation
- Storage administration
- Data protection concepts
- Storage troubleshooting and validation

The objective is to maintain a centralized storage platform that can be independently configured, secured, tested, and integrated with other infrastructure services.

---

## Architecture Overview

TrueNAS SCALE operates as a virtualized storage workload within the Proxmox infrastructure.

```text
                 Dell PowerEdge R630
                         |
                     Proxmox VE
                         |
              +----------+----------+
              |                     |
              v                     v
        TrueNAS SCALE          Windows / Linux
              |                   Workloads
              |
              v
          ZFS Storage
              |
              v
        SMB File Sharing
              |
              v
        Network Connectivity
              |
        +-----+------+
        |            |
        v            v
   Windows 10    Windows 11
```

The architecture separates the NAS operating system from client workloads while providing centralized access to shared storage resources.

The underlying physical-disk presentation and ZFS pool configuration should be validated separately when evaluating storage redundancy and data protection.

---

## TrueNAS SCALE Deployment

TrueNAS SCALE is deployed as a dedicated virtual machine within Proxmox.

This allows the storage platform to be managed independently of other lab workloads.

### Deployment Considerations

Key considerations include:

- Virtual machine resource allocation
- Virtual disk or physical-disk presentation
- Storage pool configuration
- Virtual network connectivity
- Storage service availability
- Administrative access
- Backup and recovery requirements

Virtualizing a NAS introduces additional architectural dependencies because the storage platform relies on both the guest operating system and the underlying hypervisor.

These dependencies must be considered when evaluating availability and recovery.

---

## Storage Architecture

TrueNAS SCALE uses OpenZFS for storage management.

The general storage hierarchy is:

```text
Physical / Virtual Disks
          |
          v
       ZFS Pool
          |
          v
       Datasets
          |
          v
   File Permissions
          |
          v
    SMB File Shares
          |
          v
     Client Access
```

ZFS provides capabilities such as data-integrity checks, snapshots, and configurable redundancy.

However, the availability of these capabilities does not automatically mean that a particular pool has redundancy or that snapshots and backups are configured.

Those protections must be implemented and validated separately.

---

## SMB File Sharing

SMB is configured within the TrueNAS environment to provide network file-sharing capabilities.

This allows compatible Windows systems to access centralized storage resources over the lab network.

Conceptually:

```text
             Windows Endpoint
                    |
                    | SMB
                    v
              TrueNAS SCALE
                    |
                    v
               SMB Service
                    |
                    v
                Dataset
                    |
                    v
              Stored Files
```

SMB provides a useful platform for evaluating network file access, permissions, authentication, and client connectivity.

---

## Access Control

Storage security requires more than creating a network share.

Access should be controlled at multiple layers:

1. Network connectivity
2. SMB service configuration
3. Share permissions and restrictions
4. Dataset and filesystem permissions
5. User or group authentication

Conceptually:

```text
       Windows Client
             |
             v
       Network Access
             |
             v
        SMB Service
             |
             v
       Authentication
             |
             v
       Share Restrictions
             |
             v
      Filesystem Permissions
             |
             v
        Data Access
```

A client must satisfy the applicable access controls before performing an operation.

Permissions should be configured according to the intended access model rather than broadly allowing all users full control.

---

## Windows Client Integration

Windows systems within the lab can connect to SMB shares hosted by TrueNAS.

A typical UNC path follows this format:

```text
\\truenas.example.local\SharedData
```

The example above is illustrative and does not represent a published lab hostname.

### Validation Areas

- DNS resolution
- Network connectivity
- SMB service availability
- Authentication
- Read permissions
- Write permissions
- Access denial for unauthorized users
- File creation and deletion
- Share availability

Windows access can be tested using File Explorer or supported command-line utilities.

For example:

```powershell
Test-NetConnection truenas.example.local -Port 445
```

A successful TCP connection confirms basic reachability to the SMB service port, not successful authentication or file access.

---

## Authentication and Identity

TrueNAS supports different authentication models depending on deployment requirements.

Potential approaches include:

- Local TrueNAS users
- Local groups
- Active Directory integration

Because my lab also includes Active Directory Domain Services, storage authentication provides an opportunity to evaluate the relationship between centralized identity and file access.

Active Directory integration should be treated as a separate configuration and validation scenario rather than assumed to be enabled simply because AD exists in the environment.

---

## Network Architecture

TrueNAS operates within the virtual networking environment provided by Proxmox.

Network connectivity is an important part of the storage design because clients must be able to reach the SMB service without receiving unnecessary access to storage administration interfaces.

```text
                Windows Client
                      |
                      v
               Lab Network
                      |
                      v
              pfSense / Routing
                (if required)
                      |
                      v
                TrueNAS SCALE
                      |
               +------+------+
               |             |
               v             v
           SMB Access   Administration
```

Traffic between clients and TrueNAS may remain within the same network segment or traverse a firewall, depending on the actual topology.

Administrative access and file-sharing access should be considered separately when designing security policy.

---

## Storage Security

The storage implementation is evaluated using several security principles.

### Least Privilege

Users and systems should receive only the access required for their intended tasks.

### Controlled Administration

Administrative interfaces should be restricted to authorized users and management systems.

### Network Segmentation

Storage connectivity should not create unnecessary communication paths between isolated security zones.

### Data Integrity

Storage configuration should account for detecting and responding to data-integrity problems.

### Access Validation

Successful SMB connectivity should not be confused with authorization to access every dataset or share.

### Recovery Planning

Snapshots, backups, and redundancy serve different purposes and should be evaluated independently.

---

## Data Protection

TrueNAS SCALE provides storage capabilities that can support data protection and recovery strategies.

Relevant concepts include:

| Capability | Purpose |
|---|---|
| ZFS snapshots | Preserve point-in-time dataset states |
| ZFS checksums | Detect data corruption |
| Redundant pool layouts | Improve tolerance to certain disk failures |
| Replication | Copy supported snapshot data to another destination |
| Backups | Maintain recoverable copies of important data |
| Scrubs | Check stored data integrity and repair eligible errors where redundancy permits |

**Snapshots are not a replacement for independent backups.**

Similarly, storage redundancy protects against certain hardware failures but does not eliminate the need for backup and recovery planning.

---

## Operational Validation

A storage platform should be validated beyond confirming that its management interface is accessible.

The operational validation workflow includes:

```text
       TrueNAS VM Running
                |
                v
        Storage Pool Healthy
                |
                v
          Dataset Available
                |
                v
         SMB Service Running
                |
                v
         Client Connectivity
                |
                v
       Authentication Successful
                |
                v
        Permissions Enforced
                |
                v
       File Operations Validated
```

Each layer can fail independently.

This makes a layered validation approach useful when troubleshooting storage availability or access issues.

---

## Troubleshooting Methodology

When a Windows endpoint cannot access an SMB share, I evaluate the complete connection path.

### Network Layer

Confirm:

- Client network configuration
- DNS resolution
- TrueNAS IP reachability
- TCP 445 connectivity
- Applicable firewall rules

### Service Layer

Confirm:

- TrueNAS is operational
- SMB service is running
- Share is enabled
- Correct dataset path is configured

### Authentication Layer

Confirm:

- Correct user credentials
- Supported authentication method
- User account status
- Group membership where applicable

### Authorization Layer

Confirm:

- Share restrictions
- Dataset ACLs
- User and group permissions
- Effective access

### Storage Layer

Confirm:

- Pool availability
- Dataset availability
- Sufficient capacity
- Filesystem health

This separates network connectivity problems from authentication, authorization, service, and storage problems.

---

## Virtualization Considerations

Hosting TrueNAS inside Proxmox introduces architectural tradeoffs.

### Benefits

- Consolidated infrastructure
- Centralized VM management
- Flexible lab deployment
- Independent NAS operating system
- Integration with other virtualized workloads

### Considerations

- Dependence on Proxmox availability
- Storage controller and disk presentation
- Resource allocation
- Backup and recovery design
- Physical-disk failure handling
- Avoiding circular storage dependencies

For example, a virtualized NAS should not be the sole storage dependency required to boot the hypervisor or recover the NAS itself.

Where ZFS manages physical disks, direct disk or controller passthrough is often preferable to layering ZFS over opaque virtual disks, depending on hardware and operational requirements.

---

## Integration with the Broader Lab

TrueNAS complements the existing infrastructure and security components.

| Component | Relationship |
|---|---|
| Proxmox VE | Hosts the TrueNAS virtual machine |
| Windows endpoints | SMB clients |
| Active Directory | Potential centralized authentication integration |
| pfSense | Network segmentation and access control |
| Wazuh | Potential host/security telemetry integration |
| Qualys | Potential vulnerability assessment |
| Microsoft Defender for Endpoint | Security protection for supported Windows clients |

These relationships represent architectural capabilities. Specific integrations should be documented individually after configuration and validation.

---

## Architecture Principles

The storage environment follows several design principles.

**Separate storage from compute workloads.** Maintain a dedicated storage-service layer even when both operate on the same physical virtualization host.

**Enforce access controls.** Network reachability does not automatically justify access to stored data.

**Validate the complete data path.** Successful connectivity does not necessarily mean authentication, permissions, and file operations are working.

**Understand virtualization dependencies.** A virtualized NAS introduces additional considerations for disk presentation, availability, and recovery.

**Design for recoverability.** Redundancy, snapshots, replication, and backups should be evaluated as distinct controls.

**Document operational behavior.** Storage architecture should include troubleshooting and validation, not only deployment steps.

---

## Lessons Learned

Implementing TrueNAS SCALE within Proxmox reinforces that storage architecture includes several interconnected layers.

```text
Virtualization
      +
Disk Presentation
      +
Storage Pools
      +
Datasets
      +
Network Services
      +
Authentication
      +
Authorization
      +
Data Protection
      =
Storage Architecture
```

A functioning SMB share is only one component of a reliable storage implementation.

The broader architecture must also account for network connectivity, filesystem permissions, administrative security, data integrity, and recovery requirements.

Maintaining this environment provides an independent platform for evaluating how centralized storage services interact with Windows infrastructure and security controls.

---

## Future Improvements

Potential areas for continued development include:

- Active Directory authentication integration
- Additional SMB shares and access-control scenarios
- Group-based permissions
- ZFS snapshot testing
- Backup and restore validation
- Storage replication
- Storage monitoring
- Capacity and performance analysis
- Additional network segmentation
- SMB protocol security
- Storage recovery testing
- Expanded storage architecture diagrams

These improvements will be documented as they are implemented and validated.

---

## Planned Documentation

- [TrueNAS SCALE and SMB File Sharing](truenas-smb.md)
- Storage permissions and access validation
- Storage backup and recovery
- Storage security and troubleshooting

---
