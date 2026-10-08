# Pi-hole — DNS Filtering and Network Security

## Overview

I deployed **Pi-hole** within my Enterprise Security Home Lab to provide centralized DNS-based filtering, query visibility, and additional control over domain resolution for selected network clients.

The implementation operates alongside pfSense, Active Directory Domain Services, and the virtualized infrastructure hosted on Proxmox VE.

Pi-hole introduces a DNS security layer that can restrict resolution of unwanted domains while providing visibility into DNS activity.

The architecture is designed around several objectives:

- Centralized DNS filtering
- Domain-based access restrictions
- DNS query visibility
- Client resolver configuration
- Network segmentation
- DNS troubleshooting
- Integration with existing network services
- Security-control validation

The objective is not simply to block advertisements. Pi-hole provides a platform for evaluating DNS behavior, filtering effectiveness, network dependencies, and the limitations of DNS-based security controls.

---

## 1. Architecture Overview

Pi-hole operates as a DNS filtering service within the broader network-services architecture.

### Conceptual Topology

```text
                   Enterprise Security Lab
                             |
                        Proxmox VE
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
          pfSense          Pi-hole           DC01
             |               |               |
             v               v               v
       Firewall and      DNS Filtering    Active Directory
       Segmentation                       DNS / Identity
             |               |               |
             +---------------+---------------+
                             |
                             v
                       Lab Clients
```

Each component performs a distinct role.

**Pi-hole** evaluates DNS requests against configured filtering policies.

**pfSense** controls network routing and firewall policy.

**Active Directory DNS** supports domain-integrated name resolution and service discovery.

These services complement one another but are not interchangeable.

---

## 2. DNS Resolution Workflow

When a client uses Pi-hole as its DNS resolver, DNS queries are evaluated against the configured filtering rules.

```text
               Windows / Linux Client
                         |
                         v
                      Pi-hole
                         |
                 DNS Query Evaluation
                         |
              +----------+----------+
              |                     |
              v                     v
         Blocked Domain        Allowed Domain
              |                     |
              v                     v
         Block Response       Upstream Resolver
                                    |
                                    v
                               DNS Response
                                    |
                                    v
                                  Client
```

Blocked requests are answered according to Pi-hole's configured blocking mode.

Allowed queries are resolved through the configured DNS resolution path, which may include cache responses or upstream resolvers.

---

## 3. Pi-hole Deployment

Pi-hole can be deployed on a supported Linux host or through a container-based architecture.

The deployment should provide a stable network identity because clients depend on the resolver's availability.

### Infrastructure Requirements

- Supported Linux environment
- Stable IP address
- Reliable DNS connectivity
- Appropriate network access
- Sufficient compute and memory resources
- Administrative access
- Upstream DNS resolution

### Installation Example

For a supported Linux deployment, Pi-hole provides an official installation method:

```bash
curl -sSL https://install.pi-hole.net | bash
```

For security-conscious deployments, the installation script should be reviewed and obtained from the official project before execution.

The exact deployment method and operating system should be documented alongside the actual lab configuration.

### Post-Deployment Validation

After installation, verify the Pi-hole service and configuration.

```bash
pihole status
```

Check the installed version:

```bash
pihole -v
```

Verify that the DNS service is listening:

```bash
sudo ss -lntup
```

These checks provide initial operational visibility but do not replace client-side DNS testing.

---

## 4. DNS Filtering Configuration

Pi-hole evaluates domain-resolution requests using its configured filtering database and policies.

Filtering sources can include:

- Adlists
- Exact domain deny rules
- Regex-based filtering
- Domain allow rules
- Client and group-specific policies

The filtering configuration should be designed to reduce unwanted DNS resolution without unnecessarily disrupting legitimate applications.

### Example Filtering Flow

```text
DNS Query
    |
    v
Pi-hole Filtering Engine
    |
    v
Evaluate Applicable Policies
    |
    +------> Blocked
    |           |
    |           v
    |      Block Response
    |
    +------> Allowed
                |
                v
           DNS Resolution
```

The actual result depends on the effective policy, client group, and DNS configuration.

---

## 5. Blocklist Management

Blocklists provide a scalable method for restricting resolution of known unwanted domains.

However, increasing the number of blocklists does not automatically improve security.

