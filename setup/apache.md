# Apache Test Server

## Purpose

Apache runs on `project-x-linux-client` solely as a controlled source of web-access telemetry. It hosts no production application and has no special application configuration.

## System summary

| Setting | Value |
|---|---|
| Host | `project-x-linux-client` |
| Operating system | Ubuntu 22.04 |
| Address | DHCP; normally `10.0.0.101` |
| Service | Apache HTTP Server |
| HTTP port | `80/TCP` |
| Site | Default test page |
| Wazuh agent | Installed and connected |

## Installation and service validation

```bash
sudo apt update
sudo apt install apache2
sudo systemctl enable --now apache2
sudo systemctl status apache2 --no-pager
sudo ss -lntp | grep ':80'
```

If UFW is enabled:

```bash
sudo ufw allow 'Apache'
sudo ufw status
```

## Connectivity test

From another lab system:

```bash
curl -I http://10.0.0.101/
```

Expected result:

```text
HTTP/1.1 200 OK
```

Because the address is DHCP-assigned, replace `10.0.0.101` with the current lease if it has changed.

## Relevant logs

| Log | Purpose |
|---|---|
| `/var/log/apache2/access.log` | Requests, source addresses, methods, paths, status codes and user agents |
| `/var/log/apache2/error.log` | Startup, module, permission and request-processing errors |

Monitor logs locally:

```bash
sudo tail -F /var/log/apache2/access.log
```

```bash
sudo tail -F /var/log/apache2/error.log
```

## Wazuh collection

The current Wazuh test uses:

```xml
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/access.log</location>
</localfile>
```

Confirm that the agent opened the file:

```bash
sudo grep -Ei "access\.log|logcollector|error|warning" \
  /var/ossec/logs/ossec.log | tail -n 100
```

Do not define the same file in both the local agent configuration and centralized group configuration. A duplicate definition can cause warnings and confusing test results.

## Safe validation events

### Successful request

```bash
curl -i http://10.0.0.101/
```

This normally returns `200 OK`. Routine successful events may not appear in the Wazuh alert index unless a custom rule correlates them.

### Guaranteed error-path control test

```bash
curl -i http://10.0.0.101/wazuh-test-page-does-not-exist
```

This returns `404 Not Found` and should trigger Wazuh web rule `31101` when collection and indexing are healthy.

### Controlled HTTP GET flood test

The authorized ApacheBench simulation and rate-based Wazuh detection are documented separately:

- `attack/http-get-flood/`
- `detection/http-get-flood/`
- `evidence/http-get-flood/`

The traffic comes from one Kali host, so it is an HTTP GET flood or DoS simulation - not a DDoS test.

## Limitations

- The default Apache page is not representative of a complex application.
- The service does not currently include TLS, authentication, a database or a reverse proxy.
- Request volume alone does not prove service impact.
- Availability findings should be supported with latency, error-rate, CPU and memory measurements.

## Hardening opportunities

- Restrict access to the lab network.
- Disable unnecessary modules.
- Add TLS for future authentication tests.
- Add rate limiting through a reverse proxy for defensive validation.
- Review server tokens and directory listing behavior.
- Maintain package updates and snapshots before major tests.

## Evidence to retain

- Service status
- Current listening port
- A sanitized access-log sample
- Wazuh collection confirmation
- Wazuh alert export
- Dashboard screenshot
- Test date and observed source/target addresses