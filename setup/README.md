# Domain Controller — Windows Server 2025

## Overview

This component provides centralized identity, authentication, DNS, and DHCP services for the **Project X** enterprise homelab.

A Windows Server 2025 virtual machine was deployed and promoted to an Active Directory Domain Controller for the `corp.project-x-dc.com` domain.

The server currently provides three primary infrastructure services:

* Active Directory Domain Services (AD DS)
* DNS Server
* DHCP Server

The Domain Controller uses a static IP address so that endpoints and infrastructure services can reliably locate Active Directory and DNS services.

---

## Server Information

| Property         | Configuration           |
| ---------------- | ----------------------- |
| Operating System | Windows Server 2025     |
| Network          | Project X NAT Network   |
| IP Address       | `10.0.0.5`              |
| Subnet           | `10.0.0.0/24`           |
| Prefix Length    | `/24`                   |
| Default Gateway  | `10.0.0.1`              |
| AD Domain        | `corp.project-x-dc.com` |
| DNS Forwarder    | `8.8.8.8`               |
| Roles            | AD DS, DNS, DHCP        |

---

## Network Configuration

The Domain Controller is connected to the dedicated Project X NAT network in VirtualBox.

The virtual network uses:

```text
Network:  Project X NAT Network
Subnet:   10.0.0.0/24
Gateway:  10.0.0.1
```

The Windows Server was manually assigned:

```text
IP Address:      10.0.0.5
Prefix Length:   /24
Default Gateway: 10.0.0.1
```

A static IP address was selected because the Domain Controller provides critical infrastructure services that other domain systems need to locate consistently.

Changing the Domain Controller's address dynamically could prevent clients from locating DNS or Active Directory services.

---

## Server Roles

The required server roles were installed through **Server Manager → Add Roles and Features**.

### Active Directory Domain Services

**Active Directory Domain Services (AD DS)** was installed to provide centralized identity and authentication for the enterprise environment.

AD DS will allow Project X to centrally manage:

* Users
* Computers
* Groups
* Authentication
* Organizational Units
* Group Policy

Rather than managing independent local identities on every endpoint, domain-connected systems can authenticate against the centralized Active Directory environment.

---

## Domain Controller Promotion

After installing AD DS, the Windows Server was promoted to a Domain Controller.

A new Active Directory forest was created with the root domain:

```text
corp.project-x-dc.com
```

The resulting architecture currently follows:

```text
corp.project-x-dc.com
        │
        ▼
Windows Server 2025
    10.0.0.5
        │
        ├── Active Directory
        ├── DNS
        └── DHCP
```

The Domain Controller now provides the identity infrastructure required for enterprise endpoints to join and authenticate against the Project X domain.

---

# DNS

The **DNS Server** role is installed on the Domain Controller.

DNS is particularly important within an Active Directory environment because domain clients use DNS to discover Domain Controllers and Active Directory services.

The Domain Controller therefore acts as the internal DNS server for the Project X domain.

## DNS Forwarding

The DNS Server was configured with the following external DNS forwarder:

```text
8.8.8.8
```

This allows DNS requests that cannot be resolved internally to be forwarded for external resolution.

Conceptually:

```text
Windows Client
      │
      │ DNS request
      ▼
Domain Controller
   10.0.0.5
      │
      ├── corp.project-x-dc.com
      │        │
      │        └── Internal DNS resolution
      │
      └── External domain
               │
               ▼
             8.8.8.8
               │
               ▼
            Internet
```

This design allows domain clients to use the Domain Controller for DNS while still being able to resolve external Internet domains.

---

# DHCP

The **DHCP Server** role was installed to provide dynamic IP configuration to endpoints within the Project X corporate network.

## DHCP Address Pool

The configured DHCP pool is:

```text
Start Address: 10.0.0.100
End Address:   10.0.0.200
```

This provides a dedicated portion of the `10.0.0.0/24` network for dynamically addressed endpoints.

The Domain Controller itself is outside this pool and retains its static address:

```text
10.0.0.5
```

This separation ensures that the infrastructure server's address remains predictable while client devices can receive addresses dynamically.

### Current Addressing

| System                |                 Address | Assignment      |
| --------------------- | ----------------------: | --------------- |
| Default Gateway       |              `10.0.0.1` | Virtual network |
| Domain Controller     |              `10.0.0.5` | Static          |
| Windows 11 Enterprise |            `10.0.0.100` | Client address  |
| DHCP Clients          | `10.0.0.100–10.0.0.200` | Dynamic pool    |

As additional servers and infrastructure are introduced, static addresses can be kept outside the DHCP pool or appropriate DHCP reservations/exclusions can be configured.

---

# Windows Client Integration

A Windows 11 Enterprise workstation has been deployed as the first corporate endpoint.

