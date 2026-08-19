# Evidence

This directory contains sanitized artifacts supporting the results of Project X attack-and-detection case studies.

Create one folder per case:

```text
evidence/<case-name>/
|-- README.md
|-- alert-sanitized.json
|-- logs/
`-- screenshots/
```

Each evidence README should include:

- Test date and time zone
- Observed source and target addresses
- Detection rule and severity
- Result summary
- Key findings
- Root cause of any initial failure
- Limitations
- Recommended improvements
- Links to the matching attack and detection documentation

## Sanitization requirements

Before committing evidence:

- Remove passwords, tokens, private keys and cookies.
- Remove unrelated usernames and personal information.
- Remove public IP addresses unless specifically required and safe to publish.
- Avoid publishing VM MAC addresses and unique enrollment keys.
- Crop screenshots to the relevant evidence.
- Preserve timestamps, rule IDs, source/target roles and log fields needed to support the finding.
- Prefer small representative log samples over complete raw logs.

Raw, unsanitized artifacts should remain outside the public repository.