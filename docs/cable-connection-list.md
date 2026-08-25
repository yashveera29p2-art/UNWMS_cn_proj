# Cable and Connection List

Project **R075458**. 48 cables total. Build them in this exact order and
the topology will match the configs with no rework.

In Packet Tracer choose the cable from the **Connections** (lightning
bolt) category:
`Copper Straight-Through` and `Copper Cross-Over`.

---

## Section A — Backbone links (9 cables, all TRUNK)

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 1 | R1-CORE | Gi0/0 | → | SW-DIST | Gi0/1 | **TRUNK** (802.1Q, native 99) | Copper Straight-Through |
| 2 | SW-DIST | Fa0/1 | → | SW-ADMIN | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |
| 3 | SW-DIST | Fa0/2 | → | SW-CSE | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |
| 4 | SW-DIST | Fa0/3 | → | SW-ECE | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |
| 5 | SW-DIST | Fa0/4 | → | SW-MECH | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |
| 6 | SW-DIST | Fa0/5 | → | SW-LIB | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |
| 7 | SW-DIST | Fa0/6 | → | SW-FAC | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |
| 8 | SW-DIST | Fa0/7 | → | SW-LAB | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |
| 9 | SW-DIST | Fa0/8 | → | SW-SRV | Fa0/24 | **TRUNK** (802.1Q, native 99) | Copper Cross-Over |

## Section B — End-device links (39 cables, all ACCESS)

### Administration — VLAN 10 on `SW-ADMIN`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 10 | PC-ADM1 | FastEthernet0 | → | SW-ADMIN | Fa0/1 | **ACCESS** VLAN 10 | Copper Straight-Through |
| 11 | PC-ADM2 | FastEthernet0 | → | SW-ADMIN | Fa0/2 | **ACCESS** VLAN 10 | Copper Straight-Through |
| 12 | PC-ADM3 | FastEthernet0 | → | SW-ADMIN | Fa0/3 | **ACCESS** VLAN 10 | Copper Straight-Through |
| 13 | PC-ADM4 | FastEthernet0 | → | SW-ADMIN | Fa0/4 | **ACCESS** VLAN 10 | Copper Straight-Through |

### Computer Science — VLAN 20 on `SW-CSE`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 14 | PC-CSE1 | FastEthernet0 | → | SW-CSE | Fa0/1 | **ACCESS** VLAN 20 | Copper Straight-Through |
| 15 | PC-CSE2 | FastEthernet0 | → | SW-CSE | Fa0/2 | **ACCESS** VLAN 20 | Copper Straight-Through |
| 16 | PC-CSE3 | FastEthernet0 | → | SW-CSE | Fa0/3 | **ACCESS** VLAN 20 | Copper Straight-Through |
| 17 | PC-CSE4 | FastEthernet0 | → | SW-CSE | Fa0/4 | **ACCESS** VLAN 20 | Copper Straight-Through |
| 18 | PC-CSE5 | FastEthernet0 | → | SW-CSE | Fa0/5 | **ACCESS** VLAN 20 | Copper Straight-Through |
| 19 | PC-CSE6 | FastEthernet0 | → | SW-CSE | Fa0/6 | **ACCESS** VLAN 20 | Copper Straight-Through |

### Electronics — VLAN 30 on `SW-ECE`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 20 | PC-ECE1 | FastEthernet0 | → | SW-ECE | Fa0/1 | **ACCESS** VLAN 30 | Copper Straight-Through |
| 21 | PC-ECE2 | FastEthernet0 | → | SW-ECE | Fa0/2 | **ACCESS** VLAN 30 | Copper Straight-Through |
| 22 | PC-ECE3 | FastEthernet0 | → | SW-ECE | Fa0/3 | **ACCESS** VLAN 30 | Copper Straight-Through |
| 23 | PC-ECE4 | FastEthernet0 | → | SW-ECE | Fa0/4 | **ACCESS** VLAN 30 | Copper Straight-Through |
| 24 | PC-ECE5 | FastEthernet0 | → | SW-ECE | Fa0/5 | **ACCESS** VLAN 30 | Copper Straight-Through |
| 25 | PC-ECE6 | FastEthernet0 | → | SW-ECE | Fa0/6 | **ACCESS** VLAN 30 | Copper Straight-Through |

### Mechanical — VLAN 40 on `SW-MECH`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 26 | PC-MECH1 | FastEthernet0 | → | SW-MECH | Fa0/1 | **ACCESS** VLAN 40 | Copper Straight-Through |
| 27 | PC-MECH2 | FastEthernet0 | → | SW-MECH | Fa0/2 | **ACCESS** VLAN 40 | Copper Straight-Through |
| 28 | PC-MECH3 | FastEthernet0 | → | SW-MECH | Fa0/3 | **ACCESS** VLAN 40 | Copper Straight-Through |
| 29 | PC-MECH4 | FastEthernet0 | → | SW-MECH | Fa0/4 | **ACCESS** VLAN 40 | Copper Straight-Through |

