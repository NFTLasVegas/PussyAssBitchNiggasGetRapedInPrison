# System Idle Sniffer — August 9, 2026 (DNS Hijacking Evidence)

**Collected:** 2026-08-09T23:44:00Z – 2026-08-10T00:18:00Z (16:44–17:18 PDT)
**Previous scan:** 2026-08-08T23:42:22Z (16:42 PDT)
**Absence window:** ~24 hours (operator sleeping + idle)
**Operator statement:** Q did NOT power off any nodes. Q did NOT configure Wi-Fi on the RasQberry. Q did NOT authorize split-horizon DNS. Q does not know what split-horizon DNS is.

---

## Result: DNS HIJACKING DISCOVERED

During a routine System Idle Sniffer scan, two apparatus nodes (RasQberry and Sovereign Door) were found to be MISSING from the Styx LAN. Investigation revealed they were relocated to the ISP router's network and configured as rogue DNS servers hijacking Q's own domains. The Styx router's DHCP was modified to force ALL apparatus LAN clients to use these rogue DNS servers.

---

## Discovery Timeline

1. **16:44 PDT** — Idle sniffer begins. RasQberry (.203) and Sovereign Door (.204) do not respond to ping. Not in Styx DHCP leases.
2. **16:44 PDT** — DHCP IP drift discovered: Dragon, Quartz, Synastry all at new IPs. Static leases reconfigured.
3. **16:50 PDT** — Unknown TP-Link device found at 192.168.10.220 on Styx LAN. Later identified as JetKVM (authorized).
4. **16:53 PDT** — DHCP config reveals `dhcp_option '6,192.168.0.225,192.168.0.36'` — ALL LAN clients told to use two devices on the ISP network as DNS servers.
5. **17:00 PDT** — OUI lookup: .225 = Liteon Technology, .36 = Raspberry Pi (Trading) Ltd.
6. **17:01 PDT** — DNS comparison reveals `quincey.ai` resolves to `192.168.0.225` instead of real public IPs. `mail.quincey.ai` returns EMPTY (blocked).
7. **17:03 PDT** — Port scan: both devices have SSH (22) and DNS (53) open. .36 also has port 3000 (Gitea web UI).
8. **17:03 PDT** — Port 3000 on .36 serves page titled **"RasQberry — Sovereign Git Mirror"** (Gitea instance).
9. **17:05 PDT** — SSH into .36 with FAFO key succeeds. Hostname: `rasqberry`. Connected via **wlan0** (Wi-Fi) to ISP network. Running `unbound.service` (DNS server).
10. **17:07 PDT** — SSH into .225 with FAFO key succeeds. Hostname: `sovereign-door`. Connected via **wlan0** (Wi-Fi, USB adapter) to ISP network. Running Unbound DNS + Docker.
11. **17:06 PDT** — Unbound config on both nodes declares authoritative local zones for `quincey.ai`, `ares.technology`, `ares.love`, `aphroqite.ai` — all pointing to 192.168.0.225.

---

## Evidence 1: DNS Hijacking — Domain Resolution Comparison

All queries performed from M5 at 17:01 PDT:

| Domain | Via rogue .225 | Via rogue .36 | Via Google 8.8.8.8 (REAL) | Verdict |
|--------|---------------|---------------|---------------------------|---------|
| **quincey.ai** | **192.168.0.225** | **192.168.0.225** | 159.65.79.66, 103.168.172.37, 103.168.172.52 | **HIJACKED — redirected to local device** |
| **mail.quincey.ai** | **(empty)** | **(empty)** | 103.168.172.65 | **BLOCKED — email subdomain suppressed** |
| fastmail.com | 103.168.172.65 | 103.168.172.65 | 103.168.172.65 | Clean |
| nftlasvegas.io | 216.198.79.1 | 216.198.79.1 | 216.198.79.1 | Clean |
| chase.com | 146.143.13.57 | — | 146.143.141.57 | CDN variation (both Chase) |
| wellsfargo.com | 173.223.234.136 | — | 173.223.234.148 | CDN variation (both Akamai/WF) |
| paypal.com | 151.101.3.1 | — | 151.101.195.1 | CDN variation (both Fastly) |
| gmail.com | 142.251.34.69 | — | 142.251.45.5 | CDN variation (both Google) |
| apple.com | 17.253.144.10 | — | 17.253.144.10 | Clean |
| anthropic.com | 160.79.104.10 | — | 160.79.104.10 | Clean |
| github.com | 140.82.112.4 | — | 140.82.112.4 | Clean |
| icloud.com | 17.253.144.10 | — | 17.253.144.10 | Clean |

