# Storage & Compute

Drive/shelf replacement verification, and GPU/compute host component detection
issues.

---

## 1. Storage — drive/shelf replacement verification

**Symptom:** A drive/carrier fault reported on a multi-drawer storage shelf.

**Risk:** It's easy to misidentify the failed drive bay by eye — the fault LED can
be on a different carrier than where you're counting from, especially under time
pressure.

**Check:**
1. Confirm the asset serial number matches the work order before pulling anything.
2. Cross-reference the fault LED directly against the drawer/bay diagram rather
   than counting visually.
3. On NetApp, confirm the exact failed disk before pulling:
   ```
   disk show -state failed
   disk show -owner <controller> -shelf <shelf_id>
   ```

**Fix:** Replace the confirmed drive. After replacement, a "bypassed" or similar
alert at the same bay usually indicates a seating issue with the new drive, not a
new unrelated failure — reseat before escalating.

---

## 2. GPU/compute host — detected component count below expected

**Symptom:** Management console reports fewer GPUs (or other hot-swap components)
detected than expected.

**Root cause — two distinct failure modes can produce the identical symptom:**
1. Physically present but improperly seated — fixable with a reseat.
2. A separate component with a thermal fault throwing an unrelated-looking flag at
   the same time.

**Check:**
```
nvidia-smi --query-gpu=index,name,temperature.gpu,power.draw --format=csv
nvidia-smi -q -d TEMPERATURE
```
Compare the returned GPU count against the expected count, and check per-GPU
temperature/power for the outlier before assuming it's purely a seating issue.

**Process for a live/production host:**
1. Get explicit approval before touching anything, especially on SLA-bound
   accounts.
2. Use a graceful shutdown, not a force power-off, with live workloads.
3. Inspect each flagged component independently — confirm physical presence first,
   then detection.
4. Get independent confirmation the count is back to expected before closing out.