Excessive or poorly maintained lists can introduce:

- False positives
- Duplicate entries
- Application compatibility issues
- Additional troubleshooting overhead
- Unnecessary policy complexity

### Management Principles

1. Select reputable and maintained blocklist sources.
2. Avoid unnecessary overlap.
3. Validate filtering behavior.
4. Review legitimate domains that are unexpectedly blocked.
5. Document exceptions.
6. Periodically review the effectiveness of the filtering configuration.

### Updating the Filtering Database

```bash
pihole -g
```

This updates the gravity database using the configured lists and filtering configuration.

---

## 6. Allow and Deny Rules

Pi-hole supports domain-level allow and deny controls.

These are useful when a domain requires a specific filtering decision.

### Example Scenarios

| Scenario | Intended action |
|---|---|
| Known unwanted domain | Deny |
| Legitimate application domain incorrectly blocked | Review and allow if appropriate |
| Test domain | Apply a controlled rule |
| Domain requiring different policies by client | Evaluate group-specific filtering |

Exceptions should be narrow and justified.

Broadly disabling DNS filtering to resolve one application issue weakens the intended security control.

---

## 7. Client DNS Configuration

For Pi-hole to evaluate DNS requests, clients must direct their queries through the intended DNS resolution path.

Client configuration may be applied through:

- DHCP
- Static network configuration
- Operating-system DNS settings
- Network management systems

### Windows Validation

```powershell
Get-DnsClientServerAddress
```

Review the DNS server addresses assigned to the active network interface.

### Linux Validation

```bash
resolvectl status
```

Depending on the Linux distribution and resolver configuration, additional tools may be required.

### Architectural Consideration

A client configured to use an external DNS resolver directly may bypass Pi-hole filtering.

Therefore, successful Pi-hole deployment requires both resolver configuration and validation of the actual client DNS path.

---

## 8. pfSense Integration

pfSense provides network segmentation and firewall enforcement within the lab.

Depending on the selected architecture, pfSense can distribute Pi-hole as a DNS resolver through DHCP or control which network segments can access the DNS service.

### Reference Architecture

```text
                     pfSense
                        |
                 DHCP / Routing
                        |
             +----------+----------+
             |                     |
             v                     v
        Lab Network           Test Network
             |                     |
             v                     v
        DNS Clients            DNS Clients
             |                     |
             +----------+----------+
                        |
                        v
                     Pi-hole
```

This is a reference design. The actual DHCP and DNS configuration should be verified before documenting it as implemented.

### Firewall Considerations

Typical DNS traffic uses:

- UDP 53
- TCP 53

TCP DNS must not be overlooked because it is also used for normal DNS operations.

Firewall policy should permit required DNS traffic while restricting unauthorized access to internal resolver services.

---

## 9. Active Directory DNS Integration

The lab also includes Active Directory Domain Services hosted on DC01.

Active Directory relies on DNS for domain controller discovery, authentication services, and internal name resolution.

Domain-joined systems should use a DNS design that reliably resolves Active Directory records.

### Reference Architecture

```text
              Domain-Joined Windows Client
                          |
                          v
                    AD DNS (DC01)
                          |
              +-----------+-----------+
              |                       |
              v                       v
       Internal AD Records       External Queries
              |                       |
              v                       v
        AD DNS Resolution            Pi-hole
                                      |
                                      v
                               Upstream Resolver
```

In this design, Active Directory DNS remains responsible for internal domain resolution while forwarding appropriate external queries through Pi-hole.

This configuration is an architectural option, not a confirmed description of the current forwarding setup.

### Important Design Considerations

- Avoid DNS forwarding loops.
- Preserve Active Directory SRV record resolution.
- Validate domain controller discovery.
- Confirm external DNS filtering.
- Avoid configuring public DNS as an alternate resolver on domain-joined endpoints when it can disrupt AD service discovery.

---

## 10. DNS Security and Bypass Considerations

DNS filtering is a useful security layer, but it does not prevent every form of unwanted communication.

Potential bypass methods include:

- Direct communication using IP addresses
- Alternate external DNS resolvers
- DNS over HTTPS
- DNS over TLS
- Application-specific resolver behavior
- Cached DNS responses

### Defense-in-Depth Model

