# Troubleshooting Guide

Project **R075458**. Twelve failures that actually happen when building
this topology, in roughly the order you will hit them.

Work top-down: physical → VLAN → trunk → routing → DHCP → services → ACL.

---

## Symptom 1 — A PC cannot ping anything, not even its own gateway

**What you see**
```
Ping request could not find host 192.168.20.1
```
or four `Request timed out.` and the link light on the switch port is
red or amber.

**Likely causes**
1. The access port was never put into the department VLAN, so it is
   still in VLAN 1.
2. The port is one of the `Fa0/x – Fa0/23` range you shut down, i.e. the
   PC is plugged into a parked port.
3. The wrong cable type was used.

**Diagnose**

On the access switch:
```
show vlan brief
show interfaces status
show interfaces FastEthernet0/3
```

`show interfaces status` shows `disabled` for a shut port and
`notconnect` for a port with no working cable.

**Fix**
```
configure terminal
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 no shutdown
 end
write memory
```
If the PC is in a parked port, move the cable to `Fa0/1`–`Fa0/6` range
for that switch and re-check the port map in `configs/SW-<DEPT>.txt`.

---

## Symptom 2 — Same-VLAN pings work, all cross-VLAN pings fail

**What you see**
`ping 192.168.20.12` succeeds, `ping 192.168.30.11` gives
`Request timed out.` from every department.

**Likely cause**
The uplink between the access switch and `SW-DIST` — or between
`SW-DIST` and `R1-CORE` — is not actually trunking. The most common
version of this is forgetting `switchport mode trunk` on one end, so DTP
leaves it as an access port in VLAN 1.

**Diagnose**

On both switches:
```
show interfaces trunk
show interfaces FastEthernet0/24 switchport
```

If `show interfaces trunk` prints nothing, there is no trunk at all.

On the router:
```
show ip interface brief
show vlans
```

**Fix**
```
configure terminal
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,50,60,70,80,99
 no shutdown
 end
write memory
```

---

## Symptom 3 — `DHCP request failed` on every PC in one department

**What you see**
The IP Configuration window flashes `DHCP request failed` and the PC
keeps `0.0.0.0`.

**Likely causes**
1. That VLAN is missing from the trunk `allowed vlan` list somewhere
   along the path.
2. The router subinterface for that VLAN is missing or has the wrong
   `encapsulation dot1Q` VLAN ID.
3. The pool `network` statement does not match the subinterface subnet,
   so the router has no pool to answer from.

**Diagnose**

On the router:
```
show ip dhcp pool
show ip interface brief
show running-config | section dhcp
debug ip dhcp server events
```
(Stop the debug afterwards with `undebug all`.)

On the switches:
```
show interfaces trunk
```

**Fix** — make the pool network and the subinterface agree:
```
configure terminal
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
 exit
ip dhcp pool MECH_POOL
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 192.168.80.10
 end
write memory
```
Then on the PC: **Static → DHCP** again, or from the command prompt
`ipconfig /release` followed by `ipconfig /renew`.

---

## Symptom 4 — One PC gets an address from the *wrong* subnet

**What you see**
A CSE PC receives `192.168.30.x` instead of `192.168.20.x`.

**Likely cause**
The access port is in the wrong VLAN. DHCP is behaving perfectly — the
frame arrived tagged as VLAN 30, so the router answered from `ECE_POOL`.

**Diagnose**
```
show vlan brief
show mac address-table | include <last 4 of the PC MAC>
```

**Fix**
```
configure terminal
interface FastEthernet0/2
 switchport access vlan 20
 end
write memory
```
Then release/renew on the PC, and clear the stale lease on the router:
```
clear ip dhcp binding 192.168.30.13
```

---

## Symptom 5 — Everything works except one department, and its switch cannot be managed either

**What you see**
The whole VLAN is isolated. `ping 192.168.99.1` from that switch fails.
`show interfaces trunk` looks fine on both ends.

**Likely cause**
**Native VLAN mismatch.** One end says `native vlan 99`, the other is
still on the default `native vlan 1`. Untagged frames land in the wrong
VLAN and the switch SVI in VLAN 99 becomes unreachable.

**Diagnose**
```
show interfaces trunk
show interfaces FastEthernet0/24 switchport
```
Compare the `Native vlan` / `Trunking Native Mode VLAN` line on **both**
switches. Packet Tracer also logs:
```
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/24
```

