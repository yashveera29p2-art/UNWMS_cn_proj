# Build Checklist — Blank File to Working Demo

Project **R075458**. Follow top to bottom. Tick each box. Total time on a
first build: roughly 60–90 minutes.

---

## Phase 0 — Before you start

- [ ] Cisco Packet Tracer **8.2 or newer** is installed and you are
      signed in (PT refuses to open without a Cisco NetAcad / Skills for
      All login).
- [ ] This repository is cloned or downloaded.
- [ ] `File → New` in Packet Tracer.
- [ ] `File → Save As` immediately → `packet-tracer/university-network.pkt`.
      Save early and save often — PT has no autosave.

---

## Phase 1 — Place the devices (no cables yet)

- [ ] From **Network Devices → Routers**, drag one **2911** onto the
      canvas near the top. Rename it `R1-CORE`.
- [ ] From **Network Devices → Switches**, drag one **2960** below it.
      Rename it `SW-DIST`.
- [ ] Drag **eight** more 2960 switches in a row below `SW-DIST`.
      Rename them left to right:
      `SW-ADMIN`, `SW-CSE`, `SW-ECE`, `SW-MECH`, `SW-LIB`, `SW-FAC`,
      `SW-LAB`, `SW-SRV`.
- [ ] From **End Devices**, drag PCs under each access switch and rename:

  | Switch | PCs to place | Names |
  |--------|--------------|-------|
  | SW-ADMIN | 4 | `PC-ADM1` … `PC-ADM4` |
  | SW-CSE | 6 | `PC-CSE1` … `PC-CSE6` |
  | SW-ECE | 6 | `PC-ECE1` … `PC-ECE6` |
  | SW-MECH | 4 | `PC-MECH1` … `PC-MECH4` |
  | SW-LIB | 4 | `PC-LIB1` … `PC-LIB4` |
  | SW-FAC | 4 | `PC-FAC1` … `PC-FAC4` |
  | SW-LAB | 8 | `PC-LAB1` … `PC-LAB8` |

- [ ] From **End Devices**, drag **three Servers** under `SW-SRV` and
      rename them `SRV-DNS`, `SRV-WEB`, `SRV-APP`.
- [ ] Count: **49 devices** on the canvas.
- [ ] To rename: click the device, then click its label, or use
      **Config tab → Display Name**.
- [ ] Save.

> Renaming matters. Every config file, every test and every doc in this
> repository refers to these exact names.

---

## Phase 2 — Cable everything

Use `docs/cable-connection-list.md` and work through it row by row.

- [ ] **Cable 1** — Copper Straight-Through:
      `R1-CORE Gi0/0` → `SW-DIST Gi0/1`
- [ ] **Cables 2–9** — Copper Cross-Over:
      `SW-DIST Fa0/1..Fa0/8` → `Fa0/24` of each access switch, in the
      order ADMIN, CSE, ECE, MECH, LIB, FAC, LAB, SRV.
- [ ] **Cables 10–45** — Copper Straight-Through: every PC's
      `FastEthernet0` → its switch, filling `Fa0/1` upward with no gaps.
- [ ] **Cables 46–48** — Copper Straight-Through:
      `SRV-DNS`→`SW-SRV Fa0/1`, `SRV-WEB`→`Fa0/2`, `SRV-APP`→`Fa0/3`.
- [ ] Count: **48 cables**. Link lights will be amber (STP listening)
      then green.
- [ ] Save.

> Do the cabling **before** the switch configs. Port security uses sticky
> MAC learning and it is cleaner to let it learn the right device first
> time.

---

## Phase 3 — Configure the router

- [ ] Click `R1-CORE` → **CLI** tab → press Enter.
- [ ] Open `configs/R1-core-router.txt`.
- [ ] Copy from `enable` down to the end of **BLOCK 6** and paste.
- [ ] Copy **BLOCK 7** and paste. The paste will stop at
      `How many bits in the modulus [512]:` — type **1024**, press Enter.
- [ ] Paste the remainder of BLOCK 7, then `end` and `write memory`.
- [ ] Verify:
```
show ip interface brief
show ip route
show ip dhcp pool
show access-lists
show ip ssh
```
- [ ] All nine subinterfaces `up/up`, 9 `C` + 9 `L` routes, 7 pools,
      3 ACLs, `SSH Enabled - version 2.0`.
- [ ] Save the PT file.

> `Gi0/0.x` subinterfaces stay **down** until `SW-DIST` is trunking. It is
> normal for them to look down at this point. Re-run
> `show ip interface brief` after Phase 4.

---

## Phase 4 — Configure the distribution switch

- [ ] Click `SW-DIST` → **CLI** tab.
- [ ] Paste `configs/SW-DIST.txt` down to the end of **BLOCK 6**.
- [ ] Paste **BLOCK 7**; answer **1024** at the modulus prompt.
- [ ] Finish with `end` and `write memory`.
- [ ] Verify:
```
show vlan brief
show interfaces trunk
show spanning-tree vlan 10
show ip interface brief
```
- [ ] Ten VLANs, nine trunks with native VLAN 99, `This bridge is the
      root`, `Vlan99 192.168.99.2 up/up`.
- [ ] Go back to `R1-CORE` and re-run `show ip interface brief` — the
      subinterfaces are now `up/up`.
- [ ] `ping 192.168.99.1` from `SW-DIST` must succeed.
- [ ] Save.

---

## Phase 5 — Configure the eight access switches

Repeat for each of `SW-ADMIN`, `SW-CSE`, `SW-ECE`, `SW-MECH`, `SW-LIB`,
`SW-FAC`, `SW-LAB`, `SW-SRV`:

