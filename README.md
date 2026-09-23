# Field Notes: Common Infrastructure Fixes

Patterns I've run into doing hands-on field work and homelab troubleshooting across
networking, storage, servers, and hardware repair. Organized by symptom.

Client names, work order numbers, and internal case IDs are intentionally left out —
this is the technical pattern only.

---

## 1. "Client connects to WiFi but can't get out" (Layer 2 up, Layer 3 down)

**Symptom:** Device associates to the SSID fine, shows connected, but has no actual
network access.

**Root cause:** The radio/AP side (Layer 2) and the routed VLAN interface (Layer 3)
are separate administrative states on Cisco gear. A client can join an SSID
successfully while the VLAN's SVI (e.g. `interface Vlan1`) sits administratively
down — no gateway, no DHCP relay, no route out.

**Check:**
```
show ip interface brief
```
Look for the client VLAN's interface showing `administratively down` even though
physical/radio interfaces show up.

**Fix:**
```
interface Vlan1
 no shutdown
```

**Also check:** wireless radios can be separately shut down
(`ap dot11 24ghz shutdown` / `ap dot11 5ghz shutdown`). Bringing radios up doesn't
bring the VLAN interface up — they're two independent failure points that can both
be present at once.

---

## 2. Total internet outage after a config change (WAN drops)

**Root cause:** VLAN subinterfaces or other config accidentally applied to the
WAN-facing interface instead of the LAN-facing one. Disrupts the DHCP lease and/or
removes the default route.

**Check:**
```
show ip interface brief
show ip route
```

**Fix:** Strip the bad config off the WAN interface, power-cycle the upstream modem
if the lease won't return cleanly, bounce the interface:
```
interface GigabitEthernet0/0
 shutdown
 no shutdown
```

---

## 3. DHCP lease won't come up / no WAN IP

**Root cause:** An inbound ACL on the WAN interface silently blocking DHCP client
traffic (UDP 68) or return TCP traffic.

**Check:**
```
show access-lists
```
Look for a WAN-facing ACL missing an explicit permit for UDP 68 or established TCP,
with a catch-all `deny ip any any` at the end. Rising match counters on the deny
lines confirm active blocking.

**Fix:**
```
ip access-list extended WAN-IN
 10 permit udp any eq 67 any eq 68
 15 permit tcp any any established
```

**Related:** duplicate DHCP pools from IOS treating pool names as case-sensitive —
check `show ip dhcp pool` if addressing looks inconsistent.

---

## 4. DNS resolution fails but pings by IP work fine

**Root cause:** `no ip domain-lookup` disables DNS resolution entirely, or the
router has no source-interface set for DNS so replies don't route back correctly.

**Fix:**
```
ip domain lookup
ip domain lookup source-interface GigabitEthernet0/0
```
Test with `ping google.com`, not just an IP — a hostname-ping failure with
successful IP pings is the tell.

---

## 5. New/replacement hardware — config doesn't behave as expected

**Root cause:** A config template pushed via automated provisioning (PnP or
similar) was built for a different hardware model/version and doesn't fully apply
cleanly. Symptoms show up piecemeal rather than as one obvious failure.

**Lesson:** worth confirming early on any hardware swap that the template being
pushed is actually validated for that specific model — don't assume automated
provisioning "just works."

**Also:** a working config isn't saved until you explicitly save it:
```
write memory
```
On day-zero provisioning setups, skipping this can leave the device unable to
clear its provisioning state, which can lock out management access later.

---

## 6. Vendor/support scope boundaries

Pulling a config off existing hardware yourself is basic device administration.
*Applying* a config to replacement hardware during an active vendor support case
is often reserved for their assigned engineer — not about trust, but
liability/process tied to the case. If a vendor resists something, worth
clarifying directly whether it's a policy boundary or just unresponsiveness,
since the right next step differs.

---

## 7. General diagnostic sequence

