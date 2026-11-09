# The Whole Project, Explained

**University Network with Multiple Subnets** (project R075458), built in Cisco Packet Tracer 9.0.1.
Team: P. Chandhra Sekhar, B. Ravi Kishore, D. Shasank, V. Yashaswee. Faculty: Shinu Ma'am.

Read this once from top to bottom and you can answer almost anything about the
project. Section 1 is the big picture. Section 2 explains every networking idea the
project uses. Sections 3-5 explain each part. Section 6 follows real packets through
the network, which is the best way to *understand* it rather than memorise it.

---

## 1. The big picture

### The problem

A university has eight departments: Administration, Computer Science, Electronics,
Mechanical, Library, Faculty, Student Labs and a Server Room. They all share one
building network. If we simply plugged everything into one network:

- a student lab PC could reach the Administration and Faculty machines directly,
- one faulty PC could flood **every** machine on campus with broadcasts,
- there would be no single place to put security rules,
- the campus would have no protected connection to the Internet and no safe place for a public website,
- anyone plugging a laptop into a classroom socket could attack the PCs around them.

### Our answer, in three layers

| Layer | Name in the project | One-line summary |
|---|---|---|
| 1 | **Part A: the campus** | Split the campus into 8 separate networks (VLANs), join them with one router that hands out addresses and enforces rules |
| 2 | **Extension 1: edge firewall + DMZ** | Put a firewall between the campus and the Internet, with a separate safe zone for the public website |
| 3 | **Extension 2: Layer-2 security** | Make the access switches themselves stop attacks that happen inside one classroom |

Each layer stops a different kind of attacker:

| Attacker | Stopped by |
|---|---|
| A student trying to reach Admin or Faculty PCs | Part A (router ACLs) |
| Someone on the Internet | Extension 1 (firewall) |
| Someone who has hacked our public web server | Extension 1 (DMZ rules) |
| A student plugging a rogue device into a lab socket | Extension 2 (switch security) |

This is called **defence in depth**: several independent layers, so if one fails the
others still protect the network.

### What is in the simulation

| | Count | Devices |
|---|---|---|
| Routers | 2 | R1-CORE (campus core), ISP-RTR (simulated Internet provider) |
| Firewall | 1 | ASA-EDGE (Cisco ASA 5506-X) |
| Switches | 11 | SW-DIST, 8 department switches, SW-DMZ, ISP-SW |
| PCs | 41 | 36 department PCs, EXT-PC (Internet user), SPOOF-PC, FLOOD-1/2/3 |
| Servers | 6 | SRV-DNS, SRV-WEB, SRV-APP, DMZ-WEB, EXT-SRV, ROGUE-DHCP |
| Hub | 1 | FLOOD-HUB |
| **Total** | **62 devices, 61 cables** | |

---

## 2. The ideas the project uses (glossary)

Learn these and every part of the project makes sense.