**Fix** — set it identically on both ends *and* on the router:
```
configure terminal
interface FastEthernet0/24
 switchport trunk native vlan 99
 end
write memory
```
On the router the matching line is `encapsulation dot1Q 99 native` under
`GigabitEthernet0/0.99`.

---

## Symptom 6 — `nslookup` works but the browser shows "Host Name Unresolved"

**What you see**
`ping 192.168.80.20` succeeds, `ping www.university.local` succeeds, but
`http://www.university.local` fails in the PT web browser.

**Likely causes**
1. The **HTTP service on `SRV-WEB` is Off**.
2. `index.html` was emptied but never saved.
3. You typed `https://` and HTTPS is off.

**Diagnose**
- On `SRV-WEB`: **Services → HTTP** — is the radio button on **On**?
- From a PC: `ping 192.168.80.20` (network layer) then browse to
  `http://192.168.80.20` (application layer). If the IP works and the
  name does not, it is DNS. If neither works, it is the HTTP service.

**Fix**
Turn **HTTP** (and **HTTPS**) **On** on `SRV-WEB`, open `index.html`
→ **(edit)**, paste `web/index.html` from this repo, **Save**.

---

## Symptom 7 — DNS resolves nothing from any PC

**What you see**
```
nslookup www.university.local
*** Can't find server name for address 192.168.80.10: Timed out
```

**Likely causes**
1. DNS service on `SRV-DNS` is Off, or the A records were never added.
2. `SRV-DNS` has no default gateway, so it can answer only VLAN 80 and
   silently drops replies to every other subnet.
3. The DHCP pools are handing out the wrong `dns-server`.

**Diagnose**

On a PC:
```
ipconfig /all
ping 192.168.80.10
nslookup dns.university.local
```
`ipconfig /all` must show `DNS Servers . . . : 192.168.80.10`.

On the router:
```
show running-config | include dns-server
```

**Fix**
1. `SRV-DNS` → **Services → DNS → On**, re-add the five A records from
   `docs/ip-addressing-table.md` §6.
2. `SRV-DNS` → **Desktop → IP Configuration** → Default Gateway must be
   `192.168.80.1`. **This is the single most-missed field in the whole
   project.**
3. If a pool is wrong:
```
configure terminal
ip dhcp pool CSE_POOL
 dns-server 192.168.80.10
 end
write memory
```
Then release/renew on the affected PCs.

---

## Symptom 8 — Student lab PCs cannot reach *anything*, not just Admin

**What you see**
`PC-LAB1` cannot ping the gateway, cannot get DHCP, cannot browse.
Everything else in the campus is fine.

**Likely cause**
ACL 110 was mistyped or applied in the wrong direction. A classic error
is leaving out the final `permit ip any any`, which turns the implicit
deny into a total blackout. Another is applying it `out` instead of `in`.

**Diagnose**
```
show access-lists 110
show ip interface GigabitEthernet0/0.70
```
Look for `Inbound access list is 110`. Then watch the counters while
pinging — the hits should land on the `permit` lines, not the `deny`.

**Fix** — rebuild the ACL from scratch (numbered ACLs append, so the old
one must be removed first):
```
configure terminal
interface GigabitEthernet0/0.70
 no ip access-group 110 in
 exit
no access-list 110
```
Then paste **BLOCK 6** of `configs/R1-core-router.txt` again and re-apply
`ip access-group 110 in`.

---

## Symptom 9 — Everyone loses access to the servers after applying ACL 120

**What you see**
Ping to `192.168.80.20` fails campus-wide; only Administration still
works.

**Likely cause**
The `permit icmp` line or the service `permit` lines were dropped, so the
`deny ip any 192.168.80.0 0.0.0.255` line catches everything. Or the ACL
was applied `in` on `Gi0/0.80` instead of `out` — inbound on that
subinterface filters traffic **leaving** the server VLAN, which breaks
every reply.

**Diagnose**
```
show access-lists 120
show ip interface GigabitEthernet0/0.80
```
The interface must report `Outgoing access list is 120`.

**Fix**
```
configure terminal
interface GigabitEthernet0/0.80
 no ip access-group 120 in
 ip access-group 120 out
 end
write memory
```
And confirm `permit icmp any 192.168.80.0 0.0.0.255` sits **above** the
`deny ip any 192.168.80.0 0.0.0.255` line. Order is everything.

