# TrueNAS SCALE — SMB File Sharing and Storage Implementation

## Overview

I deployed **TrueNAS SCALE** as a virtual machine within my Proxmox-based Enterprise Security Home Lab to provide centralized network-attached storage (NAS) and SMB file-sharing services.

The implementation provides a dedicated storage platform for testing file access, permissions, network connectivity, and integration with Windows infrastructure.

Rather than treating TrueNAS as a standalone file server, the architecture considers the complete storage-access path, including virtualization, storage pools, datasets, SMB services, authentication, authorization, and network security.

### Implementation Scope

- TrueNAS SCALE deployment within Proxmox VE
- ZFS-based storage architecture
- SMB file-sharing configuration
- Windows client connectivity
- Dataset and filesystem permissions
- User and group access management
- Network segmentation considerations
- SMB connectivity troubleshooting
- Storage service validation
- Backup and recovery design considerations

The environment provides a controlled platform for evaluating storage architecture without depending on production infrastructure.

---

## 1. Architecture Overview

TrueNAS SCALE operates as a dedicated virtual machine hosted on my Dell PowerEdge R630 running Proxmox VE.

```text
                   Dell PowerEdge R630
                           |
                       Proxmox VE
                           |
                   TrueNAS SCALE VM
                           |
                   ZFS Storage Pool
                           |
                       Dataset
                           |
                      SMB Share
                           |
                    Lab Network
                           |
                +----------+----------+
                |                     |
                v                     v
           Windows 10             Windows 11
```

TrueNAS provides the storage services while Windows systems operate as SMB clients.

The implementation separates storage management from endpoint workloads, allowing the NAS to be administered independently.

---

## 2. TrueNAS Virtual Machine Deployment

TrueNAS SCALE was deployed as a dedicated virtual machine within Proxmox.

The general deployment workflow involves:

1. Obtain the TrueNAS SCALE installation ISO.
2. Upload the ISO to the appropriate Proxmox storage.
3. Create a dedicated virtual machine.
4. Assign CPU and memory resources.
5. Configure the boot disk.
6. Configure storage-disk presentation.
7. Attach the virtual network interface.
8. Install TrueNAS SCALE.
9. Configure management networking.
10. Access the TrueNAS administrative interface.

### Virtualization Considerations

Hosting a NAS within a hypervisor introduces additional dependencies.

Important considerations include:

- CPU and memory allocation
- Boot disk separation
- Data disk presentation
- Storage controller configuration
- Network interface configuration
- Underlying storage reliability
- Recovery dependencies

For production-oriented ZFS architectures, direct access to physical disks or a dedicated storage controller is generally preferable where supported.

For lab environments, virtual disks can provide flexibility, but their limitations should be understood.

**Design consideration:** The TrueNAS VM should not depend exclusively on storage services that require the same VM to be running.

This avoids circular storage dependencies during startup and recovery.

---

## 3. Storage Pool Architecture

TrueNAS SCALE uses OpenZFS for storage management.

The general hierarchy is:

```text
Storage Disks
     |
     v
ZFS Pool
     |
     v
Dataset
     |
     v
Filesystem Permissions
     |
     v
SMB Share
     |
     v
Windows Client
```

A storage pool provides the underlying storage capacity.

Datasets provide logical separation within the pool and allow storage properties and permissions to be managed independently.

### Design Considerations

- Pool layout
- Storage redundancy
- Dataset separation
- Available capacity
- Data integrity
- Snapshot requirements
- Recovery strategy

A ZFS pool does not automatically provide redundancy. Fault tolerance depends on the selected vdev layout and underlying storage design.

---

## 4. Creating an SMB Dataset

SMB shares should be backed by appropriately configured datasets.

### General Configuration Workflow

1. Sign in to the TrueNAS SCALE administrative interface.
2. Navigate to **Datasets**.
3. Select the intended storage pool.
4. Create a new dataset.
5. Assign a descriptive dataset name.
6. Select the SMB-oriented dataset preset when available and appropriate.
7. Review the dataset properties.
8. Save the configuration.

### Example Dataset Structure

```text
tank/
├── shared/
├── engineering/
└── backups/
```

These names are illustrative and do not disclose the actual storage configuration.

Separating datasets allows different access requirements and storage settings to be applied to different data categories.

---

## 5. Configuring SMB File Sharing

After creating the dataset, configure the SMB share.

### Configuration Workflow

