# General Troubleshooting Methodology

The mental checklist that applies across every vendor and platform, regardless of
category.

1. **Identify the problem** — gather info, check logs, ask about recent changes
2. **Establish a theory** — start with the simplest explanation
3. **Test the theory** — confirm before moving forward
4. **Plan the fix** — consider side effects, get approvals if needed
5. **Implement** (or escalate with full documentation if beyond scope)
6. **Verify functionality** — confirm the fix worked and nothing else broke
7. **Document** — always last, never skipped

**Common mistake:** jumping straight to the fix without testing a theory first —
leads to fixing the wrong thing or missing a second, unrelated fault hiding behind
the obvious one.

This sequence is the backbone behind every entry in
[`networking/`](../networking), [`storage-compute/`](../storage-compute), and
[`hardware-repair/`](../hardware-repair) — each of those documents the
symptom-specific version of steps 1-4; this file is the version that applies no
matter what's actually broken.