- [ ] Click the switch → **CLI** tab.
- [ ] Paste its file from `configs/` down to the end of **BLOCK 5**.
- [ ] Paste **BLOCK 6**; answer **1024** at the modulus prompt.
- [ ] Finish with `end` and `write memory`.
- [ ] Verify on that switch:
```
show vlan brief
show interfaces trunk
show port-security
ping 192.168.99.1
```

Tick them off:

- [ ] SW-ADMIN (VLAN 10, mgmt .99.11)
- [ ] SW-CSE (VLAN 20, mgmt .99.12)
- [ ] SW-ECE (VLAN 30, mgmt .99.13)
- [ ] SW-MECH (VLAN 40, mgmt .99.14)
- [ ] SW-LIB (VLAN 50, mgmt .99.15)
- [ ] SW-FAC (VLAN 60, mgmt .99.16)
- [ ] SW-LAB (VLAN 70, mgmt .99.17)
- [ ] SW-SRV (VLAN 80, mgmt .99.18)
- [ ] Save.

---

## Phase 6 — Servers (static addressing and services)

Follow `docs/pc-server-setup.md` §2–§5.

- [ ] `SRV-DNS` → IP Configuration → Static → `192.168.80.10` /
      `255.255.255.0` / GW `192.168.80.1` / DNS `192.168.80.10`.
- [ ] `SRV-WEB` → Static → `192.168.80.20`, same mask/GW/DNS.
- [ ] `SRV-APP` → Static → `192.168.80.30`, same mask/GW/DNS.
- [ ] `SRV-DNS` → Services → DNS → **On**, add all five A records.
- [ ] `SRV-DNS` → Services → **DHCP Off**, HTTP Off, FTP Off.
- [ ] `SRV-WEB` → Services → HTTP **On** and HTTPS **On**; edit
      `index.html` and paste `web/index.html`; **Save**.
- [ ] `SRV-WEB` → Services → DHCP **Off**.
- [ ] `SRV-APP` → Services → HTTP **On**; edit `index.html` and paste
      `web/portal.html`; **Save**.
- [ ] `SRV-APP` → Services → FTP **On**; add user `student` /
      `Student@2025` with Write, Read, List.
- [ ] `SRV-APP` → Services → DHCP **Off**.
- [ ] From `SRV-WEB`: `ping 192.168.80.1` must succeed.
- [ ] Save.

---

## Phase 7 — PCs (DHCP)

- [ ] For each of the 36 PCs: Desktop → IP Configuration → **DHCP**.
- [ ] Each one must report `DHCP request successful.`
- [ ] Do one whole department at a time and spot-check with
      `ipconfig /all` on the last PC of each department.
- [ ] On `R1-CORE`: `show ip dhcp binding` shows **36** rows.
- [ ] On `R1-CORE`: `show ip dhcp conflict` is **empty**.
- [ ] Save.

---

## Phase 8 — Full test pass

Work through `docs/testing-results.md` tests 1 to 17 and fill in the
recording sheet.

- [ ] Test 1 — same-VLAN ping, TTL 128
- [ ] Test 2 — gateway ping in all 8 VLANs, TTL 255
- [ ] Test 3 — cross-VLAN matrix, including the two expected ACL failures
- [ ] Test 4 — PC → DNS server
- [ ] Test 5 — PC → Web server
- [ ] Test 6 — `nslookup` for www / dns / portal
- [ ] Test 7 — `ping www.university.local`
- [ ] Test 8 — browser to `http://www.university.local` and
      `http://portal.university.local`
- [ ] Test 9 — `tracert` across VLANs, exactly 2 hops
- [ ] Test 10 — `show vlan brief`
- [ ] Test 11 — `show interfaces trunk`
- [ ] Test 12 — `show ip interface brief`
- [ ] Test 13 — `show ip route`
- [ ] Test 14 — `show ip dhcp binding`
- [ ] Test 15 — `show port-security` and the violation demo
- [ ] Test 16 — SSH allowed from Admin, refused from CSE
- [ ] Save.

---

## Phase 9 — Package and commit

- [ ] `File → Save` one final time.
- [ ] Confirm `packet-tracer/university-network.pkt` exists in the repo
      folder.
- [ ] `File → Save As…` → change **Save as type** to the oldest version
      you might need to open it on (see `docs/portability-notes.md`) and
      save a second copy as
      `packet-tracer/university-network-pt82.pkt`.
- [ ] Run the git commands in `README.md` §"Git workflow".
- [ ] Clone the repo on the second machine and confirm the `.pkt` opens.

---

## Phase 10 — Demo run sheet (12 minutes)

1. Show the topology, name the three layers (00:00–01:00).
2. `show vlan brief` on `SW-CSE`, `show interfaces trunk` on `SW-DIST`
   (01:00–03:00).
3. `show ip interface brief` and `show ip route` on `R1-CORE`
   (03:00–05:00).
4. `ipconfig /all` on a PC, then `show ip dhcp binding` on the router
   (05:00–06:30).
5. Same-VLAN ping (TTL 128) then cross-VLAN ping (TTL 127); explain the
   TTL difference (06:30–08:00).
6. `nslookup`, `ping www.university.local`, then the browser
   (08:00–09:30).
7. From `PC-LAB1` ping Admin → blocked; then browse the portal → allowed.
   Show `show access-lists` counters (09:30–11:00).
8. `show port-security`, move a cable, show the violation counter rise
   (11:00–12:00).