1. Navigate to **Shares**.
2. Locate **Windows (SMB) Shares**.
3. Select **Add**.
4. Choose the dataset path.
5. Configure the SMB share name.
6. Review share settings.
7. Save the share.
8. Enable or start the SMB service if required.
9. Verify the share is available.

### Example

```text
Dataset Path:
  /mnt/tank/shared

SMB Share Name:
  SharedData

Windows UNC Path:
  \\truenas.example.local\SharedData
```

The examples above use placeholder names.

### SMB Service

The SMB service must be running for clients to connect to the share.

The presence of a configured share does not guarantee that the SMB service is available or that client authentication will succeed.

---

## 6. SMB Authentication

TrueNAS supports multiple authentication approaches depending on the deployment.

Common approaches include:

- Local TrueNAS users
- Local TrueNAS groups
- Active Directory integration

For standalone SMB configurations, local accounts can provide access without requiring an Active Directory domain.

### Local User Configuration

A typical workflow includes:

1. Navigate to **Credentials → Local Users** or the corresponding user-management area for the installed version.
2. Create a user account.
3. Configure authentication credentials.
4. Ensure the account is permitted to authenticate to SMB.
5. Assign the appropriate group membership.
6. Configure dataset permissions.
7. Validate authentication from a Windows endpoint.

The exact navigation and available options depend on the TrueNAS SCALE release.

---

## 7. Dataset Permissions and ACLs

SMB security depends on the effective filesystem permissions.

A share may be reachable over the network while still denying file access because of dataset permissions.

### Permission Model

```text
Windows Client
      |
      v
SMB Authentication
      |
      v
Share-Level Restrictions
      |
      v
Dataset ACL
      |
      v
Effective Permissions
      |
      v
Read / Write / Deny
```

### Common Permission Types

| Permission | Function |
|---|---|
| Read | View files and folders |
| Modify | Read, create, edit, and delete within granted scope |
| Full Control | Broad file and permission-management capabilities |
| Deny | Explicitly restrict selected access |

Permissions should be assigned to appropriate users or groups.

Broad access should not be granted merely to simplify troubleshooting.

### Example Access Design

| Group | Share | Intended access |
|---|---|---|
| Storage-Admins | Engineering | Full Control |
| Engineering-Users | Engineering | Modify |
| Engineering-Readers | Engineering | Read |
| Unapproved Users | Engineering | No access |

This is an example of a group-based access model, not a claim about the current lab ACL configuration.

---

## 8. Windows Client Integration

Windows clients can access SMB shares using a UNC path.

### File Explorer

```text
\\truenas.example.local\SharedData
```

Alternatively, a network drive can be mapped to provide a persistent drive letter.

### PowerShell Example

```powershell
New-PSDrive `
  -Name "S" `
  -PSProvider FileSystem `
  -Root "\\truenas.example.local\SharedData" `
  -Persist
```

This example uses the current Windows security context.

Authentication requirements depend on the SMB server configuration and the identity used by the client.

### Validation

Confirm that the client can:

- Resolve the TrueNAS hostname.
- Reach the SMB service.
- Authenticate successfully.
- Browse the intended share.
- Create files when permitted.
- Modify files when permitted.
- Delete files when permitted.
- Receive access-denied responses when appropriate.

---

## 9. Network Connectivity Validation

Before troubleshooting permissions, confirm the underlying network path.

### DNS Resolution

```powershell
Resolve-DnsName truenas.example.local
```

### SMB Port Connectivity

```powershell
Test-NetConnection truenas.example.local -Port 445
```

### Existing SMB Connections

```powershell
Get-SmbConnection
```

### Network Configuration

```powershell
ipconfig /all
```

A successful TCP 445 test confirms that the SMB port is reachable.

It does not confirm successful authentication, correct ACLs, or successful file operations.

---

## 10. SMB Security

SMB provides centralized file access, but it also introduces a network service that requires protection.

### Security Considerations

- Restrict SMB access to authorized network segments.
- Use supported SMB protocol versions.
- Avoid enabling SMB1.
- Require authenticated access where appropriate.
- Apply least-privilege permissions.
- Restrict administrative access.
- Maintain TrueNAS updates.
- Protect credentials.
- Review share configuration.
- Consider SMB encryption and signing requirements.

### SMB1

SMB1 is a legacy protocol and should remain disabled unless an exceptional compatibility requirement has been explicitly assessed.

