# DNS Configuration

## Purpose

DNS runs on the Windows Server 2025 domain controller at `10.0.0.5`. It provides authoritative resolution for the Active Directory domain and forwards unresolved external queries to `8.8.8.8`.

## Resolution design

```text
Lab client -> project-x-dc (10.0.0.5) -> internal AD DNS zone
                                         `-> 8.8.8.8 for unresolved external names
```

The clients' DNS server is `10.0.0.5`. The public resolver `8.8.8.8` is configured as a forwarder on the DNS server, not as the preferred DNS server on domain clients.

## Configuration summary

| Setting | Value |
|---|---|
| DNS server | `project-x-dc` |
| DNS server address | `10.0.0.5` |
| AD-integrated zone | `corp.project-x-dc.com` |
| External forwarder | `8.8.8.8` |
| Client DNS distributed through DHCP | `10.0.0.5` |

## Important naming distinction

- Domain name: `corp.project-x-dc.com`
- DC hostname: `project-x-dc`
- Expected DC FQDN: `project-x-dc.corp.project-x-dc.com`

Verify the exact DC FQDN with:

```powershell
hostname
$env:USERDNSDOMAIN
[System.Net.Dns]::GetHostEntry($env:COMPUTERNAME).HostName
```

## Forwarder configuration record

1. Open **DNS Manager** on `project-x-dc`.
2. Open the server properties.
3. Select **Forwarders**.
4. Add `8.8.8.8`.
5. Apply the change and verify that the forwarder resolves external names.

The DC's network adapter should use the internal DNS service - normally its own address - not `8.8.8.8` as the primary client resolver.

## Recommended records

| Name | Type | Address |
|---|---|---:|
| `project-x-dc` | `A` | `10.0.0.5` |
| `project-x-corp-srv` | `A` | `10.0.0.8` |
| `project-x-sec-box` | `A` | `10.0.0.10` |

Dynamic updates may create records for domain clients. If DHCP addresses are not reserved, stale records should be monitored and scavenging configured carefully.

## Validation

From the Windows client:

```powershell
ipconfig /all
Resolve-DnsName corp.project-x-dc.com
Resolve-DnsName project-x-dc.corp.project-x-dc.com
Resolve-DnsName project-x-sec-box.corp.project-x-dc.com
Resolve-DnsName example.com
```

From Ubuntu:

```bash
resolvectl status
dig @10.0.0.5 corp.project-x-dc.com
dig @10.0.0.5 example.com
```

Expected behavior:

- Internal names are answered by the DC.
- External names are requested through the DC and forwarded upstream.
- Domain clients show `10.0.0.5` as their DNS server.

## Troubleshooting

### External names fail but internal names work

- Confirm the `8.8.8.8` forwarder is present and reachable from the DC.
- Verify the DC has a valid default gateway of `10.0.0.1`.
- Test resolution directly from the DC.
- Review the DNS Server event log.

### Domain discovery fails

- Confirm the client is using only `10.0.0.5` for DNS.
- Confirm AD SRV records exist.
- Flush the client cache and re-register DNS:

```powershell
ipconfig /flushdns
ipconfig /registerdns
```

- Test domain-controller discovery:

```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.corp.project-x-dc.com
```

## References

- [Microsoft: DNS forwarding](https://learn.microsoft.com/en-us/windows-server/networking/dns/forwarding)
- [Microsoft: DNS client best practices](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/best-practices-for-dns-client-settings)