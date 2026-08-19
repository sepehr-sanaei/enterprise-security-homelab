# Wazuh Deployment

## Purpose

Wazuh provides centralized security monitoring, log analysis, detection, alerting, and investigation for the Project X homelab.

## Server

| Setting | Value |
|---|---|
| Hostname | `project-x-sec-box` |
| Address | `10.0.0.10` |
| Operating system | Ubuntu Server 22.04 |
| Deployment | Single-node Wazuh server, indexer and dashboard |

## Enrolled endpoints

| Endpoint | Address | Operating system | Notable telemetry |
|---|---:|---|---|
| `project-x-dc` | `10.0.0.5` | Windows Server 2025 | Authentication, account and server events |
| `project-x-win-client` | DHCP; normally `10.0.0.100` | Windows 11 Enterprise | Endpoint, Defender and user activity |
| `project-x-linux-client` | DHCP; normally `10.0.0.101` | Ubuntu 22.04 | Linux system and Apache access logs |

## Network requirements

| Port | Protocol | Direction | Purpose |
|---:|---|---|---|
| `1514` | TCP | Agent -> manager | Agent event communication |
| `1515` | TCP | Agent -> manager | Automatic enrollment when enabled |
| `443` | TCP | Analyst -> dashboard | Dashboard access |

Additional component ports should be exposed only when required and should not be assumed to be safe for untrusted networks.

## Agent validation

On the Wazuh manager:

```bash
sudo /var/ossec/bin/agent_control -lc
sudo /var/ossec/bin/wazuh-control status
sudo ss -lntp | grep ':1514'
```

On a Linux agent:

```bash
sudo systemctl status wazuh-agent --no-pager
sudo cat /var/ossec/var/run/wazuh-agentd.state
nc -vz 10.0.0.10 1514
```

Healthy Linux agent state should include:

```text
status='connected'
msg_buffer='0'
```

On a Windows agent, verify the service and connectivity from an elevated PowerShell session:

```powershell
Get-Service WazuhSvc
Test-NetConnection 10.0.0.10 -Port 1514
```

## Apache log collection

The Linux agent monitors the Apache access log:

```xml
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/access.log</location>
</localfile>
```

Place the block inside an `<ossec_config>` element in the agent configuration, or deploy an equivalent centralized agent-group configuration. Avoid defining the same log twice.

Confirm collection on the Linux endpoint:

```bash
sudo grep -Ei "access\.log|logcollector|error|warning" \
  /var/ossec/logs/ossec.log | tail -n 100
```

Expected text includes:

```text
Analyzing file: '/var/log/apache2/access.log'
```

## End-to-end control test

From the Kali workstation, request a nonexistent Apache resource:

```bash
curl -i http://10.0.0.101/wazuh-test-page-does-not-exist
```

The expected `404` event should be written to Apache's access log and trigger Wazuh rule `31101`.

On the manager:

```bash
sudo tail -F /var/ossec/logs/alerts/alerts.json |
grep --line-buffered '"31101"'
```

In Discover, use the `wazuh-alerts-*` data view and search:

```text
rule.id:"31101"
```

This verifies the complete path:

```text
Apache -> Wazuh agent -> manager -> decoder/rule -> indexer -> dashboard
```

## HTTP flood detection

The lab includes a custom rate-correlation rule for successful HTTP GET requests. Its final test threshold is:

- 100 GET requests
- Same source IP
- Same log location
- Within 10 seconds
- Level 12 alert
- 60-second suppression period
- MITRE ATT&CK `T1499.003`

Store the rule under `detection/http-get-flood/` and the sanitized result under `evidence/http-get-flood/`. The corresponding attack documentation belongs under `attack/http-get-flood/`.

## Alerts versus archives

- `wazuh-alerts-*` contains events that triggered alerting rules.
- Successful routine web requests may be level `0` and therefore absent from the alert index.
- Raw event archiving can be enabled for troubleshooting, but it increases storage use.
- A professional detection should correlate routine events into one meaningful alert instead of generating hundreds of individual alerts.

## Troubleshooting sequence

1. Confirm the source application wrote the event locally.
2. Confirm Wazuh Logcollector is analyzing the file.
3. Confirm the agent state is connected and its buffer is not full.
4. Confirm TCP `1514` connectivity.
5. Confirm the event or alert exists in the manager JSON file.
6. Confirm Filebeat/indexer delivery.
7. Confirm the correct dashboard data view, time range and query.
8. Test custom rules with `wazuh-logtest` before increasing traffic volume.

## Security notes

- Use unique dashboard and API credentials.
- Restrict dashboard access to the lab network.
- Back up configuration before editing XML files.
- Do not publish enrollment keys, certificates or authentication secrets.
- Sanitize exported alerts before committing them.
- Keep rules in version control, but keep runtime databases and raw secrets out.

## References

- [Wazuh architecture and required ports](https://documentation.wazuh.com/current/getting-started/architecture.html)
- [Wazuh agent enrollment requirements](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/requirements.html)
- [Wazuh log collection behavior](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/how-it-works.html)
- [Wazuh local file monitoring](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html)