| Term | What it means, simply | Where we use it |
|---|---|---|
| **IP address / subnet** | An address like 192.168.20.15. A *subnet* (e.g. 192.168.20.0/24) is a group of addresses that can talk directly; `/24` means the first three numbers identify the network | Every department has its own /24 |
| **Default gateway** | The router address a PC sends to when the destination is *not* in its own subnet | `.1` in every department, e.g. 192.168.20.1 |
| **VLAN** | A *virtual* LAN: one physical switch split into separate networks. PCs in different VLANs cannot talk without a router | VLANs 10-80 for departments, 99 management, 999 unused, 100 DMZ |
| **Access port** | A switch port that belongs to exactly one VLAN; for PCs | Fa0/1 onwards on each department switch |
| **Trunk / 802.1Q** | A link that carries many VLANs; each frame gets a small *tag* saying which VLAN it belongs to | Every switch-to-switch link and the link to the router |
| **Native VLAN** | The one VLAN whose frames cross a trunk *untagged*. Default is 1; leaving it at 1 enables "VLAN hopping" attacks | We use VLAN 99 |
| **Router-on-a-Stick** | One physical router port split into *subinterfaces*, one per VLAN, each with that VLAN's gateway address | R1-CORE Gi0/0.10 ... Gi0/0.99 |
| **Inter-VLAN routing** | Moving packets between VLANs; only a router (Layer 3) can do it | R1-CORE |
| **TTL** | A counter in every packet, reduced by 1 at each router. Lets us *prove* a packet crossed a router | 128 same VLAN, 127 across VLANs |
| **ARP** | How a device finds the MAC (hardware) address for an IP address on its own network | Why the first ping usually loses one packet |
| **DHCP** | A server automatically gives each PC its IP address, mask, gateway and DNS server | R1-CORE, one pool per department |
| **DNS** | Turns names like www.university.local into IP addresses | SRV-DNS, zone `university.local` |
| **ACL (access control list)** | An ordered list of permit/deny rules on a router, checked top to bottom | ACL 10, 110, 120 on R1-CORE; OUTSIDE-IN on the ASA |
| **SSH** | Encrypted remote login. Telnet is the unencrypted version, which we disable | All routers, switches and the ASA |
| **Spanning tree (STP)** | Stops switching loops by blocking redundant paths | SW-DIST is the root |
| **PortFast / BPDU guard** | Access ports come up instantly; if a switch is plugged into a user port, the port shuts down | All access ports |
| **Port security** | Limits how many MAC addresses a port accepts | 1 MAC per port, "sticky" (learns the first), "restrict" (drops extras) |
| **Firewall** | A device that decides which traffic may cross between zones, and remembers connections | ASA-EDGE |
| **Security levels** | ASA zones get a number 0-100. Higher to lower is allowed; lower to higher is blocked unless an ACL allows it | inside 100, dmz 50, outside 0 |
| **Stateful inspection** | The firewall remembers every connection that went *out*, so it lets only the matching *replies* back in | ASA (`inspect icmp`, TCP/UDP automatically) |
| **NAT / PAT** | Network Address Translation: changing an address as a packet passes. PAT (Port Address Translation) lets many private PCs share one public address, told apart by port number | Whole campus leaves as 203.0.113.2 |
| **Static NAT** | One private address permanently mapped to one public address, so outsiders can reach it | DMZ-WEB 172.16.100.10 = 203.0.113.10 |
| **DMZ** | "Demilitarised zone": a separate zone for servers the Internet must reach, kept apart from the internal network | VLAN 100, DMZ-WEB |
| **Private vs public addresses** | 192.168.x.x, 172.16.x.x and 10.x.x.x are private: the Internet never routes them | Campus, transit link, DMZ |
| **DHCP snooping** | Switch feature: only *trusted* ports may send DHCP server replies; also builds a table of which IP/MAC is on which port | 7 department switches, uplink Fa0/24 trusted |
| **DAI (Dynamic ARP Inspection)** | Switch checks every ARP message against the snooping table and drops lies | 7 department switches (configured; see the PT limitation) |
| **err-disabled** | A switch shut a port automatically because of a security violation | ROGUE-DHCP's port Fa0/9 |

---

## 3. Part A: the campus network

### 3.1 The three tiers

```
                 R1-CORE (router)            <- CORE: gateways, DHCP, ACLs
                    | one trunk, all VLANs
                 SW-DIST (switch)            <- DISTRIBUTION: joins everything, STP root
   /    /    /    /    |    \    \    \
 ADMIN CSE ECE MECH  LIB  FAC  LAB  SRV      <- ACCESS: one switch per department
  PCs  PCs PCs  PCs  PCs  PCs  PCs  servers
```

- **Access switches** connect the PCs. Every PC port is an *access port* in that department's VLAN.
- **SW-DIST** connects all eight access switches with *trunks* and is the spanning-tree root.
- **R1-CORE** is the only device that can move traffic *between* departments.

### 3.2 Addressing

| VLAN | Department | Subnet | Gateway | Devices |
|---|---|---|---|---|
| 10 | Administration | 192.168.10.0/24 | .1 | 4 PCs |
| 20 | Computer Science | 192.168.20.0/24 | .1 | 6 PCs |
| 30 | Electronics | 192.168.30.0/24 | .1 | 6 PCs |
| 40 | Mechanical | 192.168.40.0/24 | .1 | 4 PCs |
| 50 | Library | 192.168.50.0/24 | .1 | 4 PCs |
| 60 | Faculty | 192.168.60.0/24 | .1 | 4 PCs |
| 70 | Student Labs | 192.168.70.0/24 | .1 | 8 PCs (+ attack lab) |
| 80 | Server Room | 192.168.80.0/24 | .1 | DNS .10, Web .20, App/FTP .30 |
| 99 | Management | 192.168.99.0/24 | .1 | one address per switch |
| 999 | Parking | none | none | every unused port, shut down |

Pattern inside every user subnet: `.1` gateway, `.2-.10` reserved (never handed
out), `.11` onwards given to PCs by DHCP.

### 3.3 Router-on-a-Stick

R1-CORE's port Gi0/0 has **no IP address**. It is divided into nine subinterfaces:
Gi0/0.10 has 192.168.10.1 and handles VLAN 10 tags, Gi0/0.20 has 192.168.20.1 and
handles VLAN 20, and so on. So one cable carries every department, and the router is
the gateway for all of them. `show ip route` lists every subnet as **directly
connected**, so no routing protocol (like OSPF) is needed.

