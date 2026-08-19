# Architecture

## Purpose

This document describes the logical design of the Project X homelab, the responsibilities of each virtual machine, and the main trust and telemetry relationships.

## Design summary

Project X uses a single private LAN, `10.0.0.0/24`, behind the virtualization platform's NAT gateway at `10.0.0.1`. The environment is intentionally small enough to run on one physical host while retaining separate infrastructure, endpoint, monitoring, mail, and attacker roles.

The architecture contains three functional planes:

| Plane | Components | Purpose |
|---|---|---|
| Identity and infrastructure | Domain controller, DNS and DHCP | Authentication, domain services, name resolution and address assignment |
| Monitored enterprise systems | Windows client and Ubuntu client | Generate endpoint and application telemetry |
| Security operations | Wazuh server and Kali workstation | Detect, investigate and safely reproduce security activity |

MailHog is an additional isolated application service used to capture test email without delivering it to the public internet.

## Host inventory

| Hostname | Addressing | Operating system | Services and responsibilities |
|---|---:|---|---|
| `project-x-dc` | Static `10.0.0.5` | Windows Server 2025 | AD DS, DNS, DHCP, Wazuh agent |
| `project-x-email-svr` | Static `10.0.0.8` | Ubuntu Server 22.04 | MailHog SMTP capture and web UI |
| `project-x-sec-box` | Static `10.0.0.10` | Ubuntu Server 22.04 | Wazuh manager, indexer and dashboard |
| `project-x-win-client` | DHCP, expected `10.0.0.100` | Windows 11 Enterprise | Domain member and Wazuh agent |
| `project-x-linux-client` | DHCP, expected `10.0.0.101` | Ubuntu 22.04 | Linux endpoint, Apache and Wazuh agent |
| `project-x-sec-work` | DHCP, expected `10.0.0.103` | Kali Linux | Authorized attack simulation and validation |

## Network parameters

| Setting | Value |
|---|---|
| Network | `10.0.0.0/24` |
| Subnet mask | `255.255.255.0` |
| Usable host addresses | `10.0.0.1-10.0.0.254` |
| Network address | `10.0.0.0` |
| Broadcast address | `10.0.0.255` |
| Default gateway | `10.0.0.1` |
| Internal DNS | `10.0.0.5` |
| External DNS forwarder | `8.8.8.8` |
| AD domain | `corp.project-x-dc.com` |

## Core communication paths

| Source | Destination | Port/protocol | Purpose |
|---|---|---|---|
| Domain members | `10.0.0.5` | DNS `53/TCP,UDP` | Internal and forwarded name resolution |
| DHCP clients | DHCP server | DHCP `67/UDP` and `68/UDP` | Address configuration |
| Wazuh agents | `10.0.0.10` | `1514/TCP` | Encrypted agent event transmission |
| Enrolling Wazuh agents | `10.0.0.10` | `1515/TCP` | Agent enrollment when enabled |
| Analyst browser | `10.0.0.10` | `443/TCP` | Wazuh dashboard |
| Test mail clients | `10.0.0.8` | `1025/TCP` | MailHog SMTP capture |
| Analyst browser | `10.0.0.8` | `8025/TCP` | MailHog web interface |
| Lab clients | `10.0.0.101` | `80/TCP` | Apache test application |

Only services required for a documented lab function should be reachable. Actual open ports should be verified with host firewall configuration and a controlled scan before publishing a case study.

## Trust boundaries

### Domain boundary

The `corp.project-x-dc.com` domain centralizes Windows identity and authentication. The domain controller is the highest-trust system and should not be used for general browsing, attack tooling, or unrelated services.

### Monitoring boundary

Wazuh agents send telemetry to `project-x-sec-box`. Agent communication is initiated toward the manager. Dashboard access should be restricted to the lab network and protected with unique credentials.

### Attack-simulation boundary

`project-x-sec-work` is untrusted from the perspective of monitored systems. It is used only for authorized tests against lab-owned targets. It should not hold production credentials or bridge the lab into unrelated networks.

### Email-testing boundary

MailHog captures test messages locally. It is not a production SMTP server and should not relay messages to public recipients.

## Addressing strategy

Infrastructure servers use static addresses below the DHCP pool:

- Gateway: `.1`
- Domain controller: `.5`
- Mail server: `.8`
- Wazuh server: `.10`

Clients use DHCP addresses from `.100-.200`. Reservations are recommended for systems whose address is referenced repeatedly in Wazuh rules, screenshots, or attack documentation.

## Telemetry flow

1. Windows and Linux endpoints generate operating-system or application events.
2. Wazuh agents collect configured sources and transmit them to `10.0.0.10:1514/TCP`.
3. The Wazuh manager decodes events and evaluates rules.
4. Alerts are indexed and presented through the Wazuh dashboard.
5. Sanitized evidence is exported into the matching repository case folder.

## Diagram

See [Network Diagram](network-diagram.md).

## Known limitations

- The environment is hosted on one physical computer, so it does not model physical network redundancy.
- The network currently uses a single `/24` rather than separate VLANs.
- DHCP client addresses can change unless reservations are configured.
- MailHog is a testing sink, not a hardened mail transfer agent.
- Apache serves only a default test site and does not represent a production application.
- Kali traffic from one host represents a single-source attack simulation, not a distributed attack.