# Attack Simulations

This directory documents authorized security simulations performed exclusively inside the Project X homelab.

Create one folder per case:

```text
attack/<case-name>/README.md
```

Each case should include:

- Objective and hypothesis
- Authorization and scope
- Source and target systems
- Preconditions
- Tools and versions
- Exact commands or actions
- Expected telemetry
- Safety limits and stop conditions
- Cleanup steps
- Links to the matching detection and evidence folders

Example:

```text
attack/http-get-flood/README.md
```

Do not include credentials, weaponize instructions for third-party targets, or describe a single-source test as DDoS.
