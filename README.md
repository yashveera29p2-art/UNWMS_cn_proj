# University Network with Multiple Subnets — R075458

A full campus network built in **Cisco Packet Tracer**: eight departments,
eight VLANs, eight subnets, router-on-a-stick inter-VLAN routing,
router-based DHCP, an internal DNS zone, a web portal, an FTP/application
server, and a layered security posture (SSH, port security, ACLs).

**Extension - edge firewall and DMZ.** A Cisco ASA 5506-X connects the
campus to a simulated Internet with three security zones (inside / dmz /
outside), PAT for the campus, a public web server in a DMZ published by
static NAT, and an outside ACL that admits HTTP only. See
[`docs/edge-firewall-dmz.md`](docs/edge-firewall-dmz.md).

Everything needed to rebuild this network **by hand** is in this
repository, so the project survives even if the `.pkt` file will not open
on another machine.

---

## Requirements

| Item | Requirement |
|------|-------------|
| Cisco Packet Tracer | **8.2 or newer**. The build machine for this project has **Packet Tracer 9.0.1** installed (`C:\Program Files\Cisco Packet Tracer 9.0.1`) |
| Cisco account | Required — PT 8.0+ will not open any file until you sign in with a NetAcad / Skills for All account |
| OS | Windows, macOS or Linux — the `.pkt` is portable across all three |
| Disk | ~5 MB for the repository |

> **This machine saves in PT 9.0 format.** A `.pkt` written by PT 9.0.1
> will **not** open on a laptop running 8.2 or older. Either install
> 9.0.1 on the laptop too, or use `File → Save As…` and pick an older
> entry in the *Save as type* dropdown before committing.

> **Version warning.** A `.pkt` saved in a newer Packet Tracer **cannot**
> be opened by an older one. Compatibility runs one way only. Use
> `File → Save As…` and pick an older version from the *Save as type*
> dropdown before sharing. Full detail in
> [`docs/portability-notes.md`](docs/portability-notes.md).

---

## Repository layout

```
R075458/
├── README.md                        <- you are here
├── .gitignore
├── packet-tracer/
│   ├── university-network.pkt       <- the simulation, built on PT 9.0.1
│   └── README.md                    <- what belongs in this folder
├── configs/
│   ├── R1-core-router.txt           <- paste-ready CLI, core router
│   ├── SW-DIST.txt                  <- distribution switch
│   ├── SW-ADMIN.txt                 <- VLAN 10
│   ├── SW-CSE.txt                   <- VLAN 20
│   ├── SW-ECE.txt                   <- VLAN 30
│   ├── SW-MECH.txt                  <- VLAN 40
│   ├── SW-LIB.txt                   <- VLAN 50
│   ├── SW-FAC.txt                   <- VLAN 60
│   ├── SW-LAB.txt                   <- VLAN 70
│   ├── SW-SRV.txt                   <- VLAN 80
│   ├── ASA-EDGE.txt                 <- edge firewall (extension)
│   ├── SW-DMZ.txt                   <- DMZ switch (extension)
│   ├── ISP-RTR.txt                  <- simulated ISP (extension)
│   └── ISP-SW.txt                   <- simulated Internet switch
├── docs/
│   ├── topology-diagram.txt         <- ASCII topology, every device and link
│   ├── device-list.md               <- device list + VLAN table
│   ├── ip-addressing-table.md       <- every interface, zero duplicates
│   ├── cable-connection-list.md     <- all 48 cables, port to port
│   ├── pc-server-setup.md           <- GUI steps for PCs and servers
│   ├── build-checklist.md           <- blank file to working demo
│   ├── testing-results.md           <- 18 tests with expected output
│   ├── troubleshooting.md           <- 12 realistic failures and fixes
│   ├── portability-notes.md         <- PT versions and simulator limits
│   └── edge-firewall-dmz.md         <- extension: build, tests, demo, viva
└── web/
    ├── index.html                   <- paste into SRV-WEB  index.html
    ├── portal.html                  <- paste into SRV-APP  index.html
    ├── public.html                  <- paste into DMZ-WEB  index.html
    └── internet.html                <- paste into EXT-SRV  index.html
```

---

## VLAN summary

