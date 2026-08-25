# packet-tracer/

The Packet Tracer simulation lives here.

| File | What it is |
|------|-----------|
| `university-network.pkt` | The working simulation, saved in the newest PT version you use |
| `university-network-pt82.pkt` | Optional. The same network re-saved down to PT 8.2 for older installs |

## The file is not in the repository yet

The `.pkt` is produced by *building* the network, which is a GUI task
Packet Tracer alone can do. Follow
[`../docs/build-checklist.md`](../docs/build-checklist.md), then:

```
File -> Save As...  ->  packet-tracer/university-network.pkt
```

and commit it:

```bash
git add packet-tracer/university-network.pkt
git commit -m "Add Packet Tracer simulation file (saved from PT 9.0)"
git push
```

## Two rules for this folder

1. **Record the PT version in the commit message.** An older Packet
   Tracer cannot open a newer `.pkt`, and there is no error message that
   tells you which version made it.
2. **Never edit the `.pkt` on two machines at once.** Git treats it as an
   opaque binary and cannot merge it. Pull before you open PT, push as
   soon as you close it.

If you ever need to hand the file to someone on an older install, use
`File -> Save As...` and choose an older entry in the *Save as type*
dropdown. See [`../docs/portability-notes.md`](../docs/portability-notes.md).
