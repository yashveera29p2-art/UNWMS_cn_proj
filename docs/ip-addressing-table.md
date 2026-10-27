# Complete IP Addressing Table

Project **R075458** — every interface in the campus, with zero duplicate
addresses. DHCP addresses are shown as the value the client **will**
receive, because `.1`–`.10` are excluded on every pool and Packet Tracer
leases in ascending order from `.11` as devices are powered on.

> If you power the PCs up in a different order, the last octet may differ.
> The subnet, mask, gateway and VLAN are guaranteed; only the host octet
> can shift. Run `show ip dhcp binding` on `R1-CORE` for the truth.

---

## 1. Infrastructure — Router

| Device | Interface | IP address | Subnet mask | Default gateway | VLAN | Method |
|--------|-----------|------------|-------------|-----------------|------|--------|
| R1-CORE | Gi0/0 | *(no IP — trunk)* | — | — | trunk | — |
| R1-CORE | Gi0/0.10 | 192.168.10.1 | 255.255.255.0 | — | 10 | Static |
| R1-CORE | Gi0/0.20 | 192.168.20.1 | 255.255.255.0 | — | 20 | Static |
| R1-CORE | Gi0/0.30 | 192.168.30.1 | 255.255.255.0 | — | 30 | Static |
| R1-CORE | Gi0/0.40 | 192.168.40.1 | 255.255.255.0 | — | 40 | Static |
| R1-CORE | Gi0/0.50 | 192.168.50.1 | 255.255.255.0 | — | 50 | Static |
| R1-CORE | Gi0/0.60 | 192.168.60.1 | 255.255.255.0 | — | 60 | Static |
| R1-CORE | Gi0/0.70 | 192.168.70.1 | 255.255.255.0 | — | 70 | Static |
| R1-CORE | Gi0/0.80 | 192.168.80.1 | 255.255.255.0 | — | 80 | Static |
| R1-CORE | Gi0/0.99 | 192.168.99.1 | 255.255.255.0 | — | 99 (native) | Static |
| R1-CORE | Gi0/1 | *(shutdown)* | — | — | — | — |
| R1-CORE | Gi0/2 | *(shutdown)* | — | — | — | — |

## 2. Infrastructure — Switch management SVIs (VLAN 99)

| Device | Interface | IP address | Subnet mask | Default gateway | VLAN | Method |
|--------|-----------|------------|-------------|-----------------|------|--------|
| SW-DIST  | Vlan99 | 192.168.99.2  | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-ADMIN | Vlan99 | 192.168.99.11 | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-CSE   | Vlan99 | 192.168.99.12 | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-ECE   | Vlan99 | 192.168.99.13 | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-MECH  | Vlan99 | 192.168.99.14 | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-LIB   | Vlan99 | 192.168.99.15 | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-FAC   | Vlan99 | 192.168.99.16 | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-LAB   | Vlan99 | 192.168.99.17 | 255.255.255.0 | 192.168.99.1 | 99 | Static |
| SW-SRV   | Vlan99 | 192.168.99.18 | 255.255.255.0 | 192.168.99.1 | 99 | Static |

## 3. Servers — VLAN 80, all STATIC

| Device | Interface | IP address | Subnet mask | Default gateway | DNS server | VLAN | Switch port |
|--------|-----------|------------|-------------|-----------------|-----------|------|-------------|
| SRV-DNS | FastEthernet0 | 192.168.80.10 | 255.255.255.0 | 192.168.80.1 | 192.168.80.10 | 80 | SW-SRV Fa0/1 |
| SRV-WEB | FastEthernet0 | 192.168.80.20 | 255.255.255.0 | 192.168.80.1 | 192.168.80.10 | 80 | SW-SRV Fa0/2 |
| SRV-APP | FastEthernet0 | 192.168.80.30 | 255.255.255.0 | 192.168.80.1 | 192.168.80.10 | 80 | SW-SRV Fa0/3 |

## 4. End devices — VLAN 10 to 70, all DHCP

### VLAN 10 — Administration (switch `SW-ADMIN`)

