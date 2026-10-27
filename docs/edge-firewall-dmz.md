# Extension: Edge Firewall, NAT and DMZ

This extension connects the campus to a simulated Internet through a
Cisco ASA 5506-X firewall, and adds a DMZ for a public web server.

The original campus already filters traffic *between departments*
(ACLs 10, 110 and 120 on R1-CORE). Until now it had no boundary at all
with the outside world. This extension adds that boundary.

| Problem | What solves it |
|---------|----------------|
| Campus uses private 192.168.x.x addresses, which the Internet will not route | **PAT** on the ASA: every campus host leaves as `203.0.113.2` |
| The university needs a public website without exposing the campus | **DMZ** zone with its own server, published by **static NAT** as `203.0.113.10` |
| Anything the Internet starts must be refused, except the website | **Security levels** + an outside ACL that permits **HTTP only** |
| A hacked public server must not become a way into the campus | DMZ (level 50) **cannot open connections** to inside (level 100) |
| Replies to campus traffic must still come back | **Stateful inspection** (`inspect icmp`, TCP/UDP tracked automatically) |
| The firewall itself needs protecting | **SSH only**, local user, only from 192.168.10.0/24, only on inside |

---

## 1. Design

```
                     "THE INTERNET"
        EXT-PC 198.51.100.10     EXT-SRV 198.51.100.20
                (web + DNS: www.example.com, www.university.edu)
                  \                 /
                   Fa0/1       Fa0/2
                      [ ISP-SW ]
                        Gi0/1
                          |
                        Gi0/1  198.51.100.1/24
                      [ ISP-RTR ]
                        Gi0/0  203.0.113.1/28
                          |
                        Gi1/1  outside  203.0.113.2/28   level 0
   SW-DMZ Gi0/1 ----- Gi1/3 [ ASA-EDGE ]
   172.16.100.0/24  dmz 172.16.100.1   level 50
        |               Gi1/2  inside  172.16.0.2/30     level 100
     Fa0/1                |
   DMZ-WEB              Gi0/1  172.16.0.1/30
   172.16.100.10      [ R1-CORE ]   (unchanged campus below)
                        Gi0/0 trunk -> SW-DIST -> 8 access switches
```

### New addressing

| Network | Purpose | Addresses |
|---------|---------|-----------|
| 172.16.0.0/30 | R1-CORE to ASA inside (transit) | .1 R1-CORE Gi0/1, .2 ASA Gi1/2 |
| 172.16.100.0/24 | DMZ | .1 ASA Gi1/3, .10 DMZ-WEB |
| 203.0.113.0/28 | University public block (from the ISP) | .1 ISP-RTR, .2 ASA outside (PAT), .10 DMZ-WEB published |
| 198.51.100.0/24 | Simulated Internet | .1 ISP-RTR, .10 EXT-PC, .20 EXT-SRV |

`203.0.113.0/24` and `198.51.100.0/24` are documentation ranges (RFC 5737),
the correct choice for a lab that imitates public addresses.

### New devices (7) and cables (7)

Totals become **56 devices** and **55 cables**.

| Device | Packet Tracer model | Where to find it |
|--------|---------------------|------------------|
| ASA-EDGE | ASA 5506-X | Network Devices > Security |
| ISP-RTR | 2911 | Network Devices > Routers |
| SW-DMZ | 2960-24TT | Network Devices > Switches |
| ISP-SW | 2960-24TT | Network Devices > Switches |
| DMZ-WEB | Server-PT | End Devices |
| EXT-SRV | Server-PT | End Devices |
| EXT-PC | PC-PT | End Devices |

| # | From | To | Cable |
|---|------|----|-------|
| 1 | R1-CORE Gi0/1 | ASA-EDGE Gi1/2 | Copper Cross-Over |
| 2 | ASA-EDGE Gi1/1 | ISP-RTR Gi0/0 | Copper Cross-Over |
| 3 | ASA-EDGE Gi1/3 | SW-DMZ Gi0/1 | Copper Straight-Through |
| 4 | DMZ-WEB Fa0 | SW-DMZ Fa0/1 | Copper Straight-Through |
| 5 | ISP-RTR Gi0/1 | ISP-SW Gi0/1 | Copper Straight-Through |
| 6 | EXT-PC Fa0 | ISP-SW Fa0/1 | Copper Straight-Through |
| 7 | EXT-SRV Fa0 | ISP-SW Fa0/2 | Copper Straight-Through |

If link 1 or 2 is still red 30 seconds after the ASA is configured, delete
it and redo it with Copper Straight-Through. The ASA's Gigabit ports
behave differently across Packet Tracer builds.

---

## 2. Build steps

Before you start: `git pull`, open `packet-tracer/university-network.pkt`,
wait for all lights to go green.

**Place every device and cable first, then configure.** That is the same
rule as the original build: port security uses sticky MAC learning, so
DMZ-WEB must be on SW-DMZ Fa0/1 before SW-DMZ is configured.

### Step 1 - Place, rename, cable

1. Place the seven devices in the table above. Put the Internet side above
   R1-CORE so the canvas reads top-to-bottom: Internet, firewall, campus.
2. Rename each one (click it, **Config** tab, **Display Name**). Use the
   names exactly: `ASA-EDGE`, `ISP-RTR`, `SW-DMZ`, `ISP-SW`, `DMZ-WEB`,
   `EXT-SRV`, `EXT-PC`.
3. Run the seven cables from the cable table, choosing the port explicitly
   each time (do not use the lightning-bolt auto cable).

### Step 2 - R1-CORE

R1-CORE is already configured, so only the new lines go in.

1. R1-CORE > CLI > press Enter.
2. Type `enable`, then password `Univ@Core#2025`.
3. Paste `paste/32_R1-CORE_edge-uplink.txt`.

**Checkpoint:** `show ip route` now shows
`S* 0.0.0.0/0 [1/0] via 172.16.0.2`. `Gi0/1` stays **down** until the
ASA is configured, which is expected.

### Step 3 - ASA-EDGE

1. ASA-EDGE > CLI > press Enter. You will see `ciscoasa>`.
2. Type `enable`. At `Password:` just press Enter.
3. Paste `paste/33_ASA-EDGE_part1.txt`, wait for the prompt.
4. Paste `paste/34_ASA-EDGE_part2.txt`. It ends with `write memory` and
   `[OK]`.

**Checkpoint (on the ASA):**

```
show interface ip brief
show nameif
show route
```

Gi1/1, Gi1/2 and Gi1/3 are up/up. `nameif` shows outside 0, inside 100,
dmz 50. Then on R1-CORE: `ping 172.16.0.2` succeeds (ignore the first
dot, ARP).

### Step 4 - ISP-RTR, ISP-SW, SW-DMZ

For each, CLI tab. On a brand-new device answer `no` to the initial
configuration dialog and press Enter. Then paste:

| Device | Paste |
|--------|-------|
| ISP-RTR | `paste/36_ISP-RTR.txt` |
| ISP-SW | `paste/37_ISP-SW.txt` |
| SW-DMZ | `paste/35_SW-DMZ.txt` |

**Checkpoint:** on the ASA, `ping 203.0.113.1` succeeds.

### Step 5 - Servers and the outside PC (GUI)

**DMZ-WEB**
1. Desktop > IP Configuration > **Static**: IP `172.16.100.10`,
   mask `255.255.255.0`, gateway `172.16.100.1`. Leave DNS empty.
2. Services > **HTTP**: HTTP On, **HTTPS Off**. Edit `index.html`, delete
   everything, paste the contents of `web/public.html`, Save.
3. Services > **DHCP**: Off. **DNS**: Off. **FTP**: Off, and delete the
   default `cisco` user. Also turn Off EMAIL, SYSLOG, AAA, NTP, TFTP if
   any show On. A public server runs only the service it publishes.

**EXT-SRV**
1. Static IP `198.51.100.20`, mask `255.255.255.0`, gateway
   `198.51.100.1`, DNS `198.51.100.20`.
2. Services > **DNS**: On. Add two A records:
   - `www.example.com` -> `198.51.100.20`
   - `www.university.edu` -> `203.0.113.10`  (the *published* address)
3. Services > **HTTP**: On. Replace `index.html` with `web/internet.html`.
4. DHCP Off, FTP Off.

**EXT-PC**
- Static IP `198.51.100.10`, mask `255.255.255.0`, gateway
  `198.51.100.1`, DNS `198.51.100.20`.

**SRV-DNS (existing campus DNS) - add two A records**
- `www.example.com` -> `198.51.100.20`
- `www.university.edu` -> `172.16.100.10`  (the *real* DMZ address)

The two DNS servers deliberately give **different answers** for
`www.university.edu`. This is *split DNS*: campus users are sent straight
to the DMZ address, and Internet users get the public NAT address.

### Step 6 - Tidy the existing servers (security clean-up)

Packet Tracer gives every server a default FTP account `cisco/cisco`.
- **SRV-WEB** > Services > FTP: turn FTP **Off** and delete user `cisco`.
- **SRV-APP** > Services > FTP: keep FTP On, delete user `cisco`, keep
  `student`.
- **SRV-DNS** > Services > HTTP and HTTPS: **Off** (it is a DNS server).

### Step 7 - Save

File > Save. Then from the project folder:

```
git add -A
git commit -m "Add edge firewall, NAT and DMZ (PT 9.0.1)"
git push
```

---

## 3. Tests

Run in this order. Ignore the first `Request timed out` of any ping (ARP).
Tests marked **FAIL** are meant to fail: that is the firewall working.

| # | Test | Run on | Command | Expected | Pass/Fail |
|---|------|--------|---------|----------|-----------|
| F1 | Default route | R1-CORE | `show ip route` | `S* 0.0.0.0/0 via 172.16.0.2` | |
| F2 | Zones | ASA-EDGE | `show nameif` | outside 0, inside 100, dmz 50 | |
| F3 | Campus to Internet (PAT) | PC-CSE1 | `ping 198.51.100.20` | Replies | |
| F4 | Translation table | ASA-EDGE | `show xlate` | PAT entry, 192.168.20.x to 203.0.113.2 | |
| F5 | Campus browses Internet | PC-CSE1 browser | `http://www.example.com` | example.com page | |
| F6 | Internet browses public site | EXT-PC browser | `http://www.university.edu` | University Public Website | |
| F7 | Outside ACL counter | ASA-EDGE | `show access-list OUTSIDE-IN` | hitcnt greater than 0 | |
| F8 | Internet pings public site | EXT-PC | `ping 203.0.113.10` | **FAIL** - timed out (only HTTP is published) | |
| F9 | Internet reaches campus | EXT-PC | `ping 192.168.20.11` | **FAIL** - unreachable (private address, no route) | |
| F10 | Internet reaches DMZ directly | EXT-PC | `ping 172.16.100.10` | **FAIL** - unreachable | |
| F11 | DMZ attacks campus | DMZ-WEB | `ping 192.168.20.11` | **FAIL** - timed out (level 50 to level 100) | |
| F12 | DMZ reaches Internet | DMZ-WEB | `ping 198.51.100.20` | Replies | |
| F13 | Campus reaches DMZ (split DNS) | PC-LAB1 browser | `http://www.university.edu` | Public website, via 172.16.100.10 | |
| F14 | Student labs still contained | PC-LAB1 | `ping 192.168.10.11` | **FAIL** - ACL 110 unchanged | |
| F15 | Firewall SSH allowed | PC-ADM1 | `ssh -l netadmin 172.16.0.2` | Password prompt, then `ASA-EDGE>` | |
| F16 | Firewall SSH refused | PC-CSE1 | `ssh -l netadmin 172.16.0.2` | Refused or timed out | |
| F17 | Live connections | ASA-EDGE | `show conn` | Sessions from F5/F6 listed | |
| F18 | DMZ port security | SW-DMZ | `show port-security` | Fa0/1 secured, max 1 | |

### Seeing NAT in Simulation mode

1. Switch to **Simulation**, Edit Filters, tick only **ICMP**.
2. From PC-CSE1: `ping 198.51.100.20`.
3. Step the packet with Capture/Forward. Click it on the link **before**
   the ASA, then on the link **after** the ASA, and open the PDU details.
4. The source IP changes from `192.168.20.x` to `203.0.113.2`. That one
   field changing is NAT made visible.

---

## 4. Demo script (about 4 minutes)

Add this after Step 8 (ACLs) of the Demonstration Guide.