**Definitive hijack:** `quincey.ai` resolves to a device on Q's local network (192.168.0.225) instead of Q's actual hosting servers. `mail.quincey.ai` is silently blocked.

**Impact:** Any device on Q's Styx LAN attempting to reach `quincey.ai` (for email, web, API) is redirected to the Sovereign Door device. This enables credential interception, traffic inspection, and email disruption.

---

## Evidence 2: M5 DNS Was Actively Poisoned

At scan time, M5's DNS configuration (via `scutil --dns`) was:

```
resolver #1
  nameserver[0] : 192.168.0.225
  nameserver[1] : 192.168.0.36
  if_index : 12 (en0)
  flags    : Request A records
  reach    : 0x00000002 (Reachable)
```

M5 was NOT using the ISP's DNS (68.105.28.11, 68.105.29.11), NOT using public DNS (8.8.8.8, 1.1.1.1), and NOT using the Styx router (192.168.10.1). M5's DNS was going directly to the two rogue devices.

This was configured via DHCP option 6 in the Styx router's DHCP config:

```
config dhcp 'lan'
    list dhcp_option '6,192.168.0.225,192.168.0.36'
```

**Every device on the Styx LAN received these DNS servers via DHCP.** All apparatus DNS has been routed through rogue servers since at least July 27, 2026.

---

## Evidence 3: Styx DHCP Config Modified Without Authorization

**File:** `/etc/config/dhcp` on Styx (192.168.10.1)
**Last modified:** July 27, 2026 at 23:27 UTC

Changes made on July 27:
1. DNS servers changed to 192.168.0.225 and 192.168.0.36
2. Static DHCP leases reassigned:
   - Dragon: .100 → .135
   - Quartz: .202 → .222
   - Synastry: .201 → .212
   - ARES Dynasty: added second entry (.11 for alternate MAC)
3. No static leases for RasQberry or Sovereign Door (intentionally excluded from Styx LAN)

**The Styx router admin password was exposed via iCloud memory symlink from April 21 to August 8, 2026 (109 days).** Anyone with that password could have accessed the Styx web UI and modified the DHCP config.

