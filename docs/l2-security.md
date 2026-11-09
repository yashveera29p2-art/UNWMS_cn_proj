# Extension 2: DHCP Snooping, Dynamic ARP Inspection and MAC-Flooding Protection

Part A filters traffic between departments. Extension 1 (the ASA) guards
the Internet edge. Both work at Layer 3. Neither can see an attack made by
a machine **inside one VLAN** against its neighbours, because that traffic
never reaches a router or a firewall. This extension closes that gap on the
access switches.

| Attack (from a student PC or a device plugged into a lab socket) | What it does | Defence added |
|---|---|---|
| **Rogue DHCP server** | Hands out addresses with itself as the gateway, so every victim's traffic passes through the attacker (man-in-the-middle) | **DHCP snooping** - only the uplink to R1-CORE may answer DHCP |
| **ARP spoofing / poisoning** | Sends ARP replies saying "192.168.70.1 is at *my* MAC", stealing traffic meant for the gateway | **Dynamic ARP Inspection** - ARP is checked against the IP-to-MAC table snooping built |
| **DHCP starvation (DHCP flooding)** | Floods DHCP requests from fake MACs until the pool is empty and real users get no address | **DHCP snooping rate limit** - 10 DHCP packets per second per port |
| **MAC flooding** | Floods frames from thousands of fake MACs; a full MAC table makes the switch flood every frame to every port like a hub, so the attacker sees all traffic | **Port security** - each port learns one MAC; extra MACs are dropped and counted |

---

## 1. How the pieces fit

```
   R1-CORE  (the only real DHCP server)
      |
   SW-DIST
      |  Fa0/24 = TRUSTED   (DHCP offers and the router's ARP allowed)
   SW-LAB  -------------------------------------------------------------
      |  Fa0/1-8  untrusted   PC-LAB1..8     DHCP leases -> binding table
      |  Fa0/9    untrusted   ROGUE-DHCP     its OFFERs are dropped
      |  Fa0/10   untrusted   SPOOF-PC       ARP for an unleased IP dropped
      |  Fa0/11   untrusted   FLOOD-HUB -- FLOOD-1, FLOOD-2, FLOOD-3
      |                                     only 1 MAC allowed on the port
```

1. **DHCP snooping** splits ports into *trusted* (the uplink) and
   *untrusted* (everything a user can plug into). DHCP server messages
   (OFFER, ACK) are accepted only on trusted ports.
2. While watching real leases go past, snooping builds a **binding table**:
   IP address, MAC address, VLAN and port for every PC.
3. **DAI** reads that table. An ARP message from an untrusted port must
   match a binding exactly, or it is dropped. A PC can therefore only claim
   the address it was actually leased.
4. **Port security** (already configured in Part A, max 1 sticky MAC,
   violation restrict) caps every port at one MAC, which is precisely what
   defeats MAC flooding.

### Design decisions

| Decision | Why |
|---|---|
| Applied to the 7 access switches with DHCP users | That is where the users, and therefore the attackers, are |
| **Not** applied to SW-SRV, SW-DMZ, SW-DIST | Servers are statically addressed, so they have no DHCP binding and DAI would drop their ARP. SW-DIST has no user ports |
| `no ip dhcp snooping information option` | Snooping normally inserts option 82. R1-CORE is the DHCP server itself (not a relay) and IOS discards option-82 requests with a zero relay address, which would break DHCP campus-wide |
| Rate limit 10 pps | A real PC sends 4 DHCP packets per lease; 10 per second never hurts a user but stops a flood |
| Violation mode stays `restrict` | Extra MACs are dropped and counted while the legitimate device keeps working |

### Packet Tracer 9.0.1 limitation: DAI is accepted but not enforced

Found during the SW-LAB pilot (9 Nov 2026):

- The 2960 accepts every `ip arp inspection` command. `show ip arp inspection` reports VLAN 70 as **Enabled / Active**.
- It never inspects anything. SPOOF-PC (192.168.70.5, no binding) cleared its ARP cache, ARPed for PC-LAB1 and got a reply. Every DAI counter stayed at **0**, including *Forwarded*, even for the 8 legitimate PCs' ARP.
- Adding `ip arp inspection validate src-mac dst-mac ip` changed nothing.
- IP Source Guard (`ip verify source`), DAI's companion feature, is rejected with `% Invalid input`.

