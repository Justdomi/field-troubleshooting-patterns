# Field Troubleshooting Patterns

A running reference of diagnostic patterns from hands-on field work and homelab
troubleshooting — networking, storage, compute/GPU, and hardware repair. Organized
by category so a specific fault type is easy to find rather than scrolling one long
file.

Every entry follows the same shape: **Symptom → Root cause → Check (how to confirm
it) → Fix.** Client names, work order numbers, and internal case IDs are
intentionally left out — this is the technical pattern only.

## Categories

- [`networking/`](./networking) — Cisco IOS: Layer 2/3 boundary issues, WAN/DHCP
  faults, ACL blocking, DNS resolution, provisioning templates, and a general
  diagnostic sequence.
- [`storage-compute/`](./storage-compute) — drive/shelf replacement verification,
  GPU/compute host component detection issues.
- [`hardware-repair/`](./hardware-repair) — printer mechanical/print-quality faults,
  out-of-band management interface failures, PSU alarms, post-repair diagnostic
  tool connectivity.
- [`methodology/`](./methodology) — the general troubleshooting methodology that
  applies across every vendor and platform.

*A running set of field-tested patterns — updated as new ones come up.*