Current configuration:

```text
OS:         Windows 11 Enterprise
IP Address: 10.0.0.100
Domain:     corp.project-x-dc.com
```

The workstation is joined to the `corp.project-x-dc.com` Active Directory domain.

This provides a foundation for later testing of:

* Domain authentication
* Active Directory users and groups
* Group Policy
* Endpoint administration
* Security logging
* SIEM telemetry
* Attack simulation
* Detection engineering

---

# Current Architecture

```text
                    Internet
                       │
                       │
              VirtualBox NAT
                       │
                10.0.0.1
                       │
                       ▼
              10.0.0.0/24
              Project X Network
                       │
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
  Windows Server 2025       Windows 11 Enterprise
      10.0.0.5                  10.0.0.100
          │                         │
          │                         │
      AD DS + DNS               Domain Joined
        + DHCP                       │
          │                          │
          └────────────┬─────────────┘
                       │
                       ▼
             corp.project-x-dc.com

DNS external forwarding:
10.0.0.5 → 8.8.8.8
```

---

# Implementation Status

### Completed

* [x] Deployed Windows Server 2025 VM
* [x] Connected server to Project X NAT network
* [x] Configured static IP `10.0.0.5/24`
* [x] Configured default gateway `10.0.0.1`
* [x] Installed Active Directory Domain Services
* [x] Installed DNS Server
* [x] Installed DHCP Server
* [x] Promoted server to Domain Controller
* [x] Created `corp.project-x-dc.com` domain
* [x] Configured DNS forwarding to `8.8.8.8`
* [x] Configured DHCP address pool
* [x] Deployed Windows 11 Enterprise endpoint
* [x] Integrated Windows endpoint with Active Directory

---

# Validation

The deployment should be validated at multiple layers rather than relying solely on successful installation.

## Network Connectivity

The Domain Controller's network configuration can be verified using:

```powershell
ipconfig /all
```

Expected primary address:

```text
IPv4 Address: 10.0.0.5
```

Connectivity to the virtual gateway can be tested with:

```powershell
ping 10.0.0.1
```

---

## DNS

DNS resolution can be tested using:

```powershell
nslookup corp.project-x-dc.com
```

External resolution can also be tested:

```powershell
nslookup google.com
```

Successful internal and external resolution demonstrates that the Domain Controller can provide internal DNS while forwarding external requests.

---

## Active Directory

Active Directory information can be inspected with PowerShell:

```powershell
Get-ADDomain
```

The Domain Controller can be queried with:

```powershell
Get-ADDomainController
```

The expected domain is:

```text
corp.project-x-dc.com
```

---

## Client Domain Membership

On the Windows 11 Enterprise client:

```powershell
whoami
```

and:

```powershell
systeminfo
```

can be used as part of verifying the system's identity and domain membership.

DNS configuration should also be checked using:

```powershell
ipconfig /all
```

The endpoint should use the internal Active Directory DNS infrastructure rather than bypassing it with a public DNS resolver.

---

# Security Relevance

The Domain Controller is one of the most security-critical systems in the environment.

Compromise of Active Directory can potentially provide an attacker with access to identities, systems, authentication mechanisms, and other enterprise resources.

This server therefore provides an important target for later security exercises involving:

* Authentication monitoring
* Account activity
* Privilege escalation
* Credential attacks
* Active Directory reconnaissance
* Lateral movement
* Group Policy
* Windows Event Logs
* SIEM detection
* Incident investigation

As the security monitoring infrastructure is deployed, telemetry from the Domain Controller will be collected and analyzed to observe how legitimate and malicious activity appears from a defender's perspective.

---

# Next Steps

The next stages of the Active Directory environment will include:

* [ ] Design Organizational Unit structure
* [ ] Create enterprise users
* [ ] Create security groups
* [ ] Configure Group Policy
* [ ] Validate DNS records and AD service discovery
* [ ] Validate DHCP leases and options
* [ ] Configure additional endpoints
* [ ] Enable enhanced endpoint telemetry
* [ ] Forward security events to the SIEM
* [ ] Establish a baseline of normal authentication activity
* [ ] Perform controlled security simulations
* [ ] Analyze resulting events and alerts

---

# Lessons Learned

This stage established the core infrastructure required for a centralized Windows enterprise environment.

Key concepts demonstrated include:

* The role of Active Directory in centralized identity management
* Why Domain Controllers require predictable addressing
* The relationship between Active Directory and DNS
* The distinction between internal DNS resolution and external DNS forwarding
* The role of DHCP in dynamically configuring enterprise endpoints
* Domain-based endpoint management
* The dependency relationships between networking, DNS, authentication, and Active Directory

Future stages of the project will build security monitoring and attack simulation capabilities on top of this infrastructure.