---

## Symptom 10 — A port stops passing traffic after moving a PC

**What you see**
The PC used to work. You moved it to a different port (or swapped two
PCs) and now it gets nothing. The link light is green.

**Likely cause**
**Port security sticky MAC.** The port has pinned the MAC of the previous
PC; the new MAC exceeds `maximum 1` and is dropped by
`violation restrict`.

**Diagnose**
```
show port-security
show port-security interface FastEthernet0/1
show port-security address
```
`Security Violation Count` will be non-zero and `Last Source Address`
will be the *new* PC.

**Fix**
```
configure terminal
interface FastEthernet0/1
 no switchport port-security
 exit
interface FastEthernet0/1
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 end
clear port-security sticky interface FastEthernet0/1
write memory
```
Shut/no-shut the port if it does not relearn immediately.

> **Prevention:** finish all cabling before pasting the switch configs,
> and do not rearrange PCs after `write memory`.

---

## Symptom 11 — SSH to the router is refused from a PC that should be allowed

**What you see**
```
% Connection refused by remote host
```
even from an Administration PC.

**Likely causes**
1. RSA keys were never generated (the interactive `1024` prompt was
   skipped during the paste).
2. `transport input ssh` is set but no `username` exists, so
   `login local` has nothing to check.
3. The source PC is genuinely outside `192.168.10.0/24` and ACL 10 is
   doing its job.

**Diagnose**
```
show ip ssh
show crypto key mypubkey rsa
show running-config | section line vty
show access-lists 10
```
`show ip ssh` must say `SSH Enabled - version 2.0`. If it says
`SSH Disabled - version 1.99`, there is no RSA key.

**Fix**
```
configure terminal
ip domain-name university.local
crypto key generate rsa
1024
ip ssh version 2
username netadmin privilege 15 secret Adm1n@Univ#2025
line vty 0 4
 login local
 transport input ssh
 end
write memory
```
Note that `crypto key generate rsa` fails silently if the hostname is
still `Router` or the domain name is unset. Set both first.

---

## Symptom 12 — Everything worked, then the whole campus went down for 30 seconds

**What you see**
Intermittent loss after plugging in a cable, and log messages such as
`%SPANTREE-2-BLOCK_BPDUGUARD` or ports flapping to `err-disabled`.

**Likely causes**
1. A second cable was accidentally run between two switches, creating a
   loop. Spanning tree is reconverging.
2. A switch was plugged into a user access port that has
   `spanning-tree bpduguard enable`, so the port was err-disabled — which
   is BPDU Guard behaving correctly.

**Diagnose**
```
show spanning-tree summary
show spanning-tree vlan 20
show interfaces status err-disabled
show logging
```

**Fix**
- Remove the duplicate cable. This design is a strict star; there must be
  exactly **one** link between `SW-DIST` and each access switch.
- Recover an err-disabled port:
```
configure terminal
interface FastEthernet0/5
 shutdown
 no shutdown
 end
```
- Confirm `SW-DIST` is still the root:
```
show spanning-tree vlan 20
```
It must print `This bridge is the root`.

---

## Quick diagnostic ladder

Run these in order; the first one that fails is your layer.

| Layer | Command | Run on |
|-------|---------|--------|
| 1 — Physical | `show interfaces status` | access switch |
| 2 — VLAN | `show vlan brief` | access switch |
| 2 — Trunk | `show interfaces trunk` | both switches |
| 2 — STP | `show spanning-tree vlan <id>` | SW-DIST |
| 3 — Interface | `show ip interface brief` | R1-CORE |
| 3 — Routing | `show ip route` | R1-CORE |
| 3 — Addressing | `ipconfig /all` | PC |
| 4 — DHCP | `show ip dhcp binding` | R1-CORE |
| 4 — Filtering | `show access-lists` | R1-CORE |
| 7 — Naming | `nslookup www.university.local` | PC |
| 7 — Web | `http://192.168.80.20` then by name | PC browser |

## Reset switches

If a switch is beyond repair, wipe it and re-paste its file from
`configs/`:

```
enable
erase startup-config
delete vlan.dat
reload
```

Answer `y` / press Enter at the prompts. **`delete vlan.dat` matters** —
`erase startup-config` alone does *not* remove the VLAN database, and the
switch will come back with the old VLANs still defined.