| Device | Interface | IP address (expected) | Subnet mask | Default gateway | DNS server | VLAN | Switch port | Method |
|--------|-----------|----------------------|-------------|-----------------|-----------|------|-------------|--------|
| PC-ADM1 | FastEthernet0 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 | 192.168.80.10 | 10 | SW-ADMIN Fa0/1 | DHCP |
| PC-ADM2 | FastEthernet0 | 192.168.10.12 | 255.255.255.0 | 192.168.10.1 | 192.168.80.10 | 10 | SW-ADMIN Fa0/2 | DHCP |
| PC-ADM3 | FastEthernet0 | 192.168.10.13 | 255.255.255.0 | 192.168.10.1 | 192.168.80.10 | 10 | SW-ADMIN Fa0/3 | DHCP |
| PC-ADM4 | FastEthernet0 | 192.168.10.14 | 255.255.255.0 | 192.168.10.1 | 192.168.80.10 | 10 | SW-ADMIN Fa0/4 | DHCP |

### VLAN 20 — Computer Science (switch `SW-CSE`)

| Device | Interface | IP address (expected) | Subnet mask | Default gateway | DNS server | VLAN | Switch port | Method |
|--------|-----------|----------------------|-------------|-----------------|-----------|------|-------------|--------|
| PC-CSE1 | FastEthernet0 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 | 192.168.80.10 | 20 | SW-CSE Fa0/1 | DHCP |
| PC-CSE2 | FastEthernet0 | 192.168.20.12 | 255.255.255.0 | 192.168.20.1 | 192.168.80.10 | 20 | SW-CSE Fa0/2 | DHCP |
| PC-CSE3 | FastEthernet0 | 192.168.20.13 | 255.255.255.0 | 192.168.20.1 | 192.168.80.10 | 20 | SW-CSE Fa0/3 | DHCP |
| PC-CSE4 | FastEthernet0 | 192.168.20.14 | 255.255.255.0 | 192.168.20.1 | 192.168.80.10 | 20 | SW-CSE Fa0/4 | DHCP |
| PC-CSE5 | FastEthernet0 | 192.168.20.15 | 255.255.255.0 | 192.168.20.1 | 192.168.80.10 | 20 | SW-CSE Fa0/5 | DHCP |
| PC-CSE6 | FastEthernet0 | 192.168.20.16 | 255.255.255.0 | 192.168.20.1 | 192.168.80.10 | 20 | SW-CSE Fa0/6 | DHCP |

### VLAN 30 — Electronics (switch `SW-ECE`)

| Device | Interface | IP address (expected) | Subnet mask | Default gateway | DNS server | VLAN | Switch port | Method |
|--------|-----------|----------------------|-------------|-----------------|-----------|------|-------------|--------|
| PC-ECE1 | FastEthernet0 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 | 192.168.80.10 | 30 | SW-ECE Fa0/1 | DHCP |
| PC-ECE2 | FastEthernet0 | 192.168.30.12 | 255.255.255.0 | 192.168.30.1 | 192.168.80.10 | 30 | SW-ECE Fa0/2 | DHCP |
| PC-ECE3 | FastEthernet0 | 192.168.30.13 | 255.255.255.0 | 192.168.30.1 | 192.168.80.10 | 30 | SW-ECE Fa0/3 | DHCP |
| PC-ECE4 | FastEthernet0 | 192.168.30.14 | 255.255.255.0 | 192.168.30.1 | 192.168.80.10 | 30 | SW-ECE Fa0/4 | DHCP |
| PC-ECE5 | FastEthernet0 | 192.168.30.15 | 255.255.255.0 | 192.168.30.1 | 192.168.80.10 | 30 | SW-ECE Fa0/5 | DHCP |
| PC-ECE6 | FastEthernet0 | 192.168.30.16 | 255.255.255.0 | 192.168.30.1 | 192.168.80.10 | 30 | SW-ECE Fa0/6 | DHCP |

### VLAN 40 — Mechanical (switch `SW-MECH`)