### 3.4 Services

| Service | Where | What it does |
|---|---|---|
| **DHCP** | R1-CORE | 7 pools (one per user VLAN). Each gives address, mask, gateway, DNS 192.168.80.10, domain `university.local`. Servers are static |
| **DNS** | SRV-DNS 192.168.80.10 | `www`, `dns`, `portal`, `ftp` under `university.local`; plus `www.university.edu` and `www.example.com` for the extension |
| **Web** | SRV-WEB 192.168.80.20 | The campus portal page |
| **App / FTP** | SRV-APP 192.168.80.30 | Second web page (`portal`) and FTP (user `student`) |

### 3.5 Security in Part A

| Control | What it does | Why |
|---|---|---|
| **ACL 110** inbound on the lab subinterface (Gi0/0.70) | Lab PCs may use DHCP, DNS and the web servers; then **deny** Admin (10) and Faculty (60); then permit everything else | Students cannot reach sensitive departments but keep normal services |
| **ACL 120** outbound on the server subinterface (Gi0/0.80) | Admin gets full access to servers; everyone else gets only DNS, HTTP, HTTPS, FTP and ping | Servers expose only the services they offer |
| **ACL 10** on the VTY (remote-login) lines | Only 192.168.10.0/24 may log in remotely | Only Administration can manage devices |
| **SSH only** | `transport input ssh`, user `netadmin`; Telnet disabled | Passwords never cross the network in clear text |
| Enable secret, console password, timeouts | Every device | Physical and privileged access control |
| **Port security** | 1 MAC per port, sticky, restrict | A swapped or extra machine is dropped and counted |
| **PortFast + BPDU guard** | Access ports | A rogue switch on a user port shuts the port |
| Unused ports in VLAN 999 + shutdown | All switches | No live unused socket anywhere |
| Native VLAN 99 | All trunks | Removes the VLAN-hopping attack |

Why ACL 110 is **inbound** and ACL 120 is **outbound**: inbound on the lab stops a
packet as close to its source as possible. Outbound on the servers lets one list
protect the servers from all seven other VLANs at once.

---

## 4. Extension 1: edge firewall, NAT and DMZ

### 4.1 Why

Part A had no connection to the outside world. A real campus needs Internet access,
a public website, and protection from the Internet. The public website must not sit
next to our internal servers, because anything the Internet can reach can be hacked.

### 4.2 Layout

```
   EXT-PC 198.51.100.10      EXT-SRV 198.51.100.20     <- "the Internet"
            \                  /
             ISP-SW -- ISP-RTR 203.0.113.1              <- simulated ISP
                          |
              outside 203.0.113.2 (level 0)
   SW-DMZ -- dmz [ ASA-EDGE ]                          <- firewall
   DMZ-WEB   172.16.100.1 (50)
   172.16.100.10    inside 172.16.0.2 (level 100)
                          |  172.16.0.0/30
                    R1-CORE Gi0/1 172.16.0.1           <- campus (Part A)
```

Addresses `203.0.113.0` and `198.51.100.0` are *documentation ranges* (RFC 5737),
the correct choice for a lab imitating public addresses.

### 4.3 How it works

| Piece | Configuration | Effect |
|---|---|---|
| **Zones** | inside 100, dmz 50, outside 0 | Higher to lower is allowed. Lower to higher is blocked unless an ACL allows it |
| **Routing** | R1: default route to the ASA. ASA: default route to the ISP, and 192.168.0.0/16 back to R1 | Campus traffic for unknown destinations goes to the firewall |
| **PAT** | `nat (inside,outside) dynamic interface` | Every campus PC appears on the Internet as 203.0.113.2 |
| **Static NAT** | DMZ-WEB 172.16.100.10 = 203.0.113.10 | The Internet can reach the public website |
| **Outside ACL** `OUTSIDE-IN` | permit TCP port 80 to 172.16.100.10 only | Internet users get the website and nothing else. The ACL names the *real* address because ASA 8.3+ ACLs match before translation |
| **Inspection** | `inspect icmp`, http, ftp | Replies to campus traffic come back; nothing unsolicited does |
| **ASA management** | SSH only, only from 192.168.10.0/24, only on inside | The firewall is managed only from Administration |
| **SW-DMZ** | No management IP, port security | A hacked DMZ server cannot attack the switch |
| **ISP-RTR** | No route to any 192.168 network | Proves the campus is only reachable *through* NAT |
| **Split DNS** | Campus DNS: www.university.edu = 172.16.100.10. Internet DNS: = 203.0.113.10 | Each side gets the address that works for it |

### 4.4 What is allowed and what is blocked

| From | To | Result | Why |
|---|---|---|---|
| Campus (100) | Internet (0) | Allowed | Higher to lower; PAT translates |
| Campus (100) | DMZ (50) | Allowed | Higher to lower |
| Internet (0) | DMZ web, HTTP | **Allowed** | Outside ACL permits port 80 |
| Internet (0) | DMZ web, ping | Blocked | Only HTTP is permitted |
| Internet (0) | Campus | Blocked | Private addresses, no route, no rule |
| DMZ (50) | Campus (100) | **Blocked** | Lower to higher, no rule. The key reason for a DMZ |
| DMZ (50) | Internet (0) | Allowed | Higher to lower |

Tests: **18 of 18 pass** (`docs/edge-firewall-dmz.md`, F1-F18).

---

## 5. Extension 2: Layer-2 security and the attack lab

### 5.1 Why

The router and the firewall work at **Layer 3** (IP). An attacker sitting in the
student lab attacks the PC next to them **inside one switch**. That traffic never
reaches a router or the firewall, so neither can stop it. The switch has to.

### 5.2 Attacks and defences

| Attack | What the attacker does | What could go wrong | Defence | Result in our network |
|---|---|---|---|---|
| **Rogue DHCP** | Runs its own DHCP server | Victims get the attacker as their gateway, so all their traffic passes through the attacker (man-in-the-middle) | **DHCP snooping**: only Fa0/24 (uplink to the real DHCP server) is trusted; offers on user ports are dropped | **Blocked**. PC-LAB2 still gets gateway .1. Also, the rogue went over the 10 packets/second limit and its port **Fa0/9 was err-disabled** |
| **DHCP starvation** | Floods fake DHCP requests | The pool runs out; real users get no address | **DHCP rate limit**: 10 DHCP packets/second per user port | Configured on all user ports |
| **ARP spoofing** | Claims an IP it was never given (e.g. the gateway's) with forged ARP replies | Victims send their traffic to the attacker | **DAI**: every ARP on a user port is checked against the snooping binding table | Configured, but **Packet Tracer 9.0.1 does not enforce it** (counters stayed 0; IP Source Guard is rejected). Works on real 2960s |
| **MAC flooding** | Sends frames from many fake MAC addresses | The switch's MAC table fills, and it starts flooding every frame to every port like a hub, so the attacker sees everyone's traffic | **Port security**: max 1 MAC per port | **Blocked**. One FLOOD PC got an address; the other two got nothing; 21 violations counted |

### 5.3 The attack lab (beside SW-LAB)

| Device | Port | Setup | Plays the role of |
|---|---|---|---|
| ROGUE-DHCP | Fa0/9 | Static 192.168.70.250, DHCP pool .100+, gateway .250 | Rogue DHCP server |
| SPOOF-PC | Fa0/10 | Static 192.168.70.5, never leased | ARP spoofer |
| FLOOD-HUB + FLOOD-1/2/3 | Fa0/11 | Three PCs behind one port | MAC-flooding tool |

### 5.4 Key design choices

- Applied to the **7 department switches** with DHCP users, **not** the server switch:
  servers are static, so they have no bindings and DAI would silence them.
- `no ip dhcp snooping information option`: snooping normally inserts "option 82";
  because R1 is the DHCP server itself (not a relay), it would reject those
  requests and every PC would lose its address.
- After enabling snooping, **every PC had to renew its address**, because the binding
  table is only built from leases the switch actually sees.
- Port security stays in **restrict** mode: bad frames are dropped and counted while
  the real user keeps working.

Tests: **12 of 13 pass** (`docs/l2-security.md`, L1-L12 + L4b). L6 (ARP spoofing) is
the documented Packet Tracer limitation.

---

## 6. Follow the packets

### 6.1 A CSE student opens www.example.com (campus to Internet)

1. **Getting an address.** PC-CSE1 broadcasts a DHCP request. SW-CSE tags it VLAN 20,
   it reaches R1-CORE's Gi0/0.20, and R1 answers from the VLAN 20 pool:
   192.168.20.x, gateway 192.168.20.1, DNS 192.168.80.10. SW-CSE records the IP,
   MAC and port in its **snooping binding table**.
2. **Name lookup.** The PC asks 192.168.80.10 for `www.example.com`. That is another
   subnet, so it goes to the gateway: up the trunk tagged **VLAN 20**, routed by R1 to
   Gi0/0.80, down tagged **VLAN 80** to SRV-DNS. ACL 120 allows DNS. Answer:
   198.51.100.20.
3. **To the router.** The PC sends the web request to its gateway. R1 looks up
   198.51.100.20: not a campus network, so it uses the **default route** to the ASA
   (172.16.0.2) out of Gi0/1.
4. **Through the firewall.** Inside (100) to outside (0) is allowed. The ASA rewrites
   the source address 192.168.20.x to **203.0.113.2** and a port number (**PAT**),
   and records this in its translation table (`show xlate`).
5. **Across the Internet.** ISP-RTR delivers it to EXT-SRV, which replies to
   203.0.113.2.
6. **The way back.** The ASA matches the reply to its translation entry, rewrites the
   destination back to 192.168.20.x and lets it in, because it belongs to a connection
   the campus started (**stateful inspection**). R1 sends it down VLAN 20 to the PC.

The ISP has **no route** to 192.168.x.x, so step 6 only works because of NAT.

### 6.2 An Internet user opens www.university.edu (Internet to DMZ)

1. EXT-PC asks EXT-SRV's DNS: www.university.edu = **203.0.113.10** (public address).
2. ISP-RTR delivers the packet to the ASA's outside interface.
3. The ASA translates 203.0.113.10 to the real address **172.16.100.10** (static NAT)
   and checks **OUTSIDE-IN**: TCP port 80 to 172.16.100.10 is permitted.
4. DMZ-WEB answers; the reply is translated back to 203.0.113.10. The page loads.
5. If EXT-PC **pings** 203.0.113.10 instead, OUTSIDE-IN has no rule for ICMP, so the
   ping is dropped. If it pings **192.168.20.11**, ISP-RTR has no route: unreachable.

### 6.3 A hacker controls DMZ-WEB and tries the campus

DMZ-WEB pings 192.168.20.11. The packet arrives at the ASA's DMZ interface (level 50)
heading for inside (level 100). Lower to higher with no ACL: **dropped**. A
compromised public server cannot reach a single campus machine.

### 6.4 A lab student tries to reach Administration

PC-LAB1 pings 192.168.10.11. The packet reaches R1 on Gi0/0.70, where **ACL 110** is
checked inbound: it is not DHCP, DNS or web, and it matches "deny any to
192.168.10.0", so R1 drops it and replies **Destination host unreachable**. The
same PC can still browse the portal, because those lines come first in the ACL.

### 6.5 The rogue DHCP server in the lab

PC-LAB2 broadcasts a DHCP request. Both the real router and ROGUE-DHCP answer. The
rogue's offer arrives on **Fa0/9, an untrusted port**, so SW-LAB drops it; the
router's offer arrives on trusted Fa0/24 and is delivered. PC-LAB2 gets gateway .1.
Because the rogue answers every request, it exceeds 10 DHCP packets per second, and
SW-LAB **err-disables Fa0/9**: the red light on that link.

### 6.6 The MAC-flooding hub

FLOOD-1, -2 and -3 all send frames into one switch port, Fa0/11. Port security
(maximum 1, sticky) learned the first MAC it saw and pins it. Frames from the other
two MACs are dropped and counted (violation count 21 in our test), so those PCs never
get an address. The MAC table holds exactly one entry for Fa0/11 and can never fill.

---

## 7. What else is in the repository

| Folder / file | Contents |
|---|---|
| `packet-tracer/university-network.pkt` | The working simulation |
| `configs/` | One annotated configuration file per device: the source of truth |
| `paste/` | 44 ready-to-paste chunks to rebuild every device in order |
| `docs/Demonstration-Guide.docx` | The demo script, split for four presenters |
| `docs/edge-firewall-dmz.md`, `docs/l2-security.md` | Build, test, demo and viva notes for each extension |
| `docs/ip-addressing-table.md`, `device-list.md`, `cable-connection-list.md`, `topology-diagram.txt` | Reference tables |
| `docs/testing-results.md`, `troubleshooting.md` | Test records and fixes |
| `web/` | The two web pages served by DMZ-WEB and EXT-SRV |

## 8. Packet Tracer limitations to admit honestly

Saying these confidently in the viva shows you tested properly.

- **DAI is not enforced** in PT 9.0.1 (accepted, shown Active, counters always 0), and `ip verify source` (IP Source Guard) is rejected.
- A ping started **from the ASA itself** to the Internet times out; from any campus PC it works.
- `tracert` from campus shows `* * *` at the ISP hop: its reply is unsolicited, so the firewall drops it.
- ASA ACL hit counters reset when the file is reopened.
- The first ping on a new path loses a packet (ARP, and NAT setup).
