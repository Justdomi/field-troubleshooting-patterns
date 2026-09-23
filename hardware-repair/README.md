# Hardware Repair

Printer mechanical/print-quality faults, out-of-band management interface
failures, PSU alarms, and post-repair diagnostic tool connectivity.

---

## 1. Printer — mechanical fault, part handling

**Symptom:** A mechanical fault (jam, feed issue) where the exact failed part
isn't obvious from the symptom alone.

**Check:** Inspect the actual feed path directly — rollers, separation pad,
pickup assembly — rather than assuming the part ordered ahead of time (based on
symptom description alone) is the one actually needed.

**Lesson:** the part that ends up actually needed isn't always obvious until
you're inspecting the feed path — a part ordered as a precaution isn't always the
one used. Document clearly when a part was ordered but not needed, rather than
leaving it as an ambiguous open item.

---

## 2. Printer — recurring toner/print-quality degradation

**Symptom:** Shadowy/faded print output; on a separate later visit to the same
unit, ink/toner smearing and not bonding to paper.

**Root cause:** Worn maintenance-kit components (drum/fuser/rollers) rather than a
single discrete part failure — this kind of gradual print-quality issue is often
kit-level wear, not one bad part.

**Check:** If this is the second/third visit for print-quality complaints on the
same unit, treat it as a maintenance-kit-wear question rather than re-diagnosing
from scratch — check total page count against the kit's rated page life.

**Fix:** Replace the maintenance kit, clean the interior of the unit, verify output
quality before closing out.

---

## 3. Management interface down, host itself fine

**Symptom:** Out-of-band management interface (e.g., Lenovo XCC, BMC/IPMI-style
controller) won't load via browser at the unit's IP, even though the host
OS/workload itself is unaffected.

**Root cause:** A dedicated management/root-of-trust module can fail independently
of the rest of the system.

**Check:**
```
ping <management-ip>
curl -k -I https://<management-ip>
```
If the management IP doesn't respond at the network level at all (not just an
empty/error page), that points more strongly at the management module itself
rather than a software/certificate issue on the interface.

**Fix:** Replace the specific FRU responsible (e.g., Root of Trust module), and
verify by actually logging into the interface via browser post-replacement — not
just confirming the part is physically seated.

---

## 4. Power supply failure alarm — check cabling first

**Symptom:** PSU failure alert/FRU suggestion from monitoring, sometimes across
multiple units simultaneously.

**Root cause:** Not every PSU alarm is a dead PSU — a disconnected or loose power
cable produces the identical alert. Especially with multiple units faulting at
once, a true simultaneous multi-unit hardware failure is far less likely than a
cabling issue introduced during other work nearby.

**Check:** Physically inspect cable seating at the PSU and PDU/outlet on each
affected unit before ordering a part. Where available, confirm power status via
the management console/CLI rather than the alarm alone, e.g.:
```
ipmitool sdr type "Power Supply"
```

**Fix:** Reseat/reconnect power cabling, verify power status shows healthy on each
affected unit, confirm with requestor before closing — no FRU replacement needed
if cabling was the cause.

---

## 5. Diagnostic tool won't connect after a repair

**Symptom:** Service/diagnostic software fails to connect to a unit immediately
after a repair (e.g., Apple Service Toolkit, similar vendor diagnostic tools).

**Root cause:** Not always a repair defect — often a connectivity issue between the
service equipment and the unit itself (cable, port, driver), rather than something
wrong with the repair.

**Check:**
1. Swap the cable and/or port on the service laptop first — isolates whether the
   fault is on the laptop/cable side.
2. Check the service laptop's Device Manager (or equivalent) for a driver error on
   the relevant interface.
3. Try a known-working unit against the same cable/port if one is available, to
   confirm the service-side setup itself is functional.

**Practice note:** Document this as a general connectivity issue between service
equipment and the unit rather than attributing it to a specific cause you haven't
confirmed — keep the write-up factual about what's known vs. what still needs
follow-up.
