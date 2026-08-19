# Active Directory Domain Services

## Purpose

Active Directory Domain Services provides centralized Windows identity, authentication, authorization, and domain management for the Project X environment.

## Domain summary

| Setting | Value |
|---|---|
| Domain | `corp.project-x-dc.com` |
| Domain controller | `project-x-dc` |
| DC address | `10.0.0.5` |
| Server operating system | Windows Server 2025 |
| DNS | AD-integrated DNS on `10.0.0.5` |
| Domain Windows endpoint | `project-x-win-client` |

## Role placement

The domain controller currently provides:

- Active Directory Domain Services
- DNS Server
- DHCP Server
- Wazuh agent telemetry

The DC should not host general-purpose user applications, attack tools, or web browsing sessions. Its administrative surface should remain as small as practical.

## Identity model

The lab uses named identities including `janed`, `sec-box`, and the built-in `Administrator` identity. Before publishing screenshots or exports, verify whether each identity is a domain account or a local host account and label it accordingly.

Recommended practice:

- Reserve the built-in `Administrator` account for administrative recovery and controlled tasks.
- Use a separate standard user account for normal workstation activity.
- Use a separate named administrative account when performing privileged changes.
- Apply least privilege and avoid granting Domain Admin membership unnecessarily.
- Never store account passwords in repository files or screenshots.

## Suggested organizational units

The following OU layout makes the lab easier to manage and demonstrate:

```text
corp.project-x-dc.com
|-- ProjectX-Users
|   |-- Standard-Users
|   `-- Administrative-Users
|-- ProjectX-Computers
|   |-- Workstations
|   `-- Servers
`-- ProjectX-Service-Accounts
```

Move objects only after confirming that Group Policy inheritance will remain correct. Keep the domain controller in the built-in **Domain Controllers** OU.

## Windows 11 domain membership

`project-x-win-client` is the Windows 11 Enterprise domain workstation.

Before joining:

1. Confirm the client has an address in `10.0.0.0/24`.
2. Confirm the default gateway is `10.0.0.1`.
3. Confirm the DNS server is `10.0.0.5`.
4. Confirm the client can resolve the domain and DC.
5. Confirm the client clock is synchronized.

Validation commands:

```powershell
ipconfig /all
Resolve-DnsName corp.project-x-dc.com
nltest /dsgetdc:corp.project-x-dc.com
whoami
whoami /fqdn
```

Expected results:

- A domain controller is discovered at `10.0.0.5`.
- The client reports membership in `corp.project-x-dc.com`.
- Domain authentication succeeds using an authorized account.

## Baseline Group Policy opportunities

Future hardening work can document and validate:

- Password and account lockout policy
- Windows Defender configuration
- Advanced audit policy
- PowerShell logging
- Windows Event Forwarding or complementary Wazuh collection
- Firewall policy
- Local administrator restrictions
- Screen-lock policy
- Removable-storage policy

Each change should include its objective, linked GPO, scope, validation command, and rollback procedure.

## Wazuh monitoring

The domain controller has a Wazuh agent. Useful telemetry includes:

- Security log authentication events
- Account creation, deletion and group changes
- Privileged logons
- Group Policy changes
- PowerShell activity when logging is enabled
- Windows Defender events

Detection rules and alert evidence belong under `detection/<case-name>/` and `evidence/<case-name>/`, not in this setup file.

## Validation evidence

Capture only sanitized evidence:

- AD Users and Computers OU structure
- Domain membership screen
- `nltest /dsgetdc` output
- Successful standard-user sign-in
- Wazuh agent status for the DC

Do not publish passwords, password hints, recovery information, or sensitive security identifiers unless there is a specific sanitized reason.