| VLAN | Name | Department | Network | Gateway | Hosts | Addressing |
|------|------|------------|---------|---------|-------|-----------|
| 10 | ADMIN | Administration | 192.168.10.0/24 | 192.168.10.1 | 4 PCs | DHCP |
| 20 | CSE | Computer Science | 192.168.20.0/24 | 192.168.20.1 | 6 PCs | DHCP |
| 30 | ECE | Electronics | 192.168.30.0/24 | 192.168.30.1 | 6 PCs | DHCP |
| 40 | MECH | Mechanical | 192.168.40.0/24 | 192.168.40.1 | 4 PCs | DHCP |
| 50 | LIBRARY | Library | 192.168.50.0/24 | 192.168.50.1 | 4 PCs | DHCP |
| 60 | FACULTY | Faculty | 192.168.60.0/24 | 192.168.60.1 | 4 PCs | DHCP |
| 70 | LAB | Student Labs | 192.168.70.0/24 | 192.168.70.1 | 8 PCs | DHCP |
| 80 | SERVERS | Server Room | 192.168.80.0/24 | 192.168.80.1 | 3 servers | Static |
| 99 | MGMT | Switch management (native VLAN) | 192.168.99.0/24 | 192.168.99.1 | 9 SVIs | Static |
| 999 | PARKING-UNUSED | Disabled ports | — | — | — | — |

**Servers:** DNS `192.168.80.10` · Web `192.168.80.20` ·
Application/File `192.168.80.30`
**Domain:** `university.local` — `www`, `dns`, `portal`, `ftp` A records

---

## How to open the simulation

1. Launch Packet Tracer and sign in with your Cisco account.
2. `File → Open` → `packet-tracer/university-network.pkt`.
3. Wait for the link lights to go green (spanning tree takes ~30 s from
   cold).
4. Verify quickly: click `R1-CORE` → CLI → `enable` →
   password `Univ@Core#2025` → `show ip interface brief`.
   All nine subinterfaces should be `up/up`.

If the file refuses to open, your Packet Tracer is older than the one
that saved it. Either upgrade PT, or rebuild from `configs/` — see below.

---

## How to rebuild from `configs/`

You need no `.pkt` at all for this. Roughly 60–90 minutes.

1. Follow [`docs/build-checklist.md`](docs/build-checklist.md) **Phase 1**
   to place the 49 devices with the exact names used in this repo.
2. Follow [`docs/cable-connection-list.md`](docs/cable-connection-list.md)
   to run all 48 cables.
3. Open each file in `configs/`, select all, and paste it into that
   device's **CLI** tab. Answer **1024** at the RSA modulus prompt.
4. Do the GUI work in
   [`docs/pc-server-setup.md`](docs/pc-server-setup.md) — static IPs and
   services on the three servers, DHCP on the 36 PCs.
5. Paste `web/index.html` into `SRV-WEB` and `web/portal.html` into
   `SRV-APP`.
6. Work through [`docs/testing-results.md`](docs/testing-results.md).
7. `File → Save As…` → `packet-tracer/university-network.pkt`.

---

## Credentials used in the lab

These are deliberately visible — this is a teaching lab, not production.
Change every one of them before reusing any of this on real hardware.

| Where | Username | Password |
|-------|----------|----------|
| `R1-CORE` enable | — | `Univ@Core#2025` |
| `SW-DIST` enable | — | `Univ@Dist#2025` |
| Access switches enable | — | `Univ@<Dept>#2025` (e.g. `Univ@Cse#2025`) |
| Console (all devices) | — | `Cons@2025` |
| VTY fallback | — | `Vty@2025` |
| SSH (all devices) | `netadmin` | `Adm1n@Univ#2025` |
| FTP on `SRV-APP` | `student` | `Student@2025` |
| `ASA-EDGE` enable | — | `Univ@Asa#2025` |
| `SW-DMZ` enable | — | `Univ@Dmz#2025` |
| `ISP-RTR` enable (simulated ISP) | — | `Isp@2025` |

SSH is reachable **only from `192.168.10.0/24`** — ACL 10 is applied to
the VTY lines with `access-class 10 in`.

---

## Security summary

