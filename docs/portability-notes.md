# Portability, Version and Packet Tracer Limitation Notes

Project **R075458**. Read this before moving the project between
machines.

---

## 1. The `.pkt` version trap — the one that will bite you

**A `.pkt` saved by a newer Packet Tracer will not open in an older one.**
The older PT throws *"The file is not a valid Packet Tracer file"* or
*"This file was created by a newer version"* and there is no repair, no
converter and no downgrade path.

Version compatibility is **one-way only**: newer PT opens older files,
older PT never opens newer files.

### The workaround — save down a version

In Packet Tracer:

1. `File → Save As…`
2. In the **Save as type** dropdown, pick the *oldest* version you might
   need. PT offers entries such as
   *Packet Tracer 8.2 File (\*.pkt)*, *Packet Tracer 8.0 File*,
   *Packet Tracer 7.3 File*.
3. Save it under a version-tagged name, e.g.
   `packet-tracer/university-network-pt82.pkt`.

### The rule for this project

| Machine | Action |
|---------|--------|
| Build PC | Install the **same PT version** as the laptop, or the older of the two |
| Laptop | Install the **same or newer** PT version than the build PC |
| Repo | Commit **both** the native `.pkt` and a saved-down `-pt82.pkt` |

Check your version at **Help → About**. Note it in your commit message.

> Saving down can silently drop features introduced after that version.
> Always re-open the saved-down copy once and re-run two or three tests
> from `docs/testing-results.md` before you rely on it.

---

## 2. Why `configs/` exists at all

The nine plain-text config files are the real insurance policy. If the
`.pkt` will not open on the other machine — wrong version, corrupted
download, PT refusing to sign in — you can rebuild the entire campus in
under an hour by:

1. Placing the devices (`docs/build-checklist.md` Phase 1)
2. Cabling them (`docs/cable-connection-list.md`)
3. Pasting the nine files from `configs/`
4. Doing the GUI steps in `docs/pc-server-setup.md`

Plain text also survives git diffs, code review and printing. A `.pkt`
does none of those things — it is an opaque binary blob to git.

---

## 3. Version-sensitive commands used in this project

Everything below works in **PT 8.2 and 9.x**. Notes are for older builds.

| Command | 8.2 / 9.x | Older PT | Workaround |
|---------|-----------|----------|-----------|
| `encapsulation dot1Q <id> native` | works | works from 5.3 | none needed |
| `switchport trunk allowed vlan <list>` | works | works | none needed |
| `switchport port-security violation restrict` | works | works from 6.x | use `violation shutdown` if rejected |
| `switchport port-security mac-address sticky` | works | works | none needed |
| `spanning-tree portfast` | works | works | on some images it is `spanning-tree portfast` only in interface mode — it already is here |
| `spanning-tree bpduguard enable` | works | works from 6.x | omit the line if the parser rejects it |
| `spanning-tree vlan <id> priority 4096` | works | works | `root primary` is an alternative but is less reliably parsed |
| `ip ssh version 2` | works | works from 5.3 | omit; SSH still runs at 1.99 |
| `crypto key generate rsa` | interactive | interactive | always answer `1024` |
| `switchport nonegotiate` | works | works | drop the line if rejected — trunk is already hard-set |
| `ip dhcp pool` + `domain-name` | works | works | none needed |
| `access-list 120 permit icmp any <net> <wc>` | works | works | none needed |
| `line vty 5 15` | works | works | if rejected, `line vty 0 4` alone is enough for a lab |

If any single line is rejected by an older image, the CLI prints
`% Invalid input detected at '^' marker.` and **skips only that line** —
the rest of the paste continues. Re-read the block afterwards with
`show running-config` to see what actually landed.

---

## 4. Packet Tracer limitations that shaped this design

These are simulator constraints, not mistakes in the network design.

### 4.1 The PT web browser understands almost no CSS

The built-in browser on a PC renders a small subset of HTML 3.2-era
markup. External stylesheets, `<style>` blocks, flexbox, grid and most
modern attributes are ignored or break the page.

**Workaround used:** `web/index.html` and `web/portal.html` are written
with `<table>`, `<font>`, `<center>` and `bgcolor` attributes. It looks
dated, and it is the only thing that renders correctly inside PT.

### 4.2 A PT `Server` can run DHCP, and it must not here

Every PT Server has a DHCP service. If it is left on, it races the
router's DHCP server and PCs get random addresses from the wrong pool.

**Workaround used:** `docs/pc-server-setup.md` explicitly turns DHCP
**Off** on all three servers, and `docs/troubleshooting.md` Symptom 4
covers the symptom if it happens anyway.

### 4.3 No real DNS forwarding

The PT DNS server answers only from its own static A-record table. There
is no root hint, no forwarder, no recursion to the outside world.

**Workaround used:** every name the project tests is present as an
explicit A record. Do not expect `ping google.com` to work — there is no
internet in this topology by design.

### 4.4 Router-on-a-Stick is a bandwidth hairpin

Every inter-VLAN packet crosses `Gi0/0` twice. On real hardware with
1500 users this is the wrong answer; a Layer-3 switch with SVIs would be
right.

**Why it is used anyway:** the brief asked for the simpler reliable
option. In Packet Tracer a 2911 with subinterfaces is far easier to
demonstrate and debug than a 3560 with `ip routing`, and the traffic in a
49-device lab is nowhere near the link limit. The SVI alternative is
sketched in §5 below.

### 4.5 Port security + sticky MAC is fragile against re-cabling

PT learns the sticky MAC on first frame. Moving a PC between ports after
`write memory` produces violations that look like a broken network.

**Workaround used:** cable everything before pasting configs
(`build-checklist.md` Phase 2 precedes Phase 5), and
`troubleshooting.md` Symptom 10 gives the recovery commands.

### 4.6 Gigabit and FastEthernet ports do not interoperate

Connecting a `Gi0/1` on one switch to a `Fa0/24` on another leaves the
link permanently down in PT.

**Workaround used:** the switch-to-switch links are Fa-to-Fa
(`SW-DIST Fa0/1-8` → access `Fa0/24`) and only the router link is
Gi-to-Gi (`R1-CORE Gi0/0` → `SW-DIST Gi0/1`).

### 4.7 PT requires a login

Packet Tracer 8.0+ will not open a file at all until you sign in with a
Cisco Networking Academy or Skills for All account. On a fresh laptop,
sign in first, then open the `.pkt`.

---

## 5. If you would rather use a Layer-3 switch

Swap `R1-CORE` + `SW-DIST` for a single **3560** and replace the
subinterfaces with SVIs:

```
ip routing
!
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
interface Vlan20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
!
... one per VLAN, through Vlan80 and Vlan99
```

Everything else — DHCP pools, ACLs, port security, DNS, HTTP — is
unchanged, because none of it depends on *how* the gateway is
implemented.

**This project does not use that option.** Reasons: the 3560 in Packet
Tracer needs `ip routing` enabled manually (a silent failure if
forgotten), DHCP-on-a-3560 behaves less predictably in PT than on a 2911,
and the brief asked for the simpler reliable choice. Router-on-a-Stick
makes the 802.1Q tagging visible in Simulation mode, which is worth a lot
in a demo.
