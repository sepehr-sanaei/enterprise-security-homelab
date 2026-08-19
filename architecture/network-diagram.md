# Network Diagram

The following diagram represents the logical Project X homelab topology. Addresses marked as DHCP assignments are expected values and may change unless reservations are configured.

```mermaid
flowchart TB
    EXT["External DNS<br/>8.8.8.8"]
    GW["NAT gateway<br/>10.0.0.1"]

    subgraph LAN["Project X LAN - 10.0.0.0/24"]
        direction TB

        subgraph INFRA["Identity and infrastructure"]
            DC["project-x-dc<br/>Windows Server 2025<br/>10.0.0.5<br/>AD DS - DNS - DHCP - Wazuh agent"]
            MAIL["project-x-corp-svr<br/>Ubuntu 22.04<br/>10.0.0.8<br/>MailHog 1025/8025"]
        end

        subgraph SOC["Security operations"]
            WAZUH["project-x-sec-box<br/>Ubuntu 22.04<br/>10.0.0.10<br/>Wazuh server"]
            KALI["project-x-attacker<br/>Kali Linux<br/>DHCP: ~10.0.0.103<br/>Authorized testing"]
        end

        subgraph ENDPOINTS["Monitored endpoints"]
            WIN["project-x-win-client<br/>Windows 11 Enterprise<br/>DHCP: ~10.0.0.100<br/>Domain member - Wazuh agent"]
            LINUX["project-x-linux-client<br/>Ubuntu 22.04<br/>DHCP: ~10.0.0.101<br/>Apache - Wazuh agent"]
        end
    end

    EXT -->|"DNS forwarding"| DC
    GW --- LAN
    DC -->|"DHCP and internal DNS"| WIN
    DC -->|"DHCP and internal DNS"| LINUX
    DC -->|"DHCP and internal DNS"| KALI
    DC -->|"Wazuh telemetry 1514/TCP"| WAZUH
    WIN -->|"Wazuh telemetry 1514/TCP"| WAZUH
    LINUX -->|"Wazuh telemetry 1514/TCP"| WAZUH
    KALI -.->|"Authorized simulations"| WIN
    KALI -.->|"Authorized simulations"| LINUX
    KALI -.->|"Test SMTP 1025/TCP"| MAIL
```

## Relationship notes

- `10.0.0.5` is the internal DNS server for lab clients.
- `8.8.8.8` is a DNS forwarder used by the DC for names it cannot resolve; domain clients should not bypass the DC and query it directly.
- The DC, Windows client, and Linux client send Wazuh telemetry to `10.0.0.10`.
- The Kali workstation is used only for authorized simulations against lab systems.
- MailHog captures test SMTP traffic on port `1025`; its browser interface is available on port `8025`.
- Apache on the Linux client is a telemetry-generation target, not a production application.

## Simplified data flows

| Flow | Path |
|---|---|
| Internal DNS | Client -> `10.0.0.5` |
| External DNS | Client -> DC -> `8.8.8.8` |
| Wazuh telemetry | Agent -> `10.0.0.10:1514/TCP` |
| Test email | Lab client -> `10.0.0.8:1025/TCP` |
| Mail review | Browser -> `10.0.0.8:8025/TCP` |
| Web test | Kali -> `10.0.0.101:80/TCP` |