```text
                  Client Activity
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
       Pi-hole        pfSense       Endpoint Security
          |             |             |
          v             v             v
     DNS Filtering   Network Policy   Host Protection
          |             |             |
          +-------------+-------------+
                        |
                        v
                Layered Security
```

Pi-hole should complement firewall and endpoint controls rather than replace them.

---

## 11. DNS Query Logging

Pi-hole provides query visibility that can support operational troubleshooting and security analysis.

Useful information can include:

- Queried domains
- Query timestamps
- Client identification
- Query types
- Filtering decisions
- Upstream resolution behavior

### Operational Use Cases

- Identify unexpectedly blocked domains.
- Validate that clients are using Pi-hole.
- Investigate application DNS failures.
- Review unusual query patterns.
- Confirm filtering behavior after policy changes.

DNS query logs should be treated as potentially sensitive because they can reveal user and device activity.

Retention and access should be configured according to the environment's requirements.

---

## 12. Filtering Validation

A DNS filtering configuration should be tested from the client perspective.

### Step 1 — Confirm Resolver Assignment

On Windows:

```powershell
Get-DnsClientServerAddress
```

### Step 2 — Test a Known Allowed Domain

```powershell
Resolve-DnsName example.com
```

### Step 3 — Query Pi-hole Directly

```powershell
Resolve-DnsName example.com -Server 192.0.2.53
```

The IP address is a documentation-only placeholder.

### Step 4 — Validate a Controlled Block Rule

Create a temporary deny rule for a dedicated test domain and query that domain against Pi-hole.

### Step 5 — Review Query Logs

Confirm that the request appears in the Pi-hole query log with the expected filtering decision.

### Step 6 — Remove the Temporary Test Rule

Return the environment to its intended configuration.

This validates the filtering engine and the client DNS path without relying exclusively on dashboard statistics.

---

## 13. Troubleshooting — Client Not Using Pi-hole

### Scenario

A client has network connectivity but does not appear in Pi-hole query logs.

### Potential Causes

- Incorrect DHCP DNS assignment
- Static DNS configuration
- Alternate DNS resolver
- VPN-provided DNS
- DNS over HTTPS
- Client DNS caching
- Firewall restrictions
- Incorrect network interface configuration

### Troubleshooting Workflow

```text
Client DNS Configuration
          |
          v
DHCP / Static Settings
          |
          v
Actual Resolver Used
          |
          v
Network Reachability
          |
          v
Pi-hole Query Log
          |
          v
Filtering Validation
```

The objective is to confirm that DNS requests actually reach Pi-hole before troubleshooting blocklists.

---

## 14. Troubleshooting — DNS Resolution Failure

### Scenario

A client cannot resolve external domains.

### Validation Steps

1. Confirm client DNS configuration.
2. Test connectivity to Pi-hole.
3. Verify the Pi-hole DNS service.
4. Review query logs.
5. Test upstream DNS resolution.
6. Review firewall policy.
7. Check for forwarding loops.
8. Validate the DNS response.

### Linux Commands

```bash
pihole status
```

```bash
dig example.com
```

```bash
dig @127.0.0.1 example.com
```

The loopback test is applicable when executed on the Pi-hole host and its DNS service is configured to listen on the relevant loopback interface.

---

## 15. Troubleshooting — Legitimate Domain Blocked

### Scenario

An application fails because Pi-hole blocks a required domain.

### Investigation

1. Reproduce the application failure.
2. Identify the client and time of the request.
3. Review Pi-hole query logs.
4. Identify the blocked domain.
5. Determine which filtering rule applies.
6. Validate whether the domain is required.
7. Create a narrowly scoped exception if justified.
8. Retest the application.

### Key Principle

A filtering exception should be based on evidence rather than automatically allowing every domain requested by an application.

---

## 16. Troubleshooting — Active Directory Issues

### Scenario

A domain-joined Windows client has internet access but cannot locate a domain controller.

### Potential Cause

The endpoint may be using a resolver that cannot resolve the Active Directory DNS zone or its required service records.

### Validation

```cmd
nltest /dsgetdc:example.local
```

Check domain controller service records:

```powershell
Resolve-DnsName `
  -Name "_ldap._tcp.dc._msdcs.example.local" `
  -Type SRV
```

