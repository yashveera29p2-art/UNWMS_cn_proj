# Running the Project on a Second Machine

Project **R075458** / repo `chandu242006/CN_proj`.

---

## Read this first

Cloning the repository does **not** give you a running network. A git
clone gives you text files. The Packet Tracer simulation (`.pkt`) is a
separate artifact that has to be built once in the GUI.

Check what you actually have:

```
ls packet-tracer/
```

| What you see | What it means | Go to |
|--------------|---------------|-------|
| `README.md` only | No simulation built yet | **Route B** |
| `university-network.pkt` | Simulation exists | **Route A** |

---

## Route A - the `.pkt` exists (5 minutes)

Use this once somebody has built the network and pushed the `.pkt`.

### A1. Prerequisites on the laptop

| Item | Notes |
|------|-------|
| Cisco Packet Tracer | **Same version or newer** than the machine that saved the file. This project was built on **9.0.1** |
| Cisco account | PT 8.0+ refuses to open any file until you sign in (NetAcad / Skills for All) |
| Git | `git --version` to check; install from git-scm.com if missing |

### A2. Clone

Pick a folder that is **not** inside OneDrive, Dropbox or Google Drive.
Cloud sync corrupts `.git` and gives you two competing copies.

```
cd C:\Users\<you>\Projects
git clone https://github.com/chandu242006/CN_proj.git
cd CN_proj
```

### A3. Open the simulation

1. Launch Packet Tracer, sign in with your Cisco account.
2. `File > Open` then `packet-tracer/university-network.pkt`
3. Wait about 30 seconds for the link lights to turn green. Spanning tree
   is converging; amber is normal during that window.

### A4. Verify it survived the trip

Click `R1-CORE`, then the **CLI** tab, press Enter:

```
enable
```

password `Univ@Core#2025`

```
show ip interface brief
```

All nine `Gi0/0.x` subinterfaces must read `up / up`.

Then on any PC, **Desktop > Command Prompt**:

```
ipconfig /all
ping www.university.local
```

If those two work, the whole stack works - DHCP, DNS, routing and the
server VLAN are all exercised by that one test.

### If the file will not open

`"The file is not a valid Packet Tracer file"` means your Packet Tracer
is **older** than the one that saved it. Compatibility is one-way only.
Either upgrade PT on this laptop, or ask the other machine to use
`File > Save As...` and pick an older version from the *Save as type*
dropdown. Fall back to Route B if neither is possible.

---

## Route B - no `.pkt` yet, build it from `configs/` (60-90 minutes)

This is the situation right now. It is also the reason `configs/` exists:
the project is fully rebuildable from plain text, so a version mismatch
or a corrupted download is an inconvenience, not a loss.

### B1. Clone (same as A2)

```
git clone https://github.com/chandu242006/CN_proj.git
```

### B2. Build, in this order

| Step | Follow this file | What you do |
|------|------------------|-------------|
| 1 | `docs/build-checklist.md` Phase 1 | Place 49 devices, rename them exactly |
| 2 | `docs/cable-connection-list.md` | Run all 48 cables |
| 3 | `configs/R1-core-router.txt` | Paste into R1-CORE's CLI tab |
| 4 | `configs/SW-DIST.txt` | Paste into SW-DIST |
| 5 | `configs/SW-*.txt` (8 files) | Paste one per access switch |
| 6 | `docs/pc-server-setup.md` | Static IPs + services on 3 servers, DHCP on 36 PCs |
| 7 | `web/index.html`, `web/portal.html` | Paste into SRV-WEB and SRV-APP |
| 8 | `docs/testing-results.md` | Run the 18 tests |

Two things that trip everyone up:

- **Cable before pasting the switch configs.** Port security uses sticky
  MAC learning - let each port see the right device first.
- **`crypto key generate rsa` stops the paste.** When PT prints
  `How many bits in the modulus [512]:` type `1024`, press Enter, then
  paste the rest of the file.

If a command is rejected, `docs/troubleshooting.md` and
`docs/portability-notes.md` section 3 cover it.

### B3. Save it into the repo and push

In Packet Tracer: `File > Save As...` then
`packet-tracer/university-network.pkt`

Then:

```
git add packet-tracer/university-network.pkt
git commit -m "Add Packet Tracer simulation file (built on PT 9.0.1)"
git push
```

**Put the PT version in the commit message.** Nothing else records it,
and the next person needs it to know whether their PT can open the file.

After this, everyone else uses Route A.

---

## Working across two machines afterwards

- `git pull` **always**, before you open Packet Tracer
- `git push` as soon as you close Packet Tracer

### Why the discipline matters

A `.pkt` is a binary blob. Git cannot merge it. If you edit the
simulation on the PC and the laptop without pulling in between, git
reports a conflict it has no way to resolve, and one side's work is
gone.

If a conflict does happen, you have to pick one side. Check out the
`.pkt` using one of these flags, then `git add` and `git commit` it:

| Which copy you want to keep | Flag |
|------------------------------|------|
| The one from GitHub (the other machine) | `--theirs` |
| The one on this machine | `--ours` |

Then rebuild whatever the losing side contained, by hand, from
`configs/`.

Text files under `configs/` and `docs/` merge normally - those you can
edit on both machines safely.

---

## Quick reference

| Task | Command |
|------|---------|
| Clone | `git clone https://github.com/chandu242006/CN_proj.git` |
| Get latest | `git pull` |
| See what changed | `git status` |
| Stage everything | `git add -A` |
| Commit | `git commit -m "message"` |
| Send changes | `git push` |
| See history | `git log --oneline` |
| Discard uncommitted edits to one file | `git restore <file>` |
