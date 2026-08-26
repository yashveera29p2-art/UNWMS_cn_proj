# packet-tracer/

The Packet Tracer simulation lives here.

| File | What it is |
|------|-----------|
| `university-network.pkt` | The working simulation. **Built and saved on Packet Tracer 9.0.1** |

## Opening it

1. Install **Packet Tracer 9.0.1 or newer**, and sign in with a Cisco
   NetAcad / Skills for All account. PT 8.0+ will not open any file until
   you are signed in.
2. `File > Open` then `university-network.pkt`
3. Wait about 30 seconds for the link lights to go green. Spanning tree is
   converging; amber during that window is normal.

Verify it opened correctly - on `R1-CORE`, CLI tab, `enable` with password
`Univ@Core#2025`:

```
show ip interface brief
```

All nine `Gi0/0.x` subinterfaces should read `up / up`. Then from any PC,
Desktop > Command Prompt:

```
ping www.university.local
```

That single command exercises DHCP, DNS, inter-VLAN routing and the server
VLAN at once. If it resolves and replies, the whole network is healthy.

## If it will not open

`"The file is not a valid Packet Tracer file"` means your Packet Tracer is
**older than 9.0.1**. Compatibility runs one way only: newer Packet Tracer
opens older files, older Packet Tracer never opens newer ones, and the
error message does not tell you which version created the file.

Three ways out, in order of preference:

1. Upgrade Packet Tracer on that machine to 9.0.1 or newer.
2. On a machine that *can* open it, use `File > Save As...` and pick an
   older entry in the *Save as type* dropdown, then commit that copy
   alongside this one with a version-tagged name such as
   `university-network-pt82.pkt`.
3. Rebuild from scratch using `../configs/` and `../paste/`. This takes
   about 90 minutes and is exactly why those folders exist - a version
   mismatch is an inconvenience, not a loss of work. See
   `../docs/build-checklist.md`.

## Two rules for this folder

1. **Record the Packet Tracer version in every commit message** that
   touches the `.pkt`. Nothing else records it, and the next person needs
   it to know whether their install can open the file.
2. **Never edit the `.pkt` on two machines at once.** Git treats it as an
   opaque binary and cannot merge it. Pull before you open Packet Tracer,
   push as soon as you close it. If a conflict does happen, one side's
   work has to be discarded and rebuilt by hand.
