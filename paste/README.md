# paste/ - Ready-to-Paste Configuration Chunks

Thirty-one files. Each one is a single copy-paste operation into one
device's CLI, in numbered order.

## Why these exist

The files in `configs/` are the readable, annotated source of truth. They
are 200-340 lines each, mostly comments, and Packet Tracer silently drops
characters when you paste that much at once - which produces a config
that looks fine and does not work.

These chunks are the same commands with every comment and blank line
removed, split into pieces of at most ~70 lines, and cut at exactly the
point where `crypto key generate rsa` stops the paste to ask for a key
size.

Nothing here contradicts `configs/`. If you want to understand *why* a
command is there, read `configs/`. If you want to build the network
quickly and without transcription errors, paste from here.

## How to use

For each file, in numeric order:

1. Open the file in Notepad
2. **Ctrl+A** then **Ctrl+C**
3. Click the device named in the filename, open its **CLI** tab
4. Click inside the CLI window, **Ctrl+V**
5. Wait for the prompt to come back before the next file

## The order

| # | File | Device | Notes |
|---|------|--------|-------|
| 01 | `01_R1-CORE_part1.txt` | R1-CORE | Identity, banner, interfaces, all 9 subinterfaces |
| 02 | `02_R1-CORE_part2.txt` | R1-CORE | DHCP exclusions and pools |
| 03 | `03_R1-CORE_part3.txt` | R1-CORE | ACLs. **Ends at the RSA prompt** |
| -- | *type* `1024` *and press Enter* | R1-CORE | |
| 04 | `04_R1-CORE_part4.txt` | R1-CORE | SSH, console, VTY, `write memory` |
| 05 | `05_SW-DIST_part1.txt` | SW-DIST | Identity, VLAN database |
| 06 | `06_SW-DIST_part2.txt` | SW-DIST | Trunks, STP, unused ports. **Ends at the RSA prompt** |
| -- | *type* `1024` *and press Enter* | SW-DIST | |
| 07 | `07_SW-DIST_part3.txt` | SW-DIST | Management SVI, SSH, lines, `write memory` |
| 08 | `08_SW-ADMIN_part1.txt` | SW-ADMIN | Identity, VLAN database |
| 09 | `09_SW-ADMIN_part2.txt` | SW-ADMIN | Ports, trunk, security. **Ends at the RSA prompt** |
| -- | *type* `1024` *and press Enter* | SW-ADMIN | |
| 10 | `10_SW-ADMIN_part3.txt` | SW-ADMIN | SVI, SSH, lines, `write memory` |
| 11-13 | `11..13_SW-CSE_*` | SW-CSE | Same three-step pattern |
| 14-16 | `14..16_SW-ECE_*` | SW-ECE | Same three-step pattern |
| 17-19 | `17..19_SW-MECH_*` | SW-MECH | Same three-step pattern |
| 20-22 | `20..22_SW-LIB_*` | SW-LIB | Same three-step pattern |
| 23-25 | `23..25_SW-FAC_*` | SW-FAC | Same three-step pattern |
| 26-28 | `26..28_SW-LAB_*` | SW-LAB | Same three-step pattern |
| 29-31 | `29..31_SW-SRV_*` | SW-SRV | Same three-step pattern |

Every `part2` on a switch, and `part3` on the router, ends on the line
`crypto key generate rsa`. That is deliberate. Packet Tracer will print:

```
The name for the keys will be: ...
How many bits in the modulus [512]:
```

Type **1024**, press Enter, wait for `[OK]`, then paste the next file.

Every device's final chunk ends with `write memory`. If you do not see
`[OK]` after it, the configuration is not saved and will be lost.

## Before the first paste on a fresh device

A brand-new router or switch asks:

```
Would you like to enter the initial configuration dialog? [yes/no]:
```

Type **no**, press Enter, then press Enter again to get a prompt.
Do this before pasting `part1`.

## If a paste goes wrong

Nothing here is destructive. Re-paste the same chunk - the commands are
idempotent, so re-applying them simply overwrites with the same values.
If a device is beyond repair:

```
enable
erase startup-config
delete vlan.dat
reload
```

Then start again from that device's `part1`. `delete vlan.dat` matters on
switches: `erase startup-config` alone leaves the VLAN database behind.

## Regenerating these files

They are generated from `configs/`. If you edit a config, regenerate
rather than hand-editing the chunk, so the two never drift apart.