Modern SMB implementations should use supported SMB2/SMB3 capabilities.

### SMB Signing and Encryption

SMB signing provides message integrity and helps protect against certain tampering attacks.

SMB encryption provides confidentiality for supported SMB sessions.

These controls have different security purposes and should be configured according to client compatibility, performance requirements, and the threat model.

---

## 11. Network Segmentation

The storage environment operates within the broader Proxmox networking architecture.

pfSense provides routing and firewall enforcement between applicable security zones.

### Conceptual Design

```text
               Windows Endpoint
                      |
                      v
                 Lab Network
                      |
                      v
              Firewall / Routing
                 If Required
                      |
                      v
                 TrueNAS SCALE
                      |
                      v
                   SMB Share
```

If the client and NAS reside within the same network segment, their traffic may not traverse pfSense.

If communication crosses routed network boundaries, firewall policy may control access.

### Example Policy

```text
Authorized Windows Clients
          |
          | TCP 445
          v
      TrueNAS SMB

Untrusted Security VLAN
          |
          X
          |
      TrueNAS SMB
```

The objective is to provide the required storage connectivity without exposing file shares unnecessarily to security-testing workloads.

---

## 12. Active Directory Integration

The broader home lab includes Active Directory Domain Services running on DC01.

TrueNAS can integrate with Active Directory to support centralized authentication and group-based access control.

### Conceptual Architecture

```text
               Active Directory
                      |
                      v
                    DC01
                      |
               Domain Identities
                      |
                      v
                 TrueNAS SCALE
                      |
                      v
                 SMB Dataset
                      |
                      v
                Windows Clients
```

Active Directory integration can provide centralized account and group management.

However, domain integration must be configured and validated separately.

The existence of Active Directory and TrueNAS within the same environment does not automatically mean that TrueNAS is domain joined.

### Potential Validation Areas

- DNS configuration
- Domain controller discovery
- Time synchronization
- Domain join status
- User authentication
- Group resolution
- Dataset ACLs
- SMB access
- Domain credential behavior

---

## 13. Troubleshooting — Share Not Accessible

### Scenario

A Windows endpoint cannot access the TrueNAS SMB share.

### Troubleshooting Sequence

```text
Windows Endpoint
      |
      v
DNS Resolution
      |
      v
Network Connectivity
      |
      v
TCP 445
      |
      v
SMB Service
      |
      v
Share Configuration
      |
      v
Authentication
      |
      v
Dataset ACL
      |
      v
File Access
```

### Common Causes

- Incorrect hostname
- DNS resolution failure
- Firewall restriction
- SMB service stopped
- Incorrect share path
- Incorrect credentials
- User not permitted for SMB access
- Missing dataset permissions
- Client-side cached credentials
- Unsupported protocol configuration

Troubleshooting should begin with connectivity and progress toward authentication and authorization.

---

## 14. Troubleshooting — Access Denied

### Scenario

The Windows endpoint can reach the share but receives an access-denied message.

This typically indicates that the network path is functioning, but access is restricted at another layer.

### Validation Steps

1. Confirm the account used for SMB authentication.
2. Review the user's group membership.
3. Check applicable share restrictions.
4. Review the dataset ACL.
5. Confirm inherited permissions.
6. Check whether explicit deny entries apply.
7. Validate effective access.
8. Retest with an authorized test account.

### Important Distinction

```text
Network Reachability
        ≠
Authentication Success
        ≠
Authorization Success
```

Each condition must be validated independently.

---

## 15. Troubleshooting — Credential Conflicts

Windows may retain existing SMB sessions or credentials for a network server.

This can cause unexpected authentication behavior when switching between test accounts.

### Review SMB Connections

```powershell
Get-SmbConnection
```

### Review Mapped Drives

```cmd
net use
```

### Disconnect a Specific Mapping

```cmd
net use S: /delete
```

Avoid clearing all SMB sessions unnecessarily, particularly on systems that may have other active file-sharing connections.

After removing the relevant connection, reconnect using the intended credentials.

---

## 16. Troubleshooting — SMB Service Unavailable

If TCP 445 is not reachable, validate:

- TrueNAS VM power state
- TrueNAS network configuration
- SMB service state
- Share configuration
- Proxmox virtual networking
- Applicable firewall rules
- Routing between network segments

A storage-service problem should not be confused with a filesystem-permissions problem.

If the client cannot reach the SMB service, changing dataset ACLs will not resolve the underlying connectivity issue.