**Decision:** DAI stays in all seven configs, because it is the correct design and works on real 2960s. For the demo, anti-spoofing is explained rather than shown live. DHCP snooping, the rate limit and port security *are* enforced by Packet Tracer and are demonstrated live.

---

## 2. New devices: the ATTACK LAB corner

Six devices, placed beside SW-LAB and boxed with a rectangle labelled
**"ATTACK LAB - contained by Layer-2 security"**.

| Device | PT model | Connects to | Cable | Settings |
|---|---|---|---|---|
| ROGUE-DHCP | Server-PT | SW-LAB **Fa0/9** | Straight-Through | Static `192.168.70.250` /24, gateway `192.168.70.250`. Services > DHCP **On**: Default Gateway `192.168.70.250`, DNS `192.168.70.250`, Start IP `192.168.70.100`, Subnet `255.255.255.0`, Max Users `50`, **Save**. All other services Off |
| SPOOF-PC | PC-PT | SW-LAB **Fa0/10** | Straight-Through | Static `192.168.70.5` /24, no gateway. `.5` sits in R1's DHCP-excluded range, so no lease and no binding can ever exist for it - a self-assigned, spoofed identity. (Packet Tracer refuses `192.168.70.1` because R1-CORE already owns it.) |
| FLOOD-HUB | Hub-PT (Network Devices > Hubs) | SW-LAB **Fa0/11** from hub **Port 0** | **Cross-Over** | none |
| FLOOD-1 | PC-PT | FLOOD-HUB Port 1 | Straight-Through | DHCP |
| FLOOD-2 | PC-PT | FLOOD-HUB Port 2 | Straight-Through | DHCP |
| FLOOD-3 | PC-PT | FLOOD-HUB Port 3 | Straight-Through | DHCP |

Totals become **62 devices** and **61 cables**.

---

## 3. Build order

The order matters: configure SW-LAB **before** plugging in ROGUE-DHCP, so a
rogue lease can never get in while the lab is unprotected.

### Step 1 - Pilot on SW-LAB

1. SW-LAB > CLI. Console password `Cons@2025`, then `enable`, `Univ@Lab#2025`.
2. Paste `paste/38_SW-LAB_l2-security.txt`.
3. **Renew all 8 lab PCs**: PC-LAB*n* > Desktop > IP Configuration > click
   **Static**, then **DHCP**. Each should get `192.168.70.x` (x >= 11)
   again.
4. Checkpoint on SW-LAB:
   ```
   show ip dhcp snooping
   show ip dhcp snooping binding
   show ip arp inspection
   ```
   Snooping enabled on VLAN 70, Fa0/24 trusted, **8 bindings**, DAI on
   VLAN 70.
5. PC-LAB1: `ping www.university.local` still works. If it does not, stop
   and see section 7 before going further.

### Step 2 - Build the ATTACK LAB corner

Place and cable the six devices from section 2, then set their IP and
services. ROGUE-DHCP's DHCP service is the last thing you switch on.

### Step 3 - Roll out to the other six departments

For each switch: console `Cons@2025`, `enable` with its own password, paste
its chunk, then renew its PCs.

| Switch | Enable password | Paste | PCs to renew |
|---|---|---|---|
| SW-ADMIN | `Univ@Admin#2025` | `39_SW-ADMIN_l2-security.txt` | PC-ADM1-4 |
| SW-CSE | `Univ@Cse#2025` | `40_SW-CSE_l2-security.txt` | PC-CSE1-6 |
| SW-ECE | `Univ@Ece#2025` | `41_SW-ECE_l2-security.txt` | PC-ECE1-6 |
| SW-MECH | `Univ@Mech#2025` | `42_SW-MECH_l2-security.txt` | PC-MECH1-4 |
| SW-LIB | `Univ@Lib#2025` | `43_SW-LIB_l2-security.txt` | PC-LIB1-4 |
| SW-FAC | `Univ@Fac#2025` | `44_SW-FAC_l2-security.txt` | PC-FAC1-4 |

