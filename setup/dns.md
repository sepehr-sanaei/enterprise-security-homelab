# DNS Configuration

## Overview

The Project X Domain Controller also provides DNS services for the Active Directory environment.

```text
DNS Server:  Windows Server 2025 Domain Controller
IP Address:  10.0.0.5
AD Domain:   corp.project-x-dc.com
```

DNS is a critical dependency of Active Directory. Domain-connected systems use DNS to locate Domain Controllers and other Active Directory services.

## Internal DNS

The Windows Server DNS role was installed alongside the Active Directory environment.

Domain systems use the Domain Controller's DNS service to resolve resources associated with:

```text
corp.project-x-dc.com
```

The intended DNS flow for domain clients is:

```text
Windows Client
  10.0.0.100
       │
       │ DNS Query
       ▼
Domain Controller
   10.0.0.5
       │
       ├── Internal AD resource
       │        │
       │        └── Resolve internally
       │
       └── External resource
                │
                ▼
          DNS Forwarder
             8.8.8.8
                │
                ▼
             Internet
```

## External DNS Forwarding

The Windows DNS Server was configured with the external forwarder:

```text
8.8.8.8
```

External DNS queries that cannot be resolved by the internal DNS infrastructure can therefore be forwarded for external resolution.

Domain clients do not need to query the external resolver directly. Instead, they can use the Domain Controller as their DNS server while the Domain Controller handles forwarding when necessary.

## Why This Matters

Using the Domain Controller as the DNS server allows Active Directory clients to locate domain services while maintaining external name resolution.

Configuring a domain workstation to use only an external resolver such as `8.8.8.8` could interfere with Active Directory service discovery because the public resolver does not contain the lab's internal AD DNS records.

## Validation

The Domain Controller's network and DNS configuration can be inspected with:

```powershell
ipconfig /all
```

Internal domain resolution can be tested using:

```powershell
nslookup corp.project-x-dc.com
```

External DNS resolution can be tested using:

```powershell
nslookup google.com
```

The Windows client can also be checked with:

```powershell
ipconfig /all
```

to verify which DNS server it is using.

## Current Status

* [x] Installed Windows DNS Server role
* [x] Integrated DNS with Active Directory environment
* [x] Configured external DNS forwarding
* [x] Added `8.8.8.8` as external forwarder
* [ ] Document DNS records
* [ ] Validate AD service discovery
* [ ] Capture validation evidence
* [ ] Monitor DNS activity through security infrastructure

## Security Relevance

DNS provides valuable security telemetry because both legitimate applications and malicious activity frequently depend on name resolution.

Later stages of Project X can use DNS telemetry to investigate activity such as unusual domain lookups, compromised endpoint behavior, reconnaissance, and communications with suspicious infrastructure.