---

## 17. Storage Health Validation

File-sharing availability depends on the health of the underlying storage.

Operational checks should include:

- ZFS pool status
- Dataset availability
- Storage capacity
- Disk health
- SMB service health
- TrueNAS system alerts
- Network connectivity
- Client access

### ZFS Pool Status

From an authorized TrueNAS shell:

```bash
zpool status
```

### Storage Capacity

```bash
zpool list
```

### Dataset Information

```bash
zfs list
```

These commands provide useful storage information, although administrative changes should follow supported TrueNAS management workflows.

---

## 18. Snapshots and Data Protection

TrueNAS SCALE supports ZFS snapshots and other data-protection capabilities.

Snapshots can preserve point-in-time filesystem states and support recovery from certain accidental modifications or deletions.

### Conceptual Recovery Model

```text
Dataset
   |
   v
Snapshot
   |
   v
Data Modified / Deleted
   |
   v
Recovery Required
   |
   v
Restore From Snapshot
```

Snapshots do not replace independent backups.

If the underlying storage pool becomes unavailable, snapshots stored only within that pool may also become inaccessible.

A complete recovery strategy should consider:

- Snapshot retention
- Independent backups
- Backup destination
- Recovery testing
- Storage redundancy
- Disaster recovery requirements

---

## 19. Operational Validation Matrix

The following matrix can be used to validate the implementation.

| Test | Expected result |
|---|---|
| TrueNAS VM starts | System boots successfully |
| ZFS pool status | Pool reports expected health |
| SMB service | Service is running |
| DNS resolution | Client resolves the NAS hostname |
| TCP 445 connectivity | SMB service is reachable |
| Authorized authentication | User can authenticate |
| Authorized read | Files can be read |
| Authorized write | Files can be created or modified |
| Unauthorized access | Access is denied |
| SMB reconnect | Client reconnects successfully |
| Service restart | Share becomes available again |
| Snapshot recovery | Recovery succeeds when configured and tested |

The matrix provides a repeatable validation framework. Test outcomes should be recorded against the actual environment rather than assumed.

---

## 20. Architecture Decisions and Tradeoffs

| Design decision | Consideration |
|---|---|
| Virtualized TrueNAS | Infrastructure consolidation and flexibility |
| Dedicated NAS VM | Separation of storage services from clients |
| ZFS datasets | Logical separation and independent storage settings |
| SMB | Native integration with Windows clients |
| Group-based permissions | Scalable authorization |
| Network segmentation | Reduced unnecessary exposure |
| Direct disk access | Improved visibility into underlying storage hardware where supported |
| Snapshots | Point-in-time recovery capability |
| Independent backups | Protection beyond the local storage pool |

These decisions help balance operational flexibility, security, reliability, and maintainability.

---

## 21. Lessons Learned

Deploying TrueNAS SCALE within Proxmox demonstrates that a functioning network share is only one component of storage architecture.

A complete implementation requires several dependencies to operate correctly:

```text
Virtualization
      +
Storage Pool
      +
Dataset
      +
SMB Service
      +
Network Connectivity
      +
Authentication
      +
Authorization
      +
Data Protection
      =
Reliable File Sharing
```

One of the most important operational distinctions is separating network connectivity, authentication, and filesystem authorization.

A client may successfully connect to TCP 445 while still being unable to authenticate or access the requested dataset.

Understanding these layers allows storage issues to be isolated and resolved systematically rather than making unrelated configuration changes.

The environment also provides a platform for evaluating how centralized storage interacts with Windows infrastructure, identity services, and network-security controls.

---

## 22. Future Improvements

Potential areas for additional implementation and testing include:

- Active Directory integration
- Group-based SMB permissions
- Additional datasets and shares
- SMB signing and encryption validation
- Snapshot schedules
- Backup and restore testing
- Storage replication
- Storage performance analysis
- Dataset quotas
- Storage monitoring and alerting
- Recovery scenarios
- Expanded storage-security testing

These capabilities will be documented as they are implemented and validated.

---

## Related Documentation

- [Storage Architecture](README.md)
- [Proxmox VE Infrastructure](../proxmox/README.md)
- [Network Security Architecture](../network-security/README.md)
- [pfSense Network Segmentation](../network-security/pfsense-segmentation.md)
- [Active Directory Domain Services](../identity/active-directory.md)
- [Hybrid Identity Architecture](../identity/hybrid-identity.md)