Then **File > Save**.

---

## 4. Tests

Ignore the first lost ping of any test (ARP). Tests marked **ATTACK** are
attacks that must fail. Results recorded in Packet Tracer 9.0.1 on
9 Nov 2026: **11 of 12 pass**; L6 is the documented PT limitation.

| # | Test | Run on | Command / action | Expected | Result |
|---|---|---|---|---|---|
| L1 | Snooping state | SW-LAB | `show ip dhcp snooping` | Enabled, VLAN 70, Fa0/24 trusted, rate limit 10 on user ports | PASS - VLAN 70, Fa0/24 trusted/unlimited, Fa0/1-11 untrusted at 10 pps |
| L2 | Binding table | SW-LAB | `show ip dhcp snooping binding` | One line per lab PC (and FLOOD-1): IP, MAC, VLAN 70, port | PASS - 8 bindings, PC-LAB1-8 on Fa0/1-8 (.11-.18) |
| L3 | No collateral damage | PC-LAB1 | `ping www.university.local` | Replies | PASS - 3/4 then 4/4 (first loss = ARP) |
| L4 | **ATTACK** rogue DHCP | PC-LAB2 | IP Configuration: Static, then DHCP - three times | Always `192.168.70.x` with gateway **192.168.70.1**, never `.100-.149` / gateway `.250` | PASS - PC-LAB2 got .11, gateway .1, DNS 192.168.80.10 (rogue pool never used) |
| L5 | DAI configured | SW-LAB | `show ip arp inspection` | VLAN 70 Enabled / Active | PASS - VLAN 70 Enabled / Active |
| L6 | **ATTACK** ARP spoofing | SPOOF-PC | `arp -d`, `ping 192.168.70.12` | On real hardware: fails, Dropped above 0. **In PT 9.0.1: succeeds, counters stay 0** (see the limitation note in section 1) | PT LIMITATION - ping to .11, .13, .12 all replied; all DAI counters 0; `validate` changed nothing; `ip verify source` rejected |
| L7 | Victims unaffected | PC-LAB1 | `ping 192.168.70.1` | Gateway ping works | PASS - lab PCs still routed via .1 (TTL 127 replies in L3 and L11) |
| L8 | **ATTACK** MAC flooding | FLOOD-1, -2, -3 | `ipconfig` on each | Only one has a 192.168.70.x address; the others get none | PASS - FLOOD-1 got 192.168.70.19; FLOOD-2/-3 stayed on 169.254.x.x |
| L9 | Port security caught it | SW-LAB | `show port-security interface FastEthernet0/11` | Max 1, Security Violation Count above 0, port still Secure-up | PASS - Max 1, Total 1, Secure-up, Violation Count 21; Last Source = FLOOD-3's MAC |
| L10 | MAC table did not grow | SW-LAB | `show mac address-table interface FastEthernet0/11` | Exactly one MAC | PASS - only 00d0.d346.0268 (FLOOD-1) |
| L11 | Rollout healthy | One PC per department | `ping www.university.local` | Replies from all seven departments | PASS - ADMIN, CSE, ECE, MECH, LIB, FAC and LAB PCs all reply; SW-LIB shows 4 bindings |
| L12 | Firewall unaffected | PC-CSE1 browser | `http://www.example.com` | Page loads | PASS - www.example.com loads via PAT 203.0.113.2 |

### Optional: show the attack succeeding first

Run this before the demo if you want a "before and after":

1. SW-LAB: `configure terminal`, `no ip arp inspection vlan 70`,
   `no ip dhcp snooping vlan 70`, `end`.
2. PC-LAB2: Static, then DHCP, a few times. Sooner or later it accepts
   ROGUE-DHCP's offer: address `192.168.70.10x`, gateway `192.168.70.250`.
   The PC now sends everything to the attacker.
3. Put the protection back: `configure terminal`,
   `ip dhcp snooping vlan 70`, `ip arp inspection vlan 70`, `end`, then
   renew PC-LAB2 and every lab PC (DAI needs fresh bindings).

---

## 5. Demo script (about 4 minutes)

