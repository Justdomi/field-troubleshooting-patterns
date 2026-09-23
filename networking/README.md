# Networking

Cisco IOS patterns — Layer 2/3 boundary issues, WAN/DHCP faults, ACL blocking, DNS
resolution, provisioning templates, and a general diagnostic sequence.

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

**Symptom:** WAN connectivity drops entirely immediately following a config push.

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

**Symptom:** WAN interface never receives an IP via DHCP.

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

**Symptom:** `ping <ip>` succeeds, `ping <hostname>` fails.

**Root cause:** `no ip domain-lookup` disables DNS resolution entirely, or the
router has no source-interface set for DNS so replies don't route back correctly.

**Check:**
```
show run | include ip domain
```

**Fix:**
```
ip domain lookup
ip domain lookup source-interface GigabitEthernet0/0
```
Test with `ping google.com`, not just an IP — a hostname-ping failure with
successful IP pings is the tell.

---

## 5. New/replacement hardware — config doesn't behave as expected

**Symptom:** Symptoms show up piecemeal rather than as one obvious failure after a
hardware swap with automated provisioning.

**Root cause:** A config template pushed via automated provisioning (PnP or
similar) was built for a different hardware model/version and doesn't fully apply
cleanly.

**Check:** Confirm the template version/target model against the actual hardware
model being provisioned before assuming the push succeeded cleanly.

**Fix / lesson:** worth confirming early on any hardware swap that the template
being pushed is actually validated for that specific model — don't assume
automated provisioning "just works."

**Also:** a working config isn't saved until you explicitly save it:
```
write memory
```
On day-zero provisioning setups, skipping this can leave the device unable to
clear its provisioning state, which can lock out management access later.

---

## 6. Vendor/support scope boundaries

**Symptom:** A vendor engineer declines to apply a config you've already pulled and
prepared, during an active support case.

**Root cause:** Pulling a config off existing hardware yourself is basic device
administration. *Applying* a config to replacement hardware during an active vendor
support case is often reserved for their assigned engineer — not about trust, but
liability/process tied to the case.

**Check:** Ask directly whether the hesitation is a policy boundary tied to the
case, or just unresponsiveness — the right next step differs completely depending
on which it is.

---

## 7. General diagnostic sequence

1. `show ip interface brief` — interface state first, always
2. `show running-config` (targeted greps) — confirm what's actually applied
3. `show ip route` — confirm the routing table matches expectations
4. `ping` by IP first, then by hostname — isolates DNS vs. routing
5. `show access-lists` — check match counters if traffic is unexpectedly blocked
6. Only then change config — and `write memory` once confirmed working