| Device | Interface | IP address (expected) | Subnet mask | Default gateway | DNS server | VLAN | Switch port | Method |
|--------|-----------|----------------------|-------------|-----------------|-----------|------|-------------|--------|
| PC-MECH1 | FastEthernet0 | 192.168.40.11 | 255.255.255.0 | 192.168.40.1 | 192.168.80.10 | 40 | SW-MECH Fa0/1 | DHCP |
| PC-MECH2 | FastEthernet0 | 192.168.40.12 | 255.255.255.0 | 192.168.40.1 | 192.168.80.10 | 40 | SW-MECH Fa0/2 | DHCP |
| PC-MECH3 | FastEthernet0 | 192.168.40.13 | 255.255.255.0 | 192.168.40.1 | 192.168.80.10 | 40 | SW-MECH Fa0/3 | DHCP |
| PC-MECH4 | FastEthernet0 | 192.168.40.14 | 255.255.255.0 | 192.168.40.1 | 192.168.80.10 | 40 | SW-MECH Fa0/4 | DHCP |

### VLAN 50 — Library (switch `SW-LIB`)

| Device | Interface | IP address (expected) | Subnet mask | Default gateway | DNS server | VLAN | Switch port | Method |
|--------|-----------|----------------------|-------------|-----------------|-----------|------|-------------|--------|
| PC-LIB1 | FastEthernet0 | 192.168.50.11 | 255.255.255.0 | 192.168.50.1 | 192.168.80.10 | 50 | SW-LIB Fa0/1 | DHCP |
| PC-LIB2 | FastEthernet0 | 192.168.50.12 | 255.255.255.0 | 192.168.50.1 | 192.168.80.10 | 50 | SW-LIB Fa0/2 | DHCP |
| PC-LIB3 | FastEthernet0 | 192.168.50.13 | 255.255.255.0 | 192.168.50.1 | 192.168.80.10 | 50 | SW-LIB Fa0/3 | DHCP |
| PC-LIB4 | FastEthernet0 | 192.168.50.14 | 255.255.255.0 | 192.168.50.1 | 192.168.80.10 | 50 | SW-LIB Fa0/4 | DHCP |

### VLAN 60 — Faculty (switch `SW-FAC`)

| Device | Interface | IP address (expected) | Subnet mask | Default gateway | DNS server | VLAN | Switch port | Method |
|--------|-----------|----------------------|-------------|-----------------|-----------|------|-------------|--------|
| PC-FAC1 | FastEthernet0 | 192.168.60.11 | 255.255.255.0 | 192.168.60.1 | 192.168.80.10 | 60 | SW-FAC Fa0/1 | DHCP |
| PC-FAC2 | FastEthernet0 | 192.168.60.12 | 255.255.255.0 | 192.168.60.1 | 192.168.80.10 | 60 | SW-FAC Fa0/2 | DHCP |
| PC-FAC3 | FastEthernet0 | 192.168.60.13 | 255.255.255.0 | 192.168.60.1 | 192.168.80.10 | 60 | SW-FAC Fa0/3 | DHCP |
| PC-FAC4 | FastEthernet0 | 192.168.60.14 | 255.255.255.0 | 192.168.60.1 | 192.168.80.10 | 60 | SW-FAC Fa0/4 | DHCP |

### VLAN 70 — Student Labs (switch `SW-LAB`)

| Device | Interface | IP address (expected) | Subnet mask | Default gateway | DNS server | VLAN | Switch port | Method |
|--------|-----------|----------------------|-------------|-----------------|-----------|------|-------------|--------|
| PC-LAB1 | FastEthernet0 | 192.168.70.11 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/1 | DHCP |
| PC-LAB2 | FastEthernet0 | 192.168.70.12 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/2 | DHCP |
| PC-LAB3 | FastEthernet0 | 192.168.70.13 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/3 | DHCP |
| PC-LAB4 | FastEthernet0 | 192.168.70.14 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/4 | DHCP |
| PC-LAB5 | FastEthernet0 | 192.168.70.15 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/5 | DHCP |
| PC-LAB6 | FastEthernet0 | 192.168.70.16 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/6 | DHCP |
| PC-LAB7 | FastEthernet0 | 192.168.70.17 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/7 | DHCP |
| PC-LAB8 | FastEthernet0 | 192.168.70.18 | 255.255.255.0 | 192.168.70.1 | 192.168.80.10 | 70 | SW-LAB Fa0/8 | DHCP |

---

## 5. Address audit — proof there are no duplicates

