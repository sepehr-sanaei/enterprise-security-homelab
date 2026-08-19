# DHCP Configuration

## Purpose

The DHCP service runs on `project-x-dc` and supplies network settings to lab workstations. Infrastructure servers use static addresses so that Active Directory, DNS, Wazuh, and MailHog remain reachable at predictable locations.

## Scope design

| Setting | Value |
|---|---|
| Scope network | `10.0.0.0/24` |
| Subnet mask | `255.255.255.0` |
| Dynamic lease pool | `10.0.0.100-10.0.0.200` |
| Default gateway | `10.0.0.1` |
| DNS server distributed to clients | `10.0.0.5` |
| DNS domain | `corp.project-x-dc.com` |

`10.0.0.0` is the network address and `10.0.0.255` is the broadcast address. Neither can be leased to a host. The `/24` value describes the subnet; it does not mean that every address from `.0` through `.255` is a valid DHCP lease.

## Static infrastructure addresses

| Address | System | Reason for static assignment |
|---:|---|---|
| `10.0.0.1` | NAT gateway | Default route |
| `10.0.0.5` | Domain controller | AD DS, DNS and DHCP |
| `10.0.0.8` | MailHog server | Stable SMTP and UI endpoint |
| `10.0.0.10` | Wazuh server | Stable manager and dashboard endpoint |

These addresses fall outside the dynamic pool, preventing lease conflicts.

## Expected client assignments

| Client | Expected address | Assignment |
|---|---:|---|
| `project-x-win-client` | `10.0.0.100` | DHCP |
| `project-x-linux-client` | `10.0.0.101` | DHCP |
| `project-x-sec-work` | `10.0.0.103` | DHCP |

The values above are expected rather than guaranteed. Configure DHCP reservations using each VM's virtual NIC MAC address if these addresses must remain stable.

## Windows Server configuration record

1. Install the DHCP Server role on `project-x-dc`.
2. Complete post-installation authorization in Active Directory.
3. Create an IPv4 scope for `10.0.0.0/24`.
4. Set the address pool to `10.0.0.100-10.0.0.200`.
5. Configure scope option `003 Router` as `10.0.0.1`.
6. Configure scope option `006 DNS Servers` as `10.0.0.5`.
7. Configure scope option `015 DNS Domain Name` as `corp.project-x-dc.com`.
8. Activate the scope.
9. Add reservations for endpoints that need repeatable addresses.

## Validation

On a Windows DHCP client:

```powershell
ipconfig /release
ipconfig /renew
ipconfig /all
```

Expected results:

- IPv4 address inside `10.0.0.100-10.0.0.200`
- Subnet mask `255.255.255.0`
- Default gateway `10.0.0.1`
- DNS server `10.0.0.5`
- DNS suffix `corp.project-x-dc.com`

On an Ubuntu client using NetworkManager:

```bash
ip -4 address
ip route
resolvectl status
```

On the DHCP server, use the DHCP console to verify the active lease and its MAC address.

## Troubleshooting

### Client receives no lease

- Confirm the VM is connected to the correct NAT network.
- Confirm the DHCP Server service is running.
- Confirm the scope is active and has available addresses.
- Check whether another DHCP service is responding on the same virtual network.
- Verify Windows Firewall permits the DHCP Server role rules.

### Client receives an address but cannot join the domain

- Verify that DHCP provides `10.0.0.5` as DNS.
- Do not distribute `8.8.8.8` directly to domain clients.
- Confirm forward and reverse DNS resolution.
- Confirm the client clock is synchronized closely enough for Kerberos.

## Evidence to retain

- Sanitized screenshot of the scope and options
- Sanitized lease table
- `ipconfig /all` or `resolvectl status` output with irrelevant identifiers removed
- Date of validation

Do not publish client MAC addresses unless they are necessary and intentionally sanitized.
