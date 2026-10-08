# Network Services Architecture

## Overview

The Network Services component of my Enterprise Security Home Lab focuses on implementing and evaluating foundational network services that support infrastructure connectivity, name resolution, security, and centralized network management.

The environment includes a **Pi-hole DNS filtering deployment** alongside other infrastructure components hosted within my Proxmox environment.

The broader architecture incorporates:

- Pi-hole for DNS-based filtering
- pfSense for routing, firewall enforcement, and network segmentation
- Active Directory DNS for domain-integrated name resolution
- Proxmox VE for virtualized infrastructure
- Windows and Linux systems for client and service validation

The objective is to evaluate how network services interact across different security zones while maintaining reliable connectivity, appropriate access controls, and visibility into network behavior.

---

## 1. Network Services Architecture

The lab uses multiple infrastructure components with distinct responsibilities.

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
       Routing and       DNS Filtering    Active Directory
       Firewalling                        DNS / Identity
             |               |               |
             +---------------+---------------+
                             |
                             v
                      Lab Endpoints
```

Each service performs a specific function within the architecture.

The goal is to avoid unnecessary overlap while ensuring that required network and identity services remain available.

---

## 2. Core Network Services

| Component | Primary responsibility |
|---|---|
| Pi-hole | DNS-based domain filtering |
| pfSense | Routing, segmentation, and firewall policy |
| Active Directory DNS | Domain name resolution and AD service discovery |
| Proxmox VE | Hosting virtualized network services |
| Windows endpoints | Client DNS and connectivity validation |
| Linux endpoints | Client DNS and network testing |

These technologies complement one another but should not be treated as interchangeable.

---

## 3. Pi-hole DNS Filtering

Pi-hole provides DNS-based filtering capabilities within the lab.

It can evaluate DNS queries against configured blocklists and filtering policies before forwarding permitted queries to an upstream DNS resolver.

### Conceptual DNS Flow

```text
             Windows / Linux Client
                       |
                       v
                    Pi-hole
                       |
              +--------+--------+
              |                 |
              v                 v
         Blocked Domain     Allowed Domain
              |                 |
              v                 v
        Block Response     Upstream Resolver
                                |
                                v
                          DNS Response
```

This provides centralized control over selected domain-resolution requests.

### Security Use Cases

- Blocking known advertising domains
- Filtering tracking domains
- Restricting selected unwanted domains
- Reviewing DNS query activity
- Evaluating blocklist effectiveness
- Testing DNS filtering policies
- Troubleshooting DNS resolution

DNS filtering provides an additional security layer but does not replace endpoint protection, firewalling, or network intrusion detection.

---

## 4. DNS Filtering vs. Network Enforcement

An important architectural distinction is the difference between DNS filtering and firewall enforcement.

**Pi-hole** evaluates DNS queries and applies domain-based filtering.

**pfSense** controls network communication through routing and firewall policies.

```text
                    Client Request
                          |
              +-----------+-----------+
              |                       |
              v                       v
           Pi-hole                 pfSense
              |                       |
              v                       v
        DNS Filtering           Network Policy
              |                       |
              v                       v
        Allow / Block           Allow / Deny
        DNS Resolution          Network Traffic
```

A blocked DNS request does not necessarily prevent communication with a destination through another resolver or a directly specified IP address.

Therefore, DNS filtering should be considered one layer of a broader defense-in-depth architecture.

---

## 5. Active Directory DNS

The environment also includes a Windows Server domain controller, DC01, which provides Active Directory Domain Services.

Active Directory relies heavily on DNS for service discovery and authentication.

Domain-integrated Windows systems require appropriate DNS configuration to locate domain controllers and other Active Directory services.

### Active Directory DNS Flow

```text
               Domain-Joined Client
                        |
                        v
                Active Directory DNS
                        |
                        v
                 Locate AD Services
                        |
                        v
                       DC01
                        |
                        v
                Domain Authentication
```

Active Directory DNS supports functionality such as:

- Domain controller discovery
- Kerberos service discovery
- LDAP service discovery
- Internal hostname resolution
- Active Directory service records
- Domain-integrated DNS zones

This makes DNS a critical dependency for the identity environment.

---

## 6. Pi-hole and Active Directory DNS Integration

Pi-hole and Active Directory DNS can coexist, but the DNS forwarding design must be carefully considered.

A domain-joined Windows client should be able to locate Active Directory services reliably.

A common architecture is to configure domain-joined systems to use AD DNS while forwarding appropriate external DNS queries toward a filtering resolver.

### Example Architecture

```text
                Domain-Joined Client
                         |
                         v
                   AD DNS (DC01)
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Internal AD Zone        External DNS Query
             |                       |
             v                       v
       Local Resolution             Pi-hole
                                     |
                                     v
                              Upstream Resolver
```

This model allows Active Directory DNS to remain responsible for internal domain resolution while Pi-hole provides filtering for forwarded external queries.

**Important:** This is a reference architecture, not a claim that the lab currently uses this exact forwarding configuration.

The forwarding chain must be designed to avoid DNS loops, and internal Active Directory records must remain resolvable.

---

## 7. DNS Architecture for Non-Domain Devices

Devices that do not depend on Active Directory may use Pi-hole directly, depending on network design.

```text
               Non-Domain Endpoint
                        |
                        v
                     Pi-hole
                        |
                        v
                 DNS Filtering
                        |
                        v
                Upstream Resolver
```

This can simplify DNS filtering for systems that do not require domain-integrated service discovery.

The actual resolver configuration should be selected according to device requirements and security-zone design.

---

## 8. pfSense Integration

pfSense provides the network enforcement layer for the lab.

Depending on the architecture, pfSense can also participate in DNS forwarding, DHCP configuration, and DNS resolver distribution.

Potential integration points include:

- DHCP-provided DNS server addresses
- Firewall rules controlling DNS access
- Network segmentation
- DNS forwarding
- DNS resolver configuration
- Restrictions on unauthorized DNS traffic

### Conceptual Architecture

```text
                     pfSense
                        |
             +----------+----------+
             |                     |
             v                     v
         Lab VLAN              Test VLAN
             |                     |
             v                     v
        DNS Clients           DNS Clients
             |                     |
             +----------+----------+
                        |
                        v
                  DNS Services
```

The DNS design must account for routing and firewall rules between clients and their configured resolvers.

---

## 9. DNS Security Considerations

DNS is a foundational infrastructure service and should be treated as security-sensitive.

### Key Considerations

**Resolver Access**

Only intended clients should be able to use internal DNS services.

**Network Segmentation**

DNS requirements should not create unnecessary access between trusted and isolated networks.

**Administrative Access**

Pi-hole and DNS management interfaces should be restricted to authorized administrators.

**DNS Bypass**

Clients may attempt to use external resolvers or encrypted DNS mechanisms.

**Availability**

DNS outages can affect application connectivity, authentication, and general network functionality.

**Logging**

DNS telemetry can support troubleshooting and security analysis.

---

## 10. DNS Filtering Limitations

Pi-hole provides domain-based filtering rather than complete network security enforcement.

Its limitations include:

- Clients using unauthorized external resolvers
- DNS over HTTPS or DNS over TLS bypass scenarios
- Direct IP-based communication
- Malicious activity on otherwise legitimate domains
- Applications that do not use the expected DNS resolver
- Limited visibility into encrypted application traffic

These limitations reinforce the importance of combining DNS filtering with firewall policy, endpoint security, and network monitoring.

---

## 11. DNS Validation

DNS configuration should be tested from the client perspective.

### Windows

Identify the configured DNS servers:

```powershell
Get-DnsClientServerAddress
```

Resolve a public domain:

```powershell
Resolve-DnsName example.com
```

Query a specific DNS resolver:

```powershell
Resolve-DnsName example.com -Server 192.0.2.53
```

The address above is a documentation-only placeholder and must be replaced with the appropriate lab DNS server.

### Linux

Review resolver configuration:

```bash
resolvectl status
```

Query a resolver:

```bash
dig example.com
```

Query a specific resolver:

```bash
dig @192.0.2.53 example.com
```

These tests help determine which resolver is being used and whether expected DNS responses are returned.

---

## 12. Active Directory DNS Validation

Domain-joined systems require additional DNS validation.

### Domain Controller Discovery

```cmd
nltest /dsgetdc:example.local
```

### DNS SRV Record Lookup

```powershell
Resolve-DnsName `
  -Name "_ldap._tcp.dc._msdcs.example.local" `
  -Type SRV
```

### DNS Configuration

```powershell
Get-DnsClientServerAddress
```

The domain name is illustrative and should be replaced with the actual lab domain when performing tests.

Successful internet DNS resolution alone does not prove that Active Directory DNS is functioning correctly.

---

## 13. DNS Troubleshooting Methodology

When DNS resolution fails, I troubleshoot the request path in layers.

```text
Client DNS Configuration
          |
          v
Network Connectivity
          |
          v
Configured DNS Resolver
          |
          v
DNS Filtering / Local Zone
          |
          v
Upstream Resolver
          |
          v