Review the configured DNS servers:

```powershell
Get-DnsClientServerAddress
```

The domain is illustrative.

Successful public DNS resolution does not prove that Active Directory DNS is functioning.

---

## 17. Troubleshooting — DNS Forwarding Loop

### Scenario

DNS requests fail or repeatedly circulate between resolvers.

### Example of an Incorrect Design

```text
       Active Directory DNS
                |
                v
              Pi-hole
                |
                v
       Active Directory DNS
                |
                v
          Forwarding Loop
```

This can occur when resolvers forward the same query back to one another without a valid resolution path.

### Recommended Approach

Clearly define which resolver is authoritative for internal zones and where external queries should be forwarded.

Test internal and external resolution separately.

---

## 18. Operational Validation Matrix

| Test | Expected result |
|---|---|
| Pi-hole service | Running |
| DNS listener | Available on intended interfaces |
| Allowed domain | Resolves successfully |
| Blocked test domain | Filtered according to policy |
| Windows client | Uses intended DNS path |
| Linux client | Uses intended DNS path |
| Query logging | Requests visible when logging is enabled |
| Upstream DNS | External queries resolve |
| AD DNS | Domain records resolve correctly |
| pfSense firewall | Required DNS traffic permitted |
| Unauthorized access | Restricted according to policy |

Actual results should be recorded separately from expected outcomes.

---

## 19. Security Considerations

### Administrative Access

Restrict the Pi-hole management interface to authorized systems and administrators.

### DNS Exposure

Avoid unintentionally exposing the resolver to untrusted external networks.

### Network Segmentation

Permit only the DNS communication required by the intended client networks.

### Updates

Maintain the Pi-hole software and underlying operating system.

### Logging

Protect DNS query logs and configure appropriate retention.

### DNS Bypass

Understand the limitations of DNS filtering when clients use alternate resolvers or encrypted DNS.

### Availability

DNS is a critical dependency. Resolver outages can affect application access and general network connectivity.

---

## 20. Architecture Decisions and Tradeoffs

| Design decision | Consideration |
|---|---|
| Centralized Pi-hole | Consistent DNS filtering |
| Client DNS assignment | Determines filtering coverage |
| pfSense integration | Controls DNS network access |
| Active Directory DNS | Preserves domain service discovery |
| Blocklist selection | Balances filtering and false positives |
| Query logging | Improves visibility but introduces privacy considerations |
| Upstream DNS | Influences resolution behavior and availability |
| DNS redundancy | Reduces dependence on a single resolver |

The implementation should balance security, performance, compatibility, and operational reliability.

---

## 21. Lessons Learned

Deploying Pi-hole demonstrates that DNS filtering depends on more than installing a resolver and enabling blocklists.

The complete implementation requires:

```text
DNS Service
     +
Client Configuration
     +
Network Connectivity
     +
Filtering Policy
     +
Upstream Resolution
     +
Logging
     +
Security Controls
     +
Validation
     =
Effective DNS Filtering
```

One of the most important architectural considerations is understanding the relationship between Pi-hole and Active Directory DNS.

DNS filtering should not interfere with the internal name resolution required by domain-integrated Windows systems.

The lab provides a controlled environment for evaluating DNS security, resolver behavior, network policy, and operational troubleshooting.

---

## 22. Future Improvements

Potential improvements include:

- Additional blocklist evaluation
- Group-based DNS filtering
- DNS forwarding validation
- Active Directory DNS integration
- DNS redundancy
- Resolver availability monitoring
- DNS over HTTPS bypass testing
- DNS over TLS testing
- Firewall-based DNS restrictions
- Query-log analysis
- DNS anomaly detection
- Automated resolver testing
- Expanded DNS security documentation

These capabilities will be documented as they are implemented and validated.

---

## Related Documentation

- [Network Services Architecture](README.md)
- [Network Security Architecture](../network-security/README.md)
- [pfSense Network Segmentation](../network-security/pfsense-segmentation.md)
- [Active Directory Domain Services](../identity/active-directory.md)
- [Hybrid Identity Architecture](../identity/hybrid-identity.md)
- [Proxmox VE Infrastructure](../proxmox/README.md)
- [Security Monitoring Architecture](../security-monitoring/README.md)
