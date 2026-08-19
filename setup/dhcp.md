# DHCP Configuration

## Overview

The Project X Domain Controller currently provides DHCP services for client systems on the corporate virtual network.

The DHCP Server role is installed on:

```text
Server:  Windows Server 2025 Domain Controller
Address: 10.0.0.5
Network: 10.0.0.0/24
```

DHCP allows endpoints to receive network configuration automatically rather than requiring each workstation to be configured manually.

## Network

The Project X virtual network uses:

```text
Network:         10.0.0.0/24
Default Gateway: 10.0.0.1
```

The Domain Controller uses the static address:

```text
10.0.0.5
```

Infrastructure services such as the Domain Controller require predictable addresses, so the DC is not dynamically addressed.

## DHCP Address Pool

The DHCP pool was configured as:

```text
Start Address: 10.0.0.100
End Address:   10.0.0.200
```

This reserves a portion of the subnet for dynamically addressed endpoints.

Current addressing can therefore be represented as:

```text
10.0.0.0/24
│
├── 10.0.0.1
│   Default Gateway
│
├── 10.0.0.5
│   Domain Controller
│   Static
│
└── 10.0.0.100 - 10.0.0.200
    DHCP Client Pool
```

## Windows Client

The first Windows 11 Enterprise workstation currently uses:

```text
IP Address: 10.0.0.100
Domain:     corp.project-x-dc.com
```

The client is connected to the same Project X virtual network as the Domain Controller.

## Why DHCP?

DHCP simplifies endpoint deployment by automatically distributing network configuration to clients.

As the environment expands, this becomes increasingly useful because new corporate endpoints can receive network settings without requiring manual IP configuration.

In an Active Directory environment, DHCP can also provide clients with important settings such as the default gateway and internal DNS server.

## Validation

Client network configuration can be inspected using:

```powershell
ipconfig /all
```

A DHCP-assigned configuration can be refreshed using:

```powershell
ipconfig /release
ipconfig /renew
```

Connectivity to the Domain Controller can then be tested:

```powershell
ping 10.0.0.5
```

DNS/domain functionality should also be validated after receiving the DHCP configuration.

## Current Status

* [x] Installed DHCP Server role
* [x] Configured DHCP address pool
* [x] Defined pool `10.0.0.100–10.0.0.200`
* [x] Connected Windows client to Project X network
* [ ] Document DHCP scope options
* [ ] Validate DHCP lease assignment
* [ ] Capture DHCP lease evidence
* [ ] Document additional clients as they are deployed

## Security Relevance

DHCP is part of the network's foundational infrastructure and determines how endpoints obtain their network configuration.

Understanding DHCP is important for both network administration and security because incorrect or malicious DHCP configuration can redirect client traffic, alter DNS configuration, or interfere with network connectivity.

DHCP activity can also assist defenders in correlating dynamically assigned IP addresses with specific endpoints during security investigations.