DNS Response
```

### Common Causes

- Incorrect DNS server assignment
- Resolver service unavailable
- Firewall blocking DNS traffic
- Incorrect forwarding configuration
- DNS forwarding loops
- Missing internal DNS records
- Incorrect conditional forwarding
- DNS blocklist interference
- Client-side caching
- Upstream DNS failure

The goal is to identify where resolution fails before changing multiple DNS settings.

---

## 14. Troubleshooting — Internet Works but Domain Join Fails

A Windows endpoint may have working internet connectivity while failing to join an Active Directory domain.

This can occur when the client uses a public DNS resolver instead of the DNS service responsible for the AD domain.

### Validation Workflow

```text
Windows Client
      |
      v
Check DNS Servers
      |
      v
Resolve AD Domain
      |
      v
Query AD SRV Records
      |
      v
Locate Domain Controller
      |
      v
Validate Connectivity
      |
      v
Retry Domain Join
```

The issue should be evaluated as a service-discovery problem before assuming Active Directory itself is unavailable.

---

## 15. Troubleshooting — Pi-hole Blocks a Required Domain

DNS filtering can occasionally interfere with legitimate applications.

When this occurs, I would validate:

1. The affected client.
2. The requested domain.
3. Pi-hole query logs.
4. The blocklist or rule responsible.
5. Whether the domain is required.
6. Whether a narrowly scoped exception is appropriate.
7. Whether the application functions after the change.

Exceptions should be deliberate rather than broadly disabling DNS filtering.

---

## 16. Operational Validation

Network services should be evaluated for both configuration correctness and operational reliability.

| Test | Expected result |
|---|---|
| Pi-hole service | DNS resolver available |
| DNS filtering | Configured blocked domains are filtered |
| Allowed DNS queries | Resolve successfully |
| Upstream resolution | External names resolve |
| AD DNS | Internal domain records resolve |
| Domain controller discovery | Required SRV records resolve |
| pfSense policy | Intended DNS communication permitted |
| Restricted network | Unauthorized access denied |
| Windows client | Uses intended DNS resolver |
| Linux client | Uses intended DNS resolver |

Actual test results should be recorded separately from expected outcomes.

---

## 17. Security Monitoring

DNS activity can provide valuable information about endpoint and network behavior.

Potential security-monitoring use cases include:

- Unusual DNS query patterns
- Repeated requests to blocked domains
- Unexpected external resolvers
- DNS resolution failures
- Suspicious domain activity
- Changes in resolver behavior

Pi-hole query logs provide one source of visibility.

Wazuh and Security Onion provide additional endpoint and network perspectives, depending on their configured data sources and monitored traffic.

These platforms are complementary but are not assumed to be directly integrated.

---

## 18. Architecture Decisions and Tradeoffs

| Design decision | Consideration |
|---|---|
| Pi-hole DNS filtering | Centralized domain-based filtering |
| Active Directory DNS | Reliable domain service discovery |
| pfSense | Network enforcement and segmentation |
| DNS forwarding | Allows multiple DNS functions to coexist |
| Client DNS configuration | Determines which resolver receives queries |
| Firewall restrictions | Can reduce unauthorized resolver access |
| DNS logging | Supports troubleshooting and visibility |
| Resolver availability | Important dependency for client connectivity |

The architecture should balance DNS filtering, identity requirements, security, and operational reliability.

---

## 19. Lessons Learned

Network services are foundational dependencies for the broader infrastructure.

A DNS issue may initially appear to be an application, authentication, network, or device-management problem.

The complete architecture includes:

```text
DNS Client Configuration
          +
Resolver Availability
          +
Internal Name Resolution
          +
DNS Forwarding
          +
Network Connectivity
          +
Security Filtering
          +
Operational Validation
          =
Reliable DNS Architecture
```

Separating the responsibilities of Pi-hole, Active Directory DNS, and pfSense provides a clearer approach to designing and troubleshooting network services.

The environment also allows DNS security controls to be evaluated alongside endpoint monitoring, network segmentation, and hybrid identity.

---

## 20. Future Improvements

Potential areas for continued development include:

- Pi-hole blocklist management
- DNS filtering validation
- Active Directory DNS forwarding integration
- Conditional DNS forwarding
- DNS redundancy
- DNS logging and monitoring
- DNS over HTTPS bypass testing
- DNS over TLS policy evaluation
- DNS-based security detection
- Resolver performance testing
- DHCP and DNS integration
- Additional segmentation testing
- Automated DNS validation

These capabilities will be documented as they are implemented and tested.

---

## Planned Documentation

- [Pi-hole DNS Filtering](pihole.md)
- DNS integration with pfSense
- Active Directory DNS architecture
- DNS security and troubleshooting