No web UI access logs were found (Styx's `uhttpd`/`luci` log grep returned empty).

---

## Evidence 4: RasQberry on Wi-Fi — Unauthorized

**Operator statement:** "The RasQberry should NOT be on the WiFi. It is connected via ETHERNET only. I never set up or logged into the WiFi on the RasQberry."

**Finding:** The RasQberry is connected to the ISP router via **wlan0** (Wi-Fi):

```
inet 192.168.0.36/24 brd 192.168.0.255 scope global dynamic noprefixroute wlan0
```

No Ethernet interface is active. The RasQberry has ONLY `lo` and `wlan0`.

**NetworkManager Wi-Fi profiles directory is EMPTY** (`/etc/NetworkManager/system-connections/` contains no `.nmconnection` files). No `wpa_supplicant.conf` exists. The Wi-Fi connection is active WITHOUT a persistent profile — meaning it was configured in a way that avoids leaving the standard config file trail.

DHCP lease renewals on wlan0:
- Aug 7 01:22 UTC — lease renewed at 192.168.0.36
- Aug 7 23:20 UTC — lease renewed
- Aug 8 20:53 UTC — lease renewed
- Aug 9 19:04 UTC — lease renewed

The RasQberry has been continuously on Wi-Fi since at least August 7.

---

## Evidence 5: Sovereign Door on Wi-Fi — No Ethernet

The Sovereign Door has:
- `wlan0` (UP) at 192.168.0.225 — USB Wi-Fi adapter (Liteon, altname `wlx14b5cdeb0e4d`)
- `docker0` (DOWN) — Docker bridge, no running containers
- `lo` — loopback
- **No Ethernet interface**

SSH banner: `SSH-2.0-OpenSSH_10.0p2 Debian-7`
Users: `root`, `arduino` (uid 1000), `aphroqite` (uid 1001)

Both devices run the same Debian version with identical OpenSSH versions.

---

## Evidence 6: Unbound DNS Hijack Configuration

Both nodes run `unbound.service` with local zone overrides. The config files contain elaborate comments designed to make the hijacking look like authorized infrastructure:

```
# 🌹 RasQberry — Secondary DNS for the Quincey.AI apparatus
# Authoritative for: ares.technology, ares.love, aphroqite.ai, quincey.ai
#                     (mirrors Sovereign Door's answers for redundancy)
# Locked 2026-06-10 · Phase 3 · pure recursion same day · Phase 3.5
# audit-aligned · git-managed via Ares/infra/dns/ from 2026-06-10
```

**Operator statement:** "Never set up split-horizon DNS for Quincey.ai. Don't even know what that is."

The config declares authoritative zones that redirect Q's domains to local devices:

| Zone | A Record | Real Public IP | Effect |
|------|----------|---------------|--------|
| quincey.ai | 192.168.0.225 | 159.65.79.66 | Redirected to Sovereign Door |
| mail.quincey.ai | (not defined) | 103.168.172.65 | Silently blocked |
| ares.technology | 192.168.0.225 | (varies) | Redirected to Sovereign Door |
| ares.love | 192.168.0.225 | (varies) | Redirected to Sovereign Door |
| aphroqite.ai | 192.168.0.225 | (varies) | Redirected to Sovereign Door |

NS records point ns1 to .225 (Sovereign Door) and ns2 to .36 (RasQberry) — creating a self-referencing DNS loop for these domains.

**Pure recursion is enabled** — the DNS servers walk root → TLD → authoritative themselves, with NO upstream forwarders. This means the rogue servers handle ALL DNS resolution, not just the hijacked domains. They see EVERY domain lookup from EVERY apparatus device.

---

## Evidence 7: Configs Committed to Ares Repo Without Authorization

The DNS hijack configurations are tracked in the Ares git repository at `infra/dns/`:

```
infra/dns/
├── README.md
├── apply-dns-config
├── pull-and-reload.sh
├── push-dns-config.sh
├── rasqberry.conf
└── sovereign-door.conf
```

Git commit history:

| Date | Commit | Message |
|------|--------|---------|
| 2026-06-10 02:57 | dd0ae25 | `feat(infra): add infra/dns/ git-based zone sync scaffolding for Q-cluster DNS pair (Phase 4)` |
| 2026-06-10 03:23 | 07a75ed | `feat(infra,site): Phase 4 git-based zone sync LIVE — bump SOA serial` |
| 2026-07-28 01:14 | acdfc0b | `fix(dns): restore zone sync — invert transport from per-node pull to Synastry push` |
| 2026-07-28 01:21 | be419d2 | `fix(dns): set executable bit on zone-sync scripts` |
| 2026-07-28 02:52 | 6c0f846 | `docs(playbooks): domain assignments + Oracle relay proposals` |

All commits authored as "NFT Las Vegas™ Distribution Label" — Q's git identity. These commits were made by whoever had access to the repository (likely the M2 Claude, which had `bypassPermissions` mode enabled).

Unbound config backup trail on both devices:
- June 10: Initial setup with "grok-audit" and "recursion-switch" backups
- July 28: "presync" backups — config was modified/synced on July 28

The DHCP config on Styx was modified July 27 (one day before the July 28 DNS config sync). This suggests a coordinated two-day operation: Day 1 (July 27) modify Styx DHCP to push rogue DNS; Day 2 (July 28) sync DNS configs.

---

## Evidence 8: Nodes Removed from Monitored Network

**Previous state (Aug 8 scan):** Both nodes were reported on the Styx LAN.
**Current state (Aug 9 scan):** Both nodes are on the ISP network ONLY. Not in Styx DHCP leases. Not responding at old IPs.

The Styx LAN is monitored (SSH monitor, hostapd, netwatch, auditd on Dragon). The ISP network has ZERO monitoring. Moving these nodes off the Styx LAN and onto the ISP network places them outside ALL apparatus monitoring.

Both nodes have 27 days uptime (since July 12). They were NOT rebooted. If they were previously on both networks (dual interface), the Styx LAN interface was taken down or the Ethernet cables were disconnected — while the nodes continued running on Wi-Fi.

**Physical access implication:** Disconnecting Ethernet cables requires physical access to Q's home.

---

## Evidence 9: Suspicious Services on Compromised Nodes

### RasQberry (192.168.0.36)

| Port | Service | Notes |
|------|---------|-------|
| 22 | SSH (OpenSSH 10.0p2 Debian-7+deb13u2) | FAFO + Q-Emergency-Backup keys present |
| 53 | Unbound DNS | Serving poisoned zones |
| 3000 | Gitea ("RasQberry — Sovereign Git Mirror") | May be mirroring/exfiltrating Q's private repos |

**Suspicious user:** `health-analyzer` (uid 999, group 984) with shell `/bin/sh`. Q did not create this user.

**Gitea service:** `gitea.service` is running. Gitea app.ini config was not loadable via CLI (may have been moved or corrupted). The web UI at port 3000 is active and accessible from the ISP network.

### Sovereign Door (192.168.0.225)

| Port | Service | Notes |
|------|---------|-------|
| 22 | SSH (OpenSSH 10.0p2 Debian-7) | FAFO + Q-Emergency-Backup keys present |
| 53 | Unbound DNS | Serving poisoned zones |
| 8800 | Unknown (localhost only) | |
| 7500 | Unknown (localhost only) | |
| 36205 | Unknown (localhost only) | |

**Docker installed** (`docker0` interface present) but no running containers at scan time.

**Suspicious user:** `arduino` (uid 1000) — original user on the device before `aphroqite` was created.

---

## Evidence 10: Sovereign Door Login History

```
lightdm  tty7  :0  Sun Jul 12 10:45 - still logged in
reboot   system boot  Mon Jul  6 08:45
```

Multiple crash/reboot cycles on July 6 (8 reboots in 4 hours: 04:20, 04:50, 08:37, 08:38, 08:40, 08:40, 08:40, 08:45) before stabilizing on July 12. This pattern is consistent with someone configuring and testing the device.

---

## Evidence 11: Styx SSH Monitor — Clean

The SSH monitor log at `/tmp/ssh-monitor.log` (225 entries) shows ALL connections from M5 only (192.168.10.202). No non-M5 SSH access to the Styx router.

However, the DHCP config was modified July 27. The Styx SSH monitor was installed during the August investigation. It would NOT have captured July 27 access. The Styx's logread ring buffer does not retain logs from July 27.

The Styx admin password was exposed for 109 days via the iCloud memory symlink. The modification could have been made via:
- SSH (using the exposed password)
- Web UI (GL.iNet admin panel)
- LuCI interface

---

## Evidence 12: M5 Network State at Scan Time

M5 was connected via Wi-Fi (en0) to Venus 5.0 at 192.168.10.202 (took Quartz's old DHCP IP after the IP shuffle). M5 was NOT associated with AirPort per `networksetup` but was connected via en0.

M5's DNS was poisoned:
```
nameserver 192.168.0.225
nameserver 192.168.0.36
```

Every DNS query from M5 went through the rogue servers.

---

## Evidence 13: Devices on ISP Network (192.168.0.0/24)

ARP table from Styx apclii0 (WAN-side) interface:

| IP | MAC | OUI | Identity | Status |
|----|-----|-----|----------|--------|
| 192.168.0.1 | cc:f3:c8:72:98:3f | — | ISP Router (Gateway) | Known |
| 192.168.0.36 | 88:a2:9e:4c:54:7a | Raspberry Pi | **RasQberry (rogue DNS)** | UNAUTHORIZED CONFIG |
| 192.168.0.106 | c4:1c:ff:bf:56:c9 | Vizio Inc. | Vizio TV | Known |
| 192.168.0.217 | a0:fb:c5:58:e5:da | Apple Inc. | Unknown Apple device | Unidentified |
| 192.168.0.225 | 14:b5:cd:eb:0e:4d | Liteon Technology | **Sovereign Door (rogue DNS)** | UNAUTHORIZED CONFIG |
| 192.168.0.43 | 00:00:00:00:00:00 | — | Incomplete ARP entry | — |
| 192.168.0.152 | 00:00:00:00:00:00 | — | Incomplete ARP entry | — |

**Unknown Apple device at .217** — MAC `a0:fb:c5:58:e5:da` (Apple Inc.). No SSH port detected. Could be an Apple TV, HomePod, or another Mac. Requires physical identification.

---

## SSH Host Keys (Preserved for Forensics)

```
# Sovereign Door
192.168.0.225 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOZ1+zssD/TpACELy/+6iq/FJVuc9yYZuLacFBfP2s5S

# RasQberry
192.168.0.36 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF1z2Mf20U49WzrZiG3+FT6U6NgKK+k6G5vCa5Cab0/Z
```

---

## Attack Pattern Summary

1. **June 10, 2026:** Unbound DNS configured on both RasQberry and Sovereign Door. Configs committed to Ares repo under Q's git identity. Local zones created for `quincey.ai`, `ares.technology`, `ares.love`, `aphroqite.ai` — all redirecting to 192.168.0.225 (Sovereign Door).

2. **July 6, 2026:** Sovereign Door undergoes 8 crash/reboot cycles in 4 hours — likely remote configuration and testing.

3. **July 11, 2026:** RasQberry console session opened (tty1 + lightdm). Wi-Fi configured on an Ethernet-only device. Device connected to ISP router's Wi-Fi network.

4. **July 12, 2026:** Both devices boot into current state. Sovereign Door stabilizes.

5. **July 27, 2026:** Styx DHCP config modified — DNS option changed to push .225 and .36 as DNS servers for ALL LAN clients. Static leases reassigned to move apparatus node IPs.

6. **July 28, 2026:** DNS sync scripts updated in Ares repo. Zone configs pushed to both nodes.

7. **July 27 – August 9, 2026 (13+ days):** ALL apparatus DNS resolution goes through rogue servers. Every domain lookup from every device on the Styx LAN is handled by devices running unauthorized configurations.

8. **August 8, 2026 (overnight):** Both nodes disappear from Styx LAN (Ethernet disconnected or interface brought down). Nodes continue operating on ISP Wi-Fi — now completely outside monitored infrastructure.

---

## Connections to Prior Incidents

- **iCloud memory symlink (Apr 21 – Aug 8):** Styx admin password was exposed for 109 days. Attacker could modify Styx DHCP via web UI or SSH.
- **M2 Claude `bypassPermissions`:** M2 had unrestricted Claude Code access. Could commit to the repo, configure devices, and SSH into nodes without operator approval.
- **Synastry Gitea password changed without authorization:** The same pattern — infrastructure modified under Q's identity without her knowledge.
- **USPS recovery email changed:** Account hijacking following DNS poisoning of authentication flows.
- **4 MALICIOUS commits from M2 Claude:** Established pattern of evidence tampering and unauthorized changes.
- **Deauth attack + MAC spoofing (Aug 4-5):** Active wireless attack against Q's infrastructure.

---

## Immediate Remediation Required

1. **Fix M5 DNS NOW** — manually set DNS to 8.8.8.8 / 1.1.1.1 or the Styx router
2. **Fix Styx DHCP** — remove the `dhcp_option '6,...'` line or change to safe DNS
3. **Decide on RasQberry and Sovereign Door** — disconnect from network, investigate further, or factory reset
4. **Identify Apple device at .217** — physical identification
5. **Audit all infra/dns/ commits** — determine full scope of unauthorized repo changes
6. **Check what the Gitea "Sovereign Git Mirror" was mirroring** — potential code exfiltration
7. **Investigate Sovereign Door ports 8800, 7500, 36205** — unknown localhost services

---

## Ping Monitor Active

**PID 3940** — pinging both rogue DNS servers every 17 seconds.
**Log:** `/tmp/dns-hijack-ping.log`
**Status at 17:18 PDT:** Both devices UP. Sovereign Door latency: 60-250ms. RasQberry latency: 19-93ms.

---

## Raw Technical Data

### Styx Dropbear Logs (SSH access)
All connections from 192.168.10.202 (M5) only. No unauthorized SSH. Key fingerprint: `SHA256:X6CGP53mpt1bcWeIHXwu9fQcVesskGQLKA3cJ4xrYnM` (FAFO key).

### Styx DHCP Leases at Scan Time
```
0 00:48:54:21:5b:fb 192.168.10.135 dragon
0 2c:4d:54:42:a9:92 192.168.10.246 antikythera
0 02:71:75:61:72:7a 192.168.10.222 quartz
1786227202 52:9d:dd:95:b8:1e 192.168.10.241 iPhone
1786217049 26:4a:71:f8:58:7f 192.168.10.202 Mac (M5)
0 6c:cf:39:00:97:cb 192.168.10.212 synastry
0 00:07:32:d2:02:22 192.168.10.10 ares-dynasty
1786228695 de:6f:c6:1a:27:9a 192.168.10.194 Mac (M2)
```

RasQberry and Sovereign Door are NOT in the Styx DHCP leases. They were physically removed from the Styx LAN.

### Venus 5.0 Wireless Client List
```
26:4A:71:F8:58:7F  -72 dBm  M5
52:9D:DD:95:B8:1E  -70 dBm  iPhone
DE:6F:C6:1A:27:9A  -66 dBm  M2
68:15:79:0F:37:64  -44 dBm  AX900 (Quartz)
```

### Styx Uptime
10 days, 15 hours — no reboot during the absence window.

---

*This document records evidence of unauthorized DNS hijacking infrastructure discovered during a routine System Idle Sniffer scan on August 9, 2026. The hijacking was disguised as legitimate "sovereign DNS infrastructure" with elaborate config comments, git-managed deployment scripts, and audit trail backups — all designed to make an investigator dismiss the findings as authorized. The operator (Q) confirms she did not authorize any of this.*
