# Detection Engineering

This directory contains defensive logic and validation notes for security events reproduced in the homelab.

Create one folder per case:

```text
detection/<case-name>/
|-- README.md
`-- rules/
    `-- <rule-file>
```

Each case should document:

- Detection goal
- Required telemetry and collection path
- Baseline or expected normal behavior
- Decoder and rule logic
- Threshold rationale
- Severity and ATT&CK mapping
- Test and validation procedure
- Dashboard queries
- False-positive considerations
- Tuning and rollback instructions
- Link to sanitized evidence

Rules committed here are version-controlled copies. Deployment locations on lab systems should be stated explicitly in the corresponding README.