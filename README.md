# Project X Enterprise Cybersecurity Homelab

Project X is an isolated enterprise-style cybersecurity homelab built to demonstrate infrastructure administration, centralized identity, endpoint monitoring, detection engineering, attack simulation, and incident investigation.

The environment combines a Windows Active Directory domain, Windows and Linux endpoints, a Wazuh security monitoring platform, an internal test mail service, an Apache test server, and a Kali Linux security workstation. All systems operate inside the private `10.0.0.0/24` lab network.

> **Authorization notice:** All testing documented in this repository is performed only against systems owned by the lab operator in an isolated environment. Nothing in this repository authorizes testing against third-party systems.

## Project objectives

- Build and administer a small enterprise network.
- Centralize authentication and name resolution with Active Directory and DNS.
- Provide controlled address assignment with Windows DHCP.
- Collect and analyze endpoint and application telemetry with Wazuh.
- Reproduce security events safely from a dedicated Kali Linux workstation.
- Write custom detections and validate them with repeatable tests.
- Preserve sanitized evidence and document findings professionally.

## Environment summary

| System | Operating system | Address | Primary function | Wazuh role |
|---|---|---:|---|---|
| `project-x-dc` | Windows Server 2025 | `10.0.0.5` | AD DS, DNS and DHCP | Agent |
| `project-x-corp-svr` | Ubuntu Server 22.04 | `10.0.0.8` | MailHog SMTP test service | Not currently enrolled |
| `project-x-sec-box` | Ubuntu Server 22.04 | `10.0.0.10` | Wazuh manager, indexer and dashboard | Manager |
| `project-x-win-client` | Windows 11 Enterprise | DHCP; normally `10.0.0.100` | Domain workstation | Agent |
| `project-x-linux-client` | Ubuntu 22.04 | DHCP; normally `10.0.0.101` | Linux endpoint and Apache test server | Agent |
| `project-x-attacker` | Kali Linux | DHCP; normally `10.0.0.50` | Authorized attack and validation workstation | Not currently enrolled |

Common network settings:

- Subnet: `10.0.0.0/24`
- Default gateway: `10.0.0.1`
- Internal DNS server: `10.0.0.5`
- DNS forwarder: `8.8.8.8`
- Active Directory domain: `corp.project-x-dc.com`

Addresses marked as DHCP assignments may change unless DHCP reservations are configured. Every attack record should therefore state the addresses observed during that specific test.

## Architecture

The Windows Server domain controller provides AD DS, authoritative DNS for the internal domain, and DHCP. Domain members query the DC at `10.0.0.5` for DNS; unresolved external requests are forwarded by the DNS server to `8.8.8.8`.

The Wazuh server at `10.0.0.10` receives telemetry from three agents:

- Windows Server 2025 domain controller
- Windows 11 Enterprise client
- Ubuntu 22.04 Linux client

The Linux client also hosts a default Apache site used only for controlled log-generation and detection tests. The MailHog server provides an isolated SMTP capture service on TCP `1025` and a web interface on TCP `8025`.

See [Architecture](architecture/README.md) and the [Network Diagram](architecture/network-diagram.md).

## Repository structure

```text
.
├── README.md
├── architecture/
│   ├── README.md
│   └── network-diagram.md
├── setup/
│   ├── active-directory.md
│   ├── apache.md
│   ├── dhcp.md
│   ├── dns.md
│   ├── mailhog.md
│   └── wazuh.md
├── attack/
│   └── README.md
├── detection/
│   └── README.md
└── evidence/
    └── README.md
```

Each security case should use the same slug under the three case-study areas. For example:

```text
attack/http-get-flood/
detection/http-get-flood/
evidence/http-get-flood/
```

This keeps execution, defensive engineering, and supporting artifacts separate while making the complete investigation easy to follow.

## Setup documentation

- [Active Directory](setup/active-directory.md)
- [DHCP](setup/dhcp.md)
- [DNS](setup/dns.md)
- [Wazuh](setup/wazuh.md)
- [Apache test server](setup/apache.md)
- [MailHog test server](setup/mailhog.md)

## Case-study workflow

1. Define the purpose, authorization boundaries, prerequisites, and execution steps under `attack/<case-name>/`.
2. Explain telemetry sources, detection logic, rule configuration, validation, and tuning under `detection/<case-name>/`.
3. Store sanitized logs, screenshots, alert exports, findings, and limitations under `evidence/<case-name>/`.
4. Link all three documents to one another and from this README.

## Security and privacy rules

- Never commit passwords, API keys, tokens, private keys, recovery codes, or browser session data.
- Never publish screenshots that display credentials.
- Use placeholders such as `<REDACTED>` in commands and evidence.
- Sanitize personally identifiable information and unrelated host data.
- Keep raw packet captures and large unfiltered logs outside the public repository.
- Record tool versions and test dates when they affect reproducibility.
- Label single-source traffic as DoS, not DDoS.

## Current status

- [x] Private `/24` network established
- [x] Windows Server 2025 domain controller deployed
- [x] Active Directory domain created
- [x] DNS and DHCP services configured
- [x] Windows 11 Enterprise client connected to the domain
- [x] Ubuntu client deployed
- [x] Wazuh manager, indexer and dashboard deployed
- [x] Wazuh agents installed on the DC, Windows client and Linux client
- [x] MailHog SMTP test server deployed
- [x] Apache test server deployed
- [x] HTTP GET flood detection validated
- [ ] Add additional attack-and-detection case studies
- [ ] Add recovery and hardening validation

## References

- [Wazuh architecture](https://documentation.wazuh.com/current/getting-started/architecture.html)
- [Wazuh log data collection](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/how-it-works.html)
- [Microsoft DNS client best practices](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/best-practices-for-dns-client-settings)
- [MailHog project documentation](https://github.com/mailhog/MailHog)