> **SAY:** So far every rule has been *inside* the campus. A real campus
> also needs a boundary with the Internet. This firewall, ASA-EDGE,
> splits the world into three zones: the campus is trusted at level 100,
> the DMZ at 50, and the Internet at 0.

1. PC-CSE1: `ping 198.51.100.20`, then the browser at
   `http://www.example.com`.
   > **SAY:** The campus reaches the Internet. The ISP has no route back
   > to 192.168, so this only works because the firewall translated my
   > address.
2. ASA-EDGE: `show xlate`.
   > **SAY:** Here is the translation: my private address mapped to the
   > single public address 203.0.113.2. That is PAT, many hosts sharing
   > one address by port number.
3. EXT-PC browser at `http://www.university.edu`, then `ping 203.0.113.10`.
   > **SAY:** From the Internet, the public website loads, but the ping to
   > the same server is dropped. The firewall publishes HTTP and nothing
   > else.
4. EXT-PC: `ping 192.168.20.11`.
   > **SAY:** And the campus itself is simply not reachable from outside.
5. DMZ-WEB: `ping 192.168.20.11`.
   > **SAY:** This is the most important one. Suppose an attacker takes
   > over the public web server. It is in the DMZ at level 50, and a lower
   > level can never start a connection to a higher one. The compromised
   > server cannot reach a single campus machine.
6. ASA-EDGE: `show access-list OUTSIDE-IN`. Point at the hit count.

---

## 5. Viva questions

| Question | Short answer |
|----------|--------------|
| Why a DMZ instead of putting the web server in VLAN 80? | Anything reachable from the Internet can be compromised. In VLAN 80 it would sit next to the DNS and application servers. In the DMZ the firewall stops it from reaching anything inside. |
| What is the difference between PAT and static NAT here? | PAT maps all campus hosts to one address (203.0.113.2) using port numbers, outbound only. Static NAT permanently maps one server to one public address (203.0.113.10) so the Internet can start connections to it. |
| Why does the outside ACL name 172.16.100.10 and not 203.0.113.10? | Since ASA 8.3, access lists match the real (untranslated) address. |
| Why `inspect icmp`? | The ASA tracks TCP and UDP sessions by default, but not ICMP. Without it, echo replies coming back to campus pings would be dropped. |
| Why can the campus reach the DMZ without any rule? | Traffic from a higher security level to a lower one is permitted by default, and the replies are allowed back by stateful inspection. |
| Why two different DNS answers for www.university.edu? | Split DNS. Inside users get the real DMZ address and go there directly; outside users get the public NAT address. Without it, campus users would have to hairpin out through the outside interface. |
| Why does SW-DMZ have no management IP? | A management address in the DMZ could be attacked from a compromised DMZ server. Console-only removes that path entirely. |
| Why a /30 between R1 and the ASA? | It is a point-to-point link with exactly two hosts; a /30 provides exactly two usable addresses. |
| Does the ASA replace the ACLs on R1? | No. R1's ACLs control traffic between departments (east-west); the ASA controls traffic to and from the Internet (north-south). That is defence in depth. |

---

## 6. Troubleshooting

| Symptom | Check |
|---------|-------|
| R1 `ping 172.16.0.2` fails | Link 1 red? Swap cable type. On ASA `show nameif`: is Gi1/2 named inside? |
| Campus ping to Internet times out, but ASA can ping 198.51.100.20 | `show nat` (CAMPUS-LAN object present?) and `show service-policy` (inspect icmp present?) |
| `show xlate` empty | Traffic never reached the ASA: check R1's `S*` default route |
| EXT-PC cannot load www.university.edu | Try `http://203.0.113.10`. If the IP works, fix the EXT-SRV DNS record. If not, `show access-list OUTSIDE-IN` and `show nat` (DMZ-WEB static) |
| Campus PCs get 192.168.1.x addresses | The ASA's factory DHCP server is still on: `no dhcpd enable inside` |
| DMZ-WEB unreachable from campus | DMZ-WEB gateway must be 172.16.100.1. SW-DMZ Gi0/1 and Fa0/1 in VLAN 100 |
| SSH to ASA refused from Admin | `show ssh` should list 192.168.10.0 255.255.255.0 inside; regenerate the key with `crypto key generate rsa modulus 1024` |