> **SAY:** The ACLs and the firewall work at Layer 3. But a student can
> attack the PC next to them without ever touching a router. This corner
> holds three attackers, and the switch stops each one by itself.

1. **Rogue DHCP.** Point at ROGUE-DHCP. Renew PC-LAB2.
   > **SAY:** That server is offering addresses with itself as the gateway.
   > PC-LAB2 still got gateway 192.168.70.1 from the real router, because DHCP
   > snooping only trusts the uplink.
   Show `show ip dhcp snooping binding`.
2. **ARP spoofing (explained, not run).** On SW-LAB, show
   `show ip dhcp snooping binding` and `show ip arp inspection`.
   > **SAY:** SPOOF-PC gave itself 192.168.70.5, an address nobody leased
   > it, the way an attacker forges ARP replies. DAI is enabled on VLAN 70
   > and checks every ARP against this binding table. There is no binding
   > for .5 on Fa0/10, so on a real switch its ARP is dropped. Packet
   > Tracer accepts the DAI configuration but does not simulate the
   > inspection itself, and it has no IP Source Guard either, so this
   > one is shown as configuration.
3. **MAC flooding.** `ipconfig` on FLOOD-1, -2, -3, then
   `show port-security interface FastEthernet0/11` and
   `show mac address-table interface FastEthernet0/11`.
   > **SAY:** A MAC-flooding tool behaves like this hub full of machines:
   > many MACs behind one port. Port security allows one. The others are
   > dropped and counted, the MAC table cannot fill, and the switch never
   > falls back to acting like a hub.

---

## 6. Viva questions

| Question | Short answer |
|---|---|
| Why trust only Fa0/24? | It is the only path to the real DHCP server (R1-CORE). Any DHCP offer arriving on a user port is by definition a rogue. |
| Why does DAI need DHCP snooping? | DAI has no idea which IP belongs to which MAC; the snooping binding table is its source of truth. |
| Why renew the PCs after enabling it? | Bindings are only created from leases the switch watches. Leases taken before snooping was on have no binding, so DAI would drop those PCs' ARP. |
| Why not on the server switch? | Servers are static, so they have no bindings and DAI would silence them. A static ARP ACL could cover them, but it adds maintenance for no gain in this design. |
| Why disable option 82? | The router is the DHCP server, not a relay. IOS drops requests carrying option 82 with a zero relay address, which would stop every PC getting an address. |
| What is MAC flooding and how is it stopped? | Filling the switch's MAC table with fake addresses so it floods all frames like a hub. Port security limits each port to one MAC, so the table cannot be filled from a user port. |
| Difference between MAC flooding and DHCP starvation? | MAC flooding attacks the switch's MAC table; DHCP starvation attacks the router's address pool. Port security stops the first, the snooping rate limit (and port security) stops the second. |
| Why `restrict` and not `shutdown`? | `restrict` drops only the offending frames and counts them; the legitimate user on that port keeps working. `shutdown` would err-disable the whole port until an admin intervenes. |

---

## 7. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| After the paste, lab PCs cannot ping anything | Old leases have no binding, so DAI drops their ARP | Renew each PC (Static, then DHCP) |
| PCs get no address at all (169.254.x.x) | Option 82 still on, or Fa0/24 not trusted | `show ip dhcp snooping` - check "Insertion of option 82 is disabled" and Fa0/24 trusted |
| `ip arp inspection` is rejected with `% Invalid input` | This PT build's 2960 lacks DAI | Keep DHCP snooping and port security; tell the examiner DAI needs a switch that supports it. Note it in the results |
| A PC got 192.168.70.10x / gateway .250 | It got a rogue lease while snooping was off | Turn snooping on, renew that PC |
| SPOOF-PC can still ping, DAI counters all 0 | Packet Tracer 9.0.1 does not enforce DAI (section 1) | Expected in PT. Explain it in the demo; do not try to "fix" it |
| PT says the IP is already in use when setting SPOOF-PC to `.1` | Packet Tracer blocks duplicate IPs | Use `192.168.70.5` as in section 2 - DAI drops it for the same reason |
| FLOOD-1 also has no address | Another FLOOD PC sent the first frame, so its MAC became the sticky one | Expected - whichever PC spoke first owns the port |
