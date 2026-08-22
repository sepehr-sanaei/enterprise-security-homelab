# Attack Simulations

This directory documents authorized security simulations performed exclusively inside the Project X homelab.

All case files live directly inside this folder (no per-case subfolders):

```text
attack/<case-name>.md
```

Each case file should include:

- Objective and hypothesis
- Authorization and scope
- Source and target systems
- Preconditions
- Tools and versions
- Exact commands or actions
- Expected telemetry
- Safety limits and stop conditions
- Cleanup steps
- Links to the matching detection and evidence entries

Example:

```text
attack/reconnaissance.md
```

Do not include credentials, weaponize instructions for third-party targets, or describe a single-source test as DDoS.