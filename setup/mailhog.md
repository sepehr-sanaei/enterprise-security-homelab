# MailHog Test Server

## Purpose

MailHog provides an isolated SMTP capture service for phishing-awareness exercises, mail-client testing, and detection validation. Messages are retained inside the lab instead of being delivered to real recipients.

## System summary

| Setting | Value |
|---|---|
| Hostname | `project-x-corp-srv` |
| Address | `10.0.0.8` |
| Operating system | Ubuntu Server 22.04 |
| SMTP listener | `1025/TCP` |
| Web interface | `8025/TCP` |

## Access

Applications send test mail to:

```text
10.0.0.8:1025
```

Analysts review captured messages at:

```text
http://10.0.0.8:8025
```

## Validation

From a lab system:

```bash
nc -vz 10.0.0.8 1025
nc -vz 10.0.0.8 8025
```

On the MailHog host, verify the service or container and listening ports:

```bash
sudo ss -lntp | grep -E ':1025|:8025'
```

If deployed with Docker:

```bash
docker ps
docker logs --tail 50 mailhog
```

## Security notes

- Do not expose MailHog to the public internet.
- Do not use real customer, university, pharmacy, or personal addresses.
- Use fictional recipients and sanitized message content.
- Do not embed live credentials or real session tokens in captured messages.
- Treat MailHog as a test sink, not a production mail server.
- Default in-memory storage may be lost after restart unless persistence is configured.

## Reference

- [MailHog repository and default ports](https://github.com/mailhog/MailHog)