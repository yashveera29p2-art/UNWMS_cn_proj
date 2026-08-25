# Testing Plan and Expected Results

Project **R075458**. Run these in order. Each test states **what to run**,
**where to run it**, and **what you must see**. Anything different sends
you to [troubleshooting.md](troubleshooting.md).

> Packet Tracer note: the **first** ping of any pair almost always shows
> one `Request timed out.` or a `.` — that is ARP resolving, not a fault.
> Judge a test by pings 2 to 4.

---

## Test 1 — Same-VLAN ping (Layer 2 only, router not involved)

**Where:** `PC-CSE1` → Desktop → Command Prompt

```
ping 192.168.20.12
```

**Expected**

```
Pinging 192.168.20.12 with 32 bytes of data:

Request timed out.
Reply from 192.168.20.12: bytes=32 time<1ms TTL=128
Reply from 192.168.20.12: bytes=32 time<1ms TTL=128
Reply from 192.168.20.12: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.20.12:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
```

**Key detail:** `TTL=128`. The packet never crossed a router, so the TTL
is untouched. Repeat for at least one pair in every VLAN.

---

## Test 2 — Gateway ping (proves the subinterface is alive)

**Where:** one PC in each VLAN → Command Prompt

```
ping 192.168.20.1
```

**Expected:** 4 replies (or 3 after the ARP miss), `TTL=255`.
`TTL=255` means the reply came from the router itself, not from a host
behind it.

| Run on | Command | Must reply |
|--------|---------|-----------|
| PC-ADM1 | `ping 192.168.10.1` | yes |
| PC-CSE1 | `ping 192.168.20.1` | yes |
| PC-ECE1 | `ping 192.168.30.1` | yes |
| PC-MECH1 | `ping 192.168.40.1` | yes |
| PC-LIB1 | `ping 192.168.50.1` | yes |
| PC-FAC1 | `ping 192.168.60.1` | yes |
| PC-LAB1 | `ping 192.168.70.1` | yes |
| SRV-WEB | `ping 192.168.80.1` | yes |

---

## Test 3 — Cross-VLAN ping (proves inter-VLAN routing works)

**Where:** `PC-CSE1` → Command Prompt

```
ping 192.168.30.11
```

**Expected**

```
Reply from 192.168.30.11: bytes=32 time<1ms TTL=127
```

**Key detail:** `TTL=127`, exactly one less than 128. The packet crossed
exactly one router hop — up the trunk into R1-CORE and back down.

### Cross-VLAN matrix to run

| From | To | Expected |
|------|----|----------|
| PC-ADM1 (10) | 192.168.20.11 (CSE) | **Success** |
| PC-CSE1 (20) | 192.168.40.11 (MECH) | **Success** |
| PC-ECE1 (30) | 192.168.50.11 (LIB) | **Success** |
| PC-LIB1 (50) | 192.168.60.11 (FAC) | **Success** |
| PC-FAC1 (60) | 192.168.10.11 (ADMIN) | **Success** |
| PC-LAB1 (70) | 192.168.20.11 (CSE) | **Success** |
| **PC-LAB1 (70)** | **192.168.10.11 (ADMIN)** | **FAILS — blocked by ACL 110** |
| **PC-LAB1 (70)** | **192.168.60.11 (FACULTY)** | **FAILS — blocked by ACL 110** |

The two failures are the design working, not a fault. Expected output for
a blocked ping:

```
Reply from 192.168.70.1: Destination host unreachable.
```

The **router's own address** answers with *host unreachable* because the
ACL dropped the packet and the router sent an ICMP administratively-
prohibited message back.

---

## Test 4 — PC to DNS server

**Where:** `PC-MECH1` → Command Prompt

```
ping 192.168.80.10
```

**Expected:** 4 replies, `TTL=127`.
ACL 120 explicitly permits `icmp any 192.168.80.0 0.0.0.255`, so ping to
the server farm is allowed from every VLAN.

---

## Test 5 — PC to Web server

**Where:** `PC-LIB1` → Command Prompt

```
ping 192.168.80.20
```

**Expected:** 4 replies, `TTL=127`.

---

## Test 6 — Name resolution

**Where:** `PC-ECE1` → Command Prompt

```
nslookup www.university.local
```

**Expected**

```
Server:  [192.168.80.10]
Address: 192.168.80.10

Name:    www.university.local
Address: 192.168.80.20
```

Also test:

```
nslookup portal.university.local     -> 192.168.80.30
nslookup dns.university.local        -> 192.168.80.10
```

---

## Test 7 — Ping by name (DNS + routing together)

**Where:** any PC → Command Prompt

```
ping www.university.local
```

**Expected**

```
Pinging 192.168.80.20 with 32 bytes of data:

Reply from 192.168.80.20: bytes=32 time<1ms TTL=127
Reply from 192.168.80.20: bytes=32 time<1ms TTL=127
Reply from 192.168.80.20: bytes=32 time<1ms TTL=127
```

The first line resolving the name to `192.168.80.20` is the proof that
the DHCP-supplied DNS server option is being used.

---

## Test 8 — Web browser

**Where:** any PC → Desktop → **Web Browser**

Type into the URL bar:

```
http://www.university.local
```

**Expected:** the blue *UNIVERSITY CAMPUS NETWORK* page listing all nine
VLANs and the campus services table.

Also try:

```
http://portal.university.local
http://192.168.80.20
```

Run it from `PC-LAB1` too — the student lab **is** allowed HTTP even
though it is blocked from Admin and Faculty. That single test proves the
ACL is selective rather than a blunt block.

---

## Test 9 — Traceroute across VLANs

**Where:** `PC-LAB1` → Command Prompt

```
tracert 192.168.20.11
```

**Expected**

```
Tracing route to 192.168.20.11 over a maximum of 30 hops:

  1   0 ms      0 ms      0 ms      192.168.70.1
  2   0 ms      0 ms      1 ms      192.168.20.11

Trace complete.
```

**Exactly two hops.** Hop 1 is the router-on-a-stick gateway, hop 2 is
the destination. A same-VLAN `tracert` shows only **one** hop.

---

## Test 10 — `show vlan brief`

**Where:** `SW-CSE` → CLI (privileged EXEC)

```
show vlan brief
```

**Expected (abridged)**

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
10   ADMIN                            active
20   CSE                              active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6
30   ECE                              active
40   MECH                             active
50   LIBRARY                          active
60   FACULTY                          active
70   LAB                              active
80   SERVERS                          active
99   MGMT                             active
999  PARKING-UNUSED                   active    Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                ... Fa0/23, Gig0/1, Gig0/2
```

**What to check**
- All ten VLANs exist and are `active`.
- The department ports are listed under the correct VLAN.
- `Fa0/24` appears under **no** VLAN — trunk ports are never listed here.
  That absence is how you confirm the uplink is trunking.
- Unused ports sit in `999`.

---

## Test 11 — `show interfaces trunk`

**Where:** `SW-DIST` → CLI

```
show interfaces trunk
```

**Expected**

```
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      99
Fa0/2       on           802.1q         trunking      99
Fa0/3       on           802.1q         trunking      99
Fa0/4       on           802.1q         trunking      99
Fa0/5       on           802.1q         trunking      99
Fa0/6       on           802.1q         trunking      99
Fa0/7       on           802.1q         trunking      99
Fa0/8       on           802.1q         trunking      99
Gig0/1      on           802.1q         trunking      99

Port        Vlans allowed on trunk
Fa0/1       10,20,30,40,50,60,70,80,99
Gig0/1      10,20,30,40,50,60,70,80,99

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,40,50,60,70,80,99

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,40,50,60,70,80,99
```

**What to check**
- Nine trunk ports, all `trunking`.
- **Native vlan is 99 on every single line.** A mismatch here is the
  number one cause of a silently broken campus.
- The allowed list contains all nine routed VLANs and nothing else.

Run the same command on each access switch; you should see only `Fa0/24`.

---

## Test 12 — `show ip interface brief`

**Where:** `R1-CORE` → CLI

```
show ip interface brief
```

**Expected**

```
Interface              IP-Address      OK? Method Status                 Protocol
GigabitEthernet0/0     unassigned      YES manual up                     up
GigabitEthernet0/0.10  192.168.10.1    YES manual up                     up
GigabitEthernet0/0.20  192.168.20.1    YES manual up                     up
GigabitEthernet0/0.30  192.168.30.1    YES manual up                     up
GigabitEthernet0/0.40  192.168.40.1    YES manual up                     up
GigabitEthernet0/0.50  192.168.50.1    YES manual up                     up
GigabitEthernet0/0.60  192.168.60.1    YES manual up                     up
GigabitEthernet0/0.70  192.168.70.1    YES manual up                     up
GigabitEthernet0/0.80  192.168.80.1    YES manual up                     up
GigabitEthernet0/0.99  192.168.99.1    YES manual up                     up
GigabitEthernet0/1     unassigned      YES NVRAM  administratively down  down
GigabitEthernet0/2     unassigned      YES NVRAM  administratively down  down
Vlan1                  unassigned      YES NVRAM  administratively down  down
```

**What to check**
- `Gi0/0` is `up/up` with **no** IP address — correct for a trunk parent.
- All nine subinterfaces `up/up`.
- Gi0/1 and Gi0/2 `administratively down` — the "shut unused ports" rule.

**Where:** `SW-ADMIN` → CLI — same command

```
Interface              IP-Address      OK? Method Status                 Protocol
Vlan99                 192.168.99.11   YES manual up                     up
```

---

## Test 13 — `show ip route`

**Where:** `R1-CORE` → CLI

```
show ip route
```

**Expected**

```
Codes: L - local, C - connected, S - static, ...

Gateway of last resort is not set

     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
     192.168.30.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.30.0/24 is directly connected, GigabitEthernet0/0.30
L       192.168.30.1/32 is directly connected, GigabitEthernet0/0.30
     192.168.40.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.40.0/24 is directly connected, GigabitEthernet0/0.40
L       192.168.40.1/32 is directly connected, GigabitEthernet0/0.40
     192.168.50.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.50.0/24 is directly connected, GigabitEthernet0/0.50
L       192.168.50.1/32 is directly connected, GigabitEthernet0/0.50
     192.168.60.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.60.0/24 is directly connected, GigabitEthernet0/0.60
L       192.168.60.1/32 is directly connected, GigabitEthernet0/0.60
     192.168.70.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.70.0/24 is directly connected, GigabitEthernet0/0.70
L       192.168.70.1/32 is directly connected, GigabitEthernet0/0.70
     192.168.80.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.80.0/24 is directly connected, GigabitEthernet0/0.80
L       192.168.80.1/32 is directly connected, GigabitEthernet0/0.80
     192.168.99.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.99.0/24 is directly connected, GigabitEthernet0/0.99
L       192.168.99.1/32 is directly connected, GigabitEthernet0/0.99
```

**What to check:** exactly **9 `C` routes and 9 `L` routes**. No routing
protocol is needed — every subnet is directly connected to a
subinterface. If a `C` route is missing, that subinterface is down.

---

## Test 14 — `show ip dhcp binding`

**Where:** `R1-CORE` → CLI, after every PC has been set to DHCP

```
show ip dhcp binding
```

**Expected (abridged — 36 rows in total)**

```
IP address       Client-ID/              Lease expiration    Type
                 Hardware address
192.168.10.11    0001.4256.9C01          --                  Automatic
192.168.10.12    0060.3E4A.11B2          --                  Automatic
192.168.20.11    00D0.BC12.44A1          --                  Automatic
192.168.20.12    0002.1799.62C3          --                  Automatic
192.168.70.18    000C.85A1.7E44          --                  Automatic
```

**What to check**
- 36 bindings, four to eight per user VLAN.
- No address below `.11` in any VLAN — that proves the
  `ip dhcp excluded-address` statements took effect.
- No entries at all for `192.168.80.x` — servers are static, as designed.

Related commands:

```
show ip dhcp pool
show ip dhcp conflict
```

`show ip dhcp conflict` must be **empty**. Any row here means a static
address collides with a pool.

---

## Test 15 — `show port-security`

**Where:** `SW-LAB` → CLI

```
show port-security
```

**Expected**

```
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
     Fa0/1          1             1                0             Restrict
     Fa0/2          1             1                0             Restrict
     Fa0/3          1             1                0             Restrict
     Fa0/4          1             1                0             Restrict
     Fa0/5          1             1                0             Restrict
     Fa0/6          1             1                0             Restrict
     Fa0/7          1             1                0             Restrict
     Fa0/8          1             1                0             Restrict
```

Then:

```
show port-security address
```

**Expected**

```
          Secure Mac Address Table
-----------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining
                                                                  Age
----    -----------       ----                          -----   ---------
  70    0001.4256.9C01    SecureSticky                  Fa0/1        -
  70    0060.3E4A.11B2    SecureSticky                  Fa0/2        -
```

**What to check**
- `MaxSecureAddr` is 1 and `CurrentAddr` is 1 on every connected port.
- `Security Action` is `Restrict`.
- `SecurityViolation` is 0 in a healthy network.
- The learned type is `SecureSticky`, not `SecureDynamic`.

### Deliberately trigger a violation — a strong demo

1. Delete the cable between `PC-LAB1` and `SW-LAB Fa0/1`.
2. Plug `PC-LAB2` into `Fa0/1` instead.
3. Ping something from `PC-LAB2`.
4. On `SW-LAB` run `show port-security interface FastEthernet0/1`.

```
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0060.3E4A.11B2:70
Security Violation Count   : 1
```

The counter increments and the frames are dropped, but the port stays
`Secure-up` — that is exactly what `violation restrict` means, as opposed
to `violation shutdown` which would err-disable the port.

Undo it by putting `PC-LAB1` back on `Fa0/1`.

---

## Test 16 — SSH management access

**Where:** `PC-ADM1` → Desktop → Command Prompt.
It must be an Administration PC — ACL 10 blocks every other subnet.

```
ssh -l netadmin 192.168.10.1
```

Enter password `Adm1n@Univ#2025`.

**Expected:** the login banner, then the `R1-CORE>` prompt. Type
`enable`, give `Univ@Core#2025`, and you are at `R1-CORE#`.

**Negative test — this one is meant to fail.** From `PC-CSE1`:

```
ssh -l netadmin 192.168.10.1
```

**Expected:** `% Connection refused by remote host`, or a silent timeout.
ACL 10 permits only `192.168.10.0/24` on the VTY lines.

Also SSH into a switch from `PC-ADM1`:

```
ssh -l netadmin 192.168.99.11
```

On the router, confirm the session:

```
show ssh
show users
```

---

## Test 17 — Result recording sheet

Fill this in while demonstrating.

| # | Test | Device | Command | Expected | Pass/Fail |
|---|------|--------|---------|----------|-----------|
| 1 | Same-VLAN ping | PC-CSE1 | `ping 192.168.20.12` | Replies, TTL 128 | |
| 2 | Gateway ping | PC-ADM1 | `ping 192.168.10.1` | Replies, TTL 255 | |
| 3 | Cross-VLAN ping | PC-CSE1 | `ping 192.168.30.11` | Replies, TTL 127 | |
| 4 | ACL block Lab→Admin | PC-LAB1 | `ping 192.168.10.11` | Host unreachable | |
| 5 | ACL block Lab→Faculty | PC-LAB1 | `ping 192.168.60.11` | Host unreachable | |
| 6 | PC→DNS | PC-MECH1 | `ping 192.168.80.10` | Replies | |
| 7 | PC→Web | PC-LIB1 | `ping 192.168.80.20` | Replies | |
| 8 | Name lookup | PC-ECE1 | `nslookup www.university.local` | 192.168.80.20 | |
| 9 | Ping by name | PC-FAC1 | `ping www.university.local` | Resolves + replies | |
| 10 | Browser | PC-LAB1 | `http://www.university.local` | Portal page loads | |
| 11 | Traceroute | PC-LAB1 | `tracert 192.168.20.11` | 2 hops | |
| 12 | VLANs | SW-CSE | `show vlan brief` | 10 VLANs, ports correct | |
| 13 | Trunks | SW-DIST | `show interfaces trunk` | 9 trunks, native 99 | |
| 14 | Interfaces | R1-CORE | `show ip interface brief` | 9 subints up/up | |
| 15 | Routes | R1-CORE | `show ip route` | 9 C + 9 L | |
| 16 | DHCP | R1-CORE | `show ip dhcp binding` | 36 bindings | |
| 17 | Port security | SW-LAB | `show port-security` | 8 ports, max 1, Restrict | |
| 18 | SSH allowed | PC-ADM1 | `ssh -l netadmin 192.168.10.1` | Logs in | |
| 19 | SSH denied | PC-CSE1 | `ssh -l netadmin 192.168.10.1` | Refused | |
| 20 | ACL counters | R1-CORE | `show access-lists` | Match counts rising | |

---

## Test 18 — Simulation mode (optional, very convincing in a demo)

1. Switch Packet Tracer from **Realtime** to **Simulation** (bottom right).
2. **Edit Filters** → uncheck everything, then check only **ICMP** and
   **ARP**.
3. From `PC-CSE1` ping `192.168.30.11`.
4. Press **Capture / Forward** repeatedly and watch the packet:
   - PC → SW-CSE (untagged access frame)
   - SW-CSE → SW-DIST (frame now carries an 802.1Q tag for VLAN 20)
   - SW-DIST → R1-CORE (still tagged VLAN 20)
   - R1-CORE re-tags it as VLAN 30 and sends it back down
   - SW-DIST → SW-ECE → PC-ECE1
5. Click any packet and open the **Inbound PDU Details** tab to see the
   802.1Q header appear and disappear.

This is the clearest possible visual proof that router-on-a-stick is
doing what the configuration claims.
