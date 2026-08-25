# PC and Server Setup — Exact Click-by-Click Steps

Project **R075458**. Everything on this page is done in the Packet Tracer
GUI, not the CLI.

---

## 1. All 36 PCs — switch them to DHCP

Do this once per PC. It is identical for every PC in every department.

1. Click the PC icon on the workspace.
2. Go to the **Desktop** tab.
3. Click **IP Configuration**.
4. In the *Interface* dropdown leave `FastEthernet0` selected.
5. Select the **DHCP** radio button (top of the IPv4 Configuration box).
6. Wait 2–5 seconds. The status line must change to
   **`DHCP request successful.`**
7. Confirm the four fields have filled themselves in:

   | Field | Expected value (example: `PC-CSE1`) |
   |-------|--------------------------------------|
   | IPv4 Address | `192.168.20.11` |
   | Subnet Mask | `255.255.255.0` |
   | Default Gateway | `192.168.20.1` |
   | DNS Server | `192.168.80.10` |

8. Close the window. **Do not** touch the IPv6 section — this project is
   IPv4 only.

> If it says **`DHCP request failed`**, jump to
> [troubleshooting.md](troubleshooting.md) → *Symptom 3*.

### Quick check from the PC itself

**Desktop → Command Prompt**:

```
ipconfig /all
```

Expect the address, mask, gateway and DNS server above.
To force a fresh lease at any time:

```
ipconfig /release
ipconfig /renew
```

---

## 2. The three servers — static addressing

Servers must **never** be DHCP clients here: their addresses are baked
into the DNS records, the ACLs and the DHCP `dns-server` option.

### 2.1 `SRV-DNS`

**Desktop → IP Configuration → Static**

| Field | Value |
|-------|-------|
| IPv4 Address | `192.168.80.10` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.80.1` |
| DNS Server | `192.168.80.10` |

### 2.2 `SRV-WEB`

**Desktop → IP Configuration → Static**

| Field | Value |
|-------|-------|
| IPv4 Address | `192.168.80.20` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.80.1` |
| DNS Server | `192.168.80.10` |

### 2.3 `SRV-APP`

**Desktop → IP Configuration → Static**

| Field | Value |
|-------|-------|
| IPv4 Address | `192.168.80.30` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.80.1` |
| DNS Server | `192.168.80.10` |

---

## 3. `SRV-DNS` — turn on the DNS service and add the records

1. Click `SRV-DNS` → **Services** tab → **DNS** in the left-hand list.
2. Set **DNS Service** to **On**.
3. Add each record below: type the *Name*, choose the *Type*, type the
   *Address*, then press **Add**. Repeat for all five.

| Name | Type | Address |
|------|------|---------|
| `www.university.local` | A Record | `192.168.80.20` |
| `dns.university.local` | A Record | `192.168.80.10` |
| `portal.university.local` | A Record | `192.168.80.30` |
| `university.local` | A Record | `192.168.80.20` |
| `ftp.university.local` | A Record | `192.168.80.30` |

4. Leave **DNS Cache** and **DNS Recursive** at their defaults.
5. Turn **every other service off** on this server (HTTP, FTP, EMAIL,
   DHCP…). Two DHCP servers on one network is the classic self-inflicted
   outage — the router is already the DHCP server.

> **Critical:** the `DHCP` service on all three servers must be **Off**.
> The router owns DHCP.

### Verify

From any PC → **Desktop → Command Prompt**:

```
nslookup www.university.local
```

Expected:

```
Server:  [192.168.80.10]
Address: 192.168.80.10

Name:    www.university.local
Address: 192.168.80.20
```

---

## 4. `SRV-WEB` — turn on HTTP and load the portal page

1. Click `SRV-WEB` → **Services** tab → **HTTP**.
2. Set **HTTP** to **On**. Set **HTTPS** to **On** as well (the ACL
   already permits port 443).
3. In the file list find **`index.html`** and click **(edit)**.
4. Select every line of the default Cisco page and delete it.
5. Open `web/index.html` from this repository, copy the **entire** file,
   and paste it into the editor.
6. Click **Save**.
7. Leave `helloworld.html`, `image.html`, `copyrights.html` alone — they
   do no harm.

## 5. `SRV-APP` — the application / file portal

1. Click `SRV-APP` → **Services** tab → **HTTP** → set **HTTP** to
   **On**.
2. Edit `index.html` and paste the contents of `web/portal.html` from
   this repository.
3. Still on `SRV-APP`, go to **Services → FTP**:
   - Set **Service** to **On**.
   - Add a user: Username `student`, Password `Student@2025`, tick
     **Write**, **Read**, **List**, then press **Add**.
   - This gives the "Application / File server" role something real to
     demonstrate, and ACL 120 already permits TCP 20 and 21 to
     `192.168.80.30`.

---

## 6. Full end-device checklist

| Device | Method | IP | Services to enable | Services to force OFF |
|--------|--------|----|--------------------|-----------------------|
| PC-ADM1–4, PC-CSE1–6, PC-ECE1–6, PC-MECH1–4, PC-LIB1–4, PC-FAC1–4, PC-LAB1–8 | DHCP | auto `.11+` | none | none |
| SRV-DNS | Static | 192.168.80.10 | DNS | HTTP, DHCP, FTP, EMAIL |
| SRV-WEB | Static | 192.168.80.20 | HTTP, HTTPS | DNS, DHCP, FTP, EMAIL |
| SRV-APP | Static | 192.168.80.30 | HTTP, FTP | DNS, DHCP, EMAIL |