### Library — VLAN 50 on `SW-LIB`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 30 | PC-LIB1 | FastEthernet0 | → | SW-LIB | Fa0/1 | **ACCESS** VLAN 50 | Copper Straight-Through |
| 31 | PC-LIB2 | FastEthernet0 | → | SW-LIB | Fa0/2 | **ACCESS** VLAN 50 | Copper Straight-Through |
| 32 | PC-LIB3 | FastEthernet0 | → | SW-LIB | Fa0/3 | **ACCESS** VLAN 50 | Copper Straight-Through |
| 33 | PC-LIB4 | FastEthernet0 | → | SW-LIB | Fa0/4 | **ACCESS** VLAN 50 | Copper Straight-Through |

### Faculty — VLAN 60 on `SW-FAC`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 34 | PC-FAC1 | FastEthernet0 | → | SW-FAC | Fa0/1 | **ACCESS** VLAN 60 | Copper Straight-Through |
| 35 | PC-FAC2 | FastEthernet0 | → | SW-FAC | Fa0/2 | **ACCESS** VLAN 60 | Copper Straight-Through |
| 36 | PC-FAC3 | FastEthernet0 | → | SW-FAC | Fa0/3 | **ACCESS** VLAN 60 | Copper Straight-Through |
| 37 | PC-FAC4 | FastEthernet0 | → | SW-FAC | Fa0/4 | **ACCESS** VLAN 60 | Copper Straight-Through |

### Student Labs — VLAN 70 on `SW-LAB`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 38 | PC-LAB1 | FastEthernet0 | → | SW-LAB | Fa0/1 | **ACCESS** VLAN 70 | Copper Straight-Through |
| 39 | PC-LAB2 | FastEthernet0 | → | SW-LAB | Fa0/2 | **ACCESS** VLAN 70 | Copper Straight-Through |
| 40 | PC-LAB3 | FastEthernet0 | → | SW-LAB | Fa0/3 | **ACCESS** VLAN 70 | Copper Straight-Through |
| 41 | PC-LAB4 | FastEthernet0 | → | SW-LAB | Fa0/4 | **ACCESS** VLAN 70 | Copper Straight-Through |
| 42 | PC-LAB5 | FastEthernet0 | → | SW-LAB | Fa0/5 | **ACCESS** VLAN 70 | Copper Straight-Through |
| 43 | PC-LAB6 | FastEthernet0 | → | SW-LAB | Fa0/6 | **ACCESS** VLAN 70 | Copper Straight-Through |
| 44 | PC-LAB7 | FastEthernet0 | → | SW-LAB | Fa0/7 | **ACCESS** VLAN 70 | Copper Straight-Through |
| 45 | PC-LAB8 | FastEthernet0 | → | SW-LAB | Fa0/8 | **ACCESS** VLAN 70 | Copper Straight-Through |

### Server Room — VLAN 80 on `SW-SRV`

| # | Source device | Source port | → | Destination device | Destination port | Link type | Cable |
|---|---------------|-------------|---|--------------------|------------------|-----------|-------|
| 46 | SRV-DNS | FastEthernet0 | → | SW-SRV | Fa0/1 | **ACCESS** VLAN 80 | Copper Straight-Through |
| 47 | SRV-WEB | FastEthernet0 | → | SW-SRV | Fa0/2 | **ACCESS** VLAN 80 | Copper Straight-Through |
| 48 | SRV-APP | FastEthernet0 | → | SW-SRV | Fa0/3 | **ACCESS** VLAN 80 | Copper Straight-Through |

---

## Ports deliberately left with NO cable

| Device | Ports | State after config |
|--------|-------|--------------------|
| R1-CORE | Gi0/1, Gi0/2 | `shutdown`, no IP |
| SW-DIST | Fa0/9 – Fa0/24, Gi0/2 | VLAN 999, `shutdown` |
| SW-ADMIN | Fa0/5 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |
| SW-CSE | Fa0/7 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |
| SW-ECE | Fa0/7 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |
| SW-MECH | Fa0/5 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |
| SW-LIB | Fa0/5 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |
| SW-FAC | Fa0/5 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |
| SW-LAB | Fa0/9 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |
| SW-SRV | Fa0/4 – Fa0/23, Gi0/1, Gi0/2 | VLAN 999, `shutdown` |

> **Order matters.** Attach the cables *before* pasting the switch
> configs. Port security uses `mac-address sticky`, so the switch learns
> and pins the MAC of whatever is already plugged in. If you cable
> afterwards it still works — the port simply learns on first frame —
> but if you later move a PC to a different port you must clear the old
> sticky entry (`no switchport port-security mac-address sticky` then
> re-enable) or the port will report violations.
