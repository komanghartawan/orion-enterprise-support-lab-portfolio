# ORION Enterprise Support Lab Portfolio

## Purpose
This repository is a public portfolio of hands-on, reproducible IT Support and Windows infrastructure labs. Each lab is documented with a runbook and supported by evidence artifacts (screenshots and raw command output) to make results verifiable.

## Labs
| Lab | Title | Focus | Key files |
|---:|---|---|---|
| 01 | [LAB-01 Core Identity](./labs/LAB-01-core-identity/README.md) | AD DS, DNS, DHCP, domain join, deterministic addressing | [runbook](./labs/LAB-01-core-identity/runbook.md) - [commands](./labs/LAB-01-core-identity/commands/commands.md) - [evidence](./labs/LAB-01-core-identity/evidence/) - [diagram](./labs/LAB-01-core-identity/diagrams/) |

## Repository standards

### Structure
- `labs/` contains all labs.
- Each lab lives under `labs/LAB-XX-<slug>/` and includes:
  - `README.md` (lab overview and evidence index)
  - `runbook.md` (step-by-step procedure)
  - `commands/commands.md` (evidence capture commands)
  - `evidence/` (authoritative proof artifacts, PNG and TXT)
  - `diagrams/` (architecture diagram(s))

### Evidence rules
- `evidence/` is the authoritative proof vault.
- TXT evidence is stored as UTF-8 and captured as raw command output (no commentary inside evidence files).
- Evidence filenames use lowercase and underscores only:
  - `lab-xx_nn_system_what.ext`
  - `nn` is two digits, appended sequentially.

### Networking posture
- Default posture is isolated lab networking:
  - no NAT
  - no Internet gateway
  - no default gateway on lab VMs unless a lab explicitly documents an exception

### Public repo hygiene
- Do not commit secrets or certificates.
- Do not publish product keys, serial numbers, or personal contact details in screenshots.
- Keep screenshots tightly cropped to the relevant UI/output.

## Quick navigation
- Start here: [LAB-01 Core Identity](./labs/LAB-01-core-identity/README.md)