| Subnet | .1 | .2 | .10 | .11–.18 | .20 | .30 | Rest |
|--------|----|----|-----|---------|-----|-----|------|
| 192.168.10.0/24 | R1 Gi0/0.10 | excluded | excluded | PC-ADM1–4 use .11–.14 | free | free | DHCP |
| 192.168.20.0/24 | R1 Gi0/0.20 | excluded | excluded | PC-CSE1–6 use .11–.16 | free | free | DHCP |
| 192.168.30.0/24 | R1 Gi0/0.30 | excluded | excluded | PC-ECE1–6 use .11–.16 | free | free | DHCP |
| 192.168.40.0/24 | R1 Gi0/0.40 | excluded | excluded | PC-MECH1–4 use .11–.14 | free | free | DHCP |
| 192.168.50.0/24 | R1 Gi0/0.50 | excluded | excluded | PC-LIB1–4 use .11–.14 | free | free | DHCP |
| 192.168.60.0/24 | R1 Gi0/0.60 | excluded | excluded | PC-FAC1–4 use .11–.14 | free | free | DHCP |
| 192.168.70.0/24 | R1 Gi0/0.70 | excluded | excluded | PC-LAB1–8 use .11–.18 | free | free | DHCP |
| 192.168.80.0/24 | R1 Gi0/0.80 | free | **SRV-DNS** | free | **SRV-WEB** | **SRV-APP** | unused |
| 192.168.99.0/24 | R1 Gi0/0.99 | **SW-DIST** | free | **8 access switches** | free | free | unused |

Total addresses consumed: **9 router + 9 switch SVI + 36 PC + 3 server = 57**.
Every one of them is unique.

## 6. DNS records served by SRV-DNS (`192.168.80.10`)

| Name | Type | Address | What it is |
|------|------|---------|-----------|
| `www.university.local`    | A | 192.168.80.20 | Main university web portal (SRV-WEB) |
| `dns.university.local`    | A | 192.168.80.10 | The DNS server itself |
| `portal.university.local` | A | 192.168.80.30 | Student/staff application portal (SRV-APP) |
| `university.local`        | A | 192.168.80.20 | Bare domain, points at the web server |
| `ftp.university.local`    | A | 192.168.80.30 | File service on SRV-APP |

---

## 7. Extension - edge firewall and DMZ

Full detail in [`edge-firewall-dmz.md`](edge-firewall-dmz.md).

| Device | Interface | Address | Network | Zone |
|--------|-----------|---------|---------|------|
| R1-CORE | Gi0/1 | 172.16.0.1/30 | 172.16.0.0/30 | campus transit |
| ASA-EDGE | Gi1/2 (inside) | 172.16.0.2/30 | 172.16.0.0/30 | inside, level 100 |
| ASA-EDGE | Gi1/3 (dmz) | 172.16.100.1/24 | 172.16.100.0/24 | dmz, level 50 |
| ASA-EDGE | Gi1/1 (outside) | 203.0.113.2/28 | 203.0.113.0/28 | outside, level 0 |
| DMZ-WEB | Fa0 | 172.16.100.10/24, gw .1 | 172.16.100.0/24 | dmz |
| DMZ-WEB (published) | static NAT | 203.0.113.10 | 203.0.113.0/28 | outside |
| ISP-RTR | Gi0/0 | 203.0.113.1/28 | 203.0.113.0/28 | Internet |
| ISP-RTR | Gi0/1 | 198.51.100.1/24 | 198.51.100.0/24 | Internet |
| EXT-PC | Fa0 | 198.51.100.10/24, gw .1, DNS .20 | 198.51.100.0/24 | Internet |
| EXT-SRV | Fa0 | 198.51.100.20/24, gw .1, DNS .20 | 198.51.100.0/24 | Internet |

New DNS records:

| Server | Name | Address | Why |
|--------|------|---------|-----|
| SRV-DNS (campus) | www.university.edu | 172.16.100.10 | split DNS: campus goes to the real DMZ address |
| SRV-DNS (campus) | www.example.com | 198.51.100.20 | lets campus PCs browse the Internet site by name |
| EXT-SRV (Internet) | www.university.edu | 203.0.113.10 | Internet users get the public NAT address |
| EXT-SRV (Internet) | www.example.com | 198.51.100.20 | |
