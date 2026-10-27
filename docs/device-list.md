# Device List and VLAN Table

Project **R075458** — University Network with Multiple Subnets
Built on **Cisco Packet Tracer 8.2 or newer**.

---

## 1. Device List

Every device below exists in a stock Packet Tracer install. No custom
modules, no add-on images.

| # | Device name | Model (PT category) | Qty | Connects to |
|---|-------------|---------------------|-----|-------------|
| 1 | `R1-CORE`  | Router → **2911** | 1 | `SW-DIST` Gi0/1 via Gi0/0 (802.1Q trunk) |
| 2 | `SW-DIST`  | Switch → **2960-24TT** | 1 | `R1-CORE` Gi0/0 (uplink trunk) + all 8 access switches on Fa0/1–Fa0/8 |
| 3 | `SW-ADMIN` | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/1 via Fa0/24; 4 Administration PCs on Fa0/1–4 |
| 4 | `SW-CSE`   | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/2 via Fa0/24; 6 CSE PCs on Fa0/1–6 |
| 5 | `SW-ECE`   | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/3 via Fa0/24; 6 ECE PCs on Fa0/1–6 |
| 6 | `SW-MECH`  | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/4 via Fa0/24; 4 Mechanical PCs on Fa0/1–4 |
| 7 | `SW-LIB`   | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/5 via Fa0/24; 4 Library PCs on Fa0/1–4 |
| 8 | `SW-FAC`   | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/6 via Fa0/24; 4 Faculty PCs on Fa0/1–4 |
| 9 | `SW-LAB`   | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/7 via Fa0/24; 8 Student-lab PCs on Fa0/1–8 |
| 10 | `SW-SRV`  | Switch → **2960-24TT** | 1 | `SW-DIST` Fa0/8 via Fa0/24; 3 servers on Fa0/1–3 |
| 11 | `PC-ADM1` … `PC-ADM4` | End Device → **PC** | 4 | `SW-ADMIN` Fa0/1–Fa0/4 |
| 12 | `PC-CSE1` … `PC-CSE6` | End Device → **PC** | 6 | `SW-CSE` Fa0/1–Fa0/6 |
| 13 | `PC-ECE1` … `PC-ECE6` | End Device → **PC** | 6 | `SW-ECE` Fa0/1–Fa0/6 |
| 14 | `PC-MECH1` … `PC-MECH4` | End Device → **PC** | 4 | `SW-MECH` Fa0/1–Fa0/4 |
| 15 | `PC-LIB1` … `PC-LIB4` | End Device → **PC** | 4 | `SW-LIB` Fa0/1–Fa0/4 |
| 16 | `PC-FAC1` … `PC-FAC4` | End Device → **PC** | 4 | `SW-FAC` Fa0/1–Fa0/4 |
| 17 | `PC-LAB1` … `PC-LAB8` | End Device → **PC** | 8 | `SW-LAB` Fa0/1–Fa0/8 |
| 18 | `SRV-DNS`  | End Device → **Server** | 1 | `SW-SRV` Fa0/1 |
| 19 | `SRV-WEB`  | End Device → **Server** | 1 | `SW-SRV` Fa0/2 |
| 20 | `SRV-APP`  | End Device → **Server** | 1 | `SW-SRV` Fa0/3 |

**Totals** — 1 router, 9 switches, 36 PCs, 3 servers = **49 devices**, **48 cables**.

### Cables required

| Cable type | Quantity | Used for |
|------------|----------|----------|
| Copper Straight-Through | 40 | 1 router→switch, 36 PC→switch, 3 server→switch |
| Copper Cross-Over | 8 | switch→switch (SW-DIST to each access switch) |

---

## 2. VLAN Table

| VLAN | Name | Department | Network | Mask | Gateway | Usable range | Address method | Access switch |
|------|------|------------|---------|------|---------|--------------|----------------|---------------|
| 10 | `ADMIN`   | Administration  | 192.168.10.0 | /24 | 192.168.10.1 | .2 – .254 | DHCP `ADMIN_POOL` | SW-ADMIN |
| 20 | `CSE`     | Computer Science | 192.168.20.0 | /24 | 192.168.20.1 | .2 – .254 | DHCP `CSE_POOL` | SW-CSE |
| 30 | `ECE`     | Electronics     | 192.168.30.0 | /24 | 192.168.30.1 | .2 – .254 | DHCP `ECE_POOL` | SW-ECE |
| 40 | `MECH`    | Mechanical      | 192.168.40.0 | /24 | 192.168.40.1 | .2 – .254 | DHCP `MECH_POOL` | SW-MECH |
| 50 | `LIBRARY` | Library         | 192.168.50.0 | /24 | 192.168.50.1 | .2 – .254 | DHCP `LIBRARY_POOL` | SW-LIB |
| 60 | `FACULTY` | Faculty         | 192.168.60.0 | /24 | 192.168.60.1 | .2 – .254 | DHCP `FACULTY_POOL` | SW-FAC |
| 70 | `LAB`     | Student Labs    | 192.168.70.0 | /24 | 192.168.70.1 | .2 – .254 | DHCP `LAB_POOL` | SW-LAB |
| 80 | `SERVERS` | Server Room     | 192.168.80.0 | /24 | 192.168.80.1 | .2 – .254 | **STATIC only** | SW-SRV |
| 99 | `MGMT`    | Switch management (native VLAN) | 192.168.99.0 | /24 | 192.168.99.1 | .2 – .254 | **STATIC only** | all switches (SVI) |
| 999 | `PARKING-UNUSED` | Dead-end VLAN for disabled ports | — | — | — | — | none (ports shut) | all switches |

### The one addition to the specified addressing plan

> **VLAN 99 (`192.168.99.0/24`) and VLAN 999 were added.** VLAN 99 exists
> because the trunks need a native VLAN that is not VLAN 1 (leaving the
> native VLAN as 1 is a VLAN-hopping exposure) and because the nine
> switches need a management IP so SSH can reach them. VLAN 999 is an
> unrouted parking VLAN for shut-down ports. **The eight departmental
> VLANs and their subnets in the brief are unchanged.**

### DHCP address allocation inside each user VLAN

| Range | Purpose |
|-------|---------|
| `.1` | Router subinterface (default gateway) |
| `.2 – .10` | Reserved / excluded — printers, future static hosts |
| `.11 – .254` | DHCP pool, handed out to PCs in ascending order |

Server VLAN 80 has **no pool at all**; `.10`, `.20` and `.30` are typed in
by hand on each server.

---

## 3. Extension - edge firewall and DMZ

| Device | Model | Role |
|--------|-------|------|
| ASA-EDGE | Cisco ASA 5506-X | Edge firewall: zones, NAT/PAT, outside ACL, inspection |
| SW-DMZ | Cisco 2960-24TT | DMZ access switch, VLAN 100, no management IP |
| DMZ-WEB | Server-PT | Public web server (www.university.edu) |
| ISP-RTR | Cisco 2911 | Simulated ISP router |
| ISP-SW | Cisco 2960-24TT | Simulated Internet switch |
| EXT-SRV | Server-PT | Internet web + DNS server (www.example.com) |
| EXT-PC | PC-PT | Outside user / attacker |

**Totals with the extension** - 2 routers, 1 firewall, 11 switches,
37 PCs, 5 servers = **56 devices**, **55 cables**.

| VLAN | Name | Where | Network |
|------|------|-------|---------|
| 100 | DMZ | SW-DMZ only, not on any campus trunk | 172.16.100.0/24 |