| Control | Where | What it does |
|---------|-------|--------------|
| `enable secret` | every device | MD5-hashed privileged-mode password |
| `service password-encryption` | every device | obscures the remaining plaintext passwords |
| Console password + `exec-timeout 5 0` | every device | physical-access control, idle logout |
| SSH v2, `transport input ssh` | every device | Telnet is disabled outright |
| `access-class 10 in` on VTY | every device | management only from the Admin subnet |
| ACL 110 in on `Gi0/0.70` | R1-CORE | Student labs cannot reach Admin or Faculty |
| ACL 120 out on `Gi0/0.80` | R1-CORE | Server VLAN accepts only DNS/HTTP/HTTPS/FTP/ICMP |
| Port security, max 1, sticky, restrict | all access ports | one MAC per port, violations counted and dropped |
| `spanning-tree bpduguard enable` | all access ports | a rogue switch on a user port is err-disabled |
| Unused ports in VLAN 999 + `shutdown` | all switches | no unattended live port anywhere |
| Native VLAN 99, not VLAN 1 | all trunks | removes the classic VLAN-hopping exposure |
| Security levels inside 100 / dmz 50 / outside 0 | ASA-EDGE | nothing from the Internet or the DMZ can open a connection into the campus |
| PAT to `203.0.113.2` | ASA-EDGE | private campus addresses never appear on the Internet |
| Static NAT + ACL `OUTSIDE-IN` (HTTP only) | ASA-EDGE | the Internet reaches the DMZ web server on port 80 and nothing else |
| Stateful inspection incl. `inspect icmp` | ASA-EDGE | only replies to campus-initiated traffic come back in |
| SSH only from `192.168.10.0/24` on inside | ASA-EDGE | firewall managed from Admin only, never from dmz/outside |
| No management IP, VTY disabled | SW-DMZ | a compromised DMZ server cannot reach the switch |

---

## Git workflow

### On the build PC — create and push

```bash
cd R075458
git init
git add .
git commit -m "University network with multiple subnets - configs, docs and web portal"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

Create the empty repository on GitHub first (no README, no `.gitignore` —
this repo already has both), then run the commands above.

### After you build the simulation

```bash
git add packet-tracer/university-network.pkt
git commit -m "Add Packet Tracer simulation file (saved from PT 9.0)"
git push
```

Put the **PT version** in the commit message. Future-you on a different
laptop will need it.

### On the laptop — clone and work

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

Open `packet-tracer/university-network.pkt` in Packet Tracer.

### Sending changes back

```bash
git add -A
git commit -m "Describe what changed - e.g. tightened ACL 120"
git push
```

### Pulling changes on the other machine

```bash
git pull
```

> **The `.pkt` is a binary blob to git.** It cannot be merged. If you edit
> the simulation on both machines you will get a conflict that git cannot
> resolve, and one side's work is lost. Always `git pull` before you
> open Packet Tracer, and `git push` as soon as you close it. If a
> conflict does happen, pick one file with
> `git checkout --theirs packet-tracer/university-network.pkt` (or
> `--ours`) and rebuild the lost changes by hand from `configs/`.

---

## Documentation index

| Document | Read it when |
|----------|--------------|
| [`docs/build-checklist.md`](docs/build-checklist.md) | Building from scratch |
| [`docs/topology-diagram.txt`](docs/topology-diagram.txt) | You need the picture |
| [`docs/device-list.md`](docs/device-list.md) | Buying/placing devices, VLAN reference |
| [`docs/ip-addressing-table.md`](docs/ip-addressing-table.md) | Any addressing question |
| [`docs/cable-connection-list.md`](docs/cable-connection-list.md) | Cabling, port by port |
| [`docs/pc-server-setup.md`](docs/pc-server-setup.md) | Configuring PCs and servers in the GUI |
| [`docs/testing-results.md`](docs/testing-results.md) | Proving it works / demo day |
| [`docs/troubleshooting.md`](docs/troubleshooting.md) | Something is broken |
| [`docs/portability-notes.md`](docs/portability-notes.md) | Moving between machines, PT limits |
| [`docs/edge-firewall-dmz.md`](docs/edge-firewall-dmz.md) | Building, testing and demonstrating the firewall/DMZ extension |

---

## Design decisions worth defending

**Router-on-a-Stick over a Layer-3 switch.** One 2911 with nine 802.1Q
subinterfaces is simpler to build, simpler to read, and makes the VLAN
tagging visible in Simulation mode. The trade-off is that every
inter-VLAN packet crosses `Gi0/0` twice — acceptable at 49 devices,
wrong at 4,900. The SVI alternative is written out in
`docs/portability-notes.md` §5.

**VLAN 99 was added to the brief's eight VLANs.** Trunks need a native
VLAN, and leaving it as VLAN 1 is a VLAN-hopping exposure. The nine
switches also need a management address so SSH can reach them. VLAN 99
solves both. The eight departmental VLANs and subnets are exactly as
specified.

**DHCP lives on the router, not on a server.** One authority, one place
to look, and it survives a server being switched off. It also means
`show ip dhcp binding` on `R1-CORE` is a single source of truth for all
36 leases.

**Servers are static.** Their addresses are referenced by DNS records,
by the ACLs and by the `dns-server` option in seven DHCP pools. A server
that changed address on reboot would break all three.