1. `show ip interface brief` — interface state first, always
2. `show running-config` (targeted greps) — confirm what's actually applied
3. `show ip route` — confirm the routing table matches expectations
4. `ping` by IP first, then by hostname — isolates DNS vs. routing
5. `show access-lists` — check match counters if traffic is unexpectedly blocked
6. Only then change config — and `write memory` once confirmed working

---

## 8. Storage — drive/shelf replacement verification

**Risk:** On multi-drawer storage shelves, it's easy to misidentify the failed
drive bay by eye — the fault LED can be on a different carrier than where you're
counting from, especially under time pressure.

**Practice:**
1. Confirm the asset serial number matches the work order before pulling
   anything.
2. Cross-reference the fault LED directly against the drawer/bay diagram rather
   than counting visually.
3. After replacement, a "bypassed" or similar alert at the same bay usually
   indicates a seating issue with the new drive, not a new unrelated failure —
   reseat before escalating.

---

## 9. GPU/compute host — detected component count below expected

**Symptom:** Management console reports fewer GPUs (or other hot-swap
components) detected than expected.

**Root cause — two distinct failure modes can produce the identical symptom:**
1. Physically present but improperly seated — fixable with a reseat.
2. A separate component with a thermal fault throwing an unrelated-looking flag
   at the same time.

**Process for a live/production host:**
1. Get explicit approval before touching anything, especially on SLA-bound
   accounts.
2. Use a graceful shutdown, not a force power-off, with live workloads.
3. Inspect each flagged component independently — confirm physical presence
   first, then detection.
4. Get independent confirmation the count is back to expected before closing
   out.

---

## 10. Printer — mechanical fault, part handling

**Lesson:** the part that ends up actually needed isn't always obvious until
you're inspecting the feed path — a part ordered as a precaution isn't always
the one used. Document clearly when a part was ordered but not needed, rather
than leaving it as an ambiguous open item.

---

## 11. Management interface down, host itself fine

**Symptom:** Out-of-band management interface (e.g., BMC/IPMI-style
controller) won't load, even though the host OS/workload is unaffected.

**Root cause:** A dedicated management/root-of-trust module can fail
independently of the rest of the system. Replace the specific FRU responsible,
and verify by actually logging into the interface post-replacement — not just
confirming the part is seated.

---

## 12. Power supply failure alarm — check cabling first

**Symptom:** PSU failure alert, sometimes across multiple units
simultaneously.

**Root cause:** Not every PSU alarm is a dead PSU — a disconnected or loose
power cable produces the identical alert. Especially with multiple units
faulting at once, a true simultaneous multi-unit hardware failure is far less
likely than a cabling issue. Check cabling before ordering a replacement part.

---

## 13. Diagnostic tool won't connect after a repair

**Symptom:** Service/diagnostic software fails to connect to a unit
immediately after a repair.

**Root cause:** Not always a repair defect — often a connectivity issue
between the service equipment and the unit (cable, port, driver). Isolate
which side is actually at fault before assuming the repair itself failed.

---

## 14. General troubleshooting methodology

The mental checklist that applies across every vendor and platform:

1. **Identify the problem** — gather info, check logs, ask about recent
   changes
2. **Establish a theory** — start with the simplest explanation
3. **Test the theory** — confirm before moving forward
4. **Plan the fix** — consider side effects, get approvals if needed
5. **Implement** (or escalate with full documentation if beyond scope)
6. **Verify functionality** — confirm the fix worked and nothing else broke
7. **Document** — always last, never skipped

**Common mistake:** jumping straight to the fix without testing a theory
first — leads to fixing the wrong thing or missing a second, unrelated fault
hiding behind the obvious one.

---

## 15. Printer — recurring toner/print-quality degradation

**Symptom:** Shadowy/faded print output; on a separate later visit to the
same unit, ink/toner smearing and not bonding to paper.

**Root cause:** Worn maintenance-kit components (drum/fuser/rollers) rather
than a single discrete part failure — this kind of gradual print-quality
issue is often kit-level wear, not one bad part.

**Fix:** Replace the maintenance kit, clean the interior of the unit, verify
output quality before closing out.

---

*A running set of field-tested patterns — updated as new ones come up.*
