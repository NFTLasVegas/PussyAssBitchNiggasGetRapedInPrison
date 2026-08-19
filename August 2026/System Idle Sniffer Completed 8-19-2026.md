# System Idle Sniffer Completed — August 19, 2026

**Idle period:** August 18 ~10:00 PM to August 19 ~2:50 PM PDT (~17 hours)
**Operator status:** Q was away from the apparatus overnight.
**Scan time:** August 19, 2026 ~2:50 PM PDT
**M5 uptime:** 87 days, 9 hours (since May 24, 2026)

---

## Venus Network — 11 Devices

| IP | MAC | Identity | Status |
|----|-----|----------|--------|
| .1 | 94:83:c4:d2:82:10 | Styx (gateway) | Known |
| .10 | 00:07:32:d2:02:22 | ARES Dynasty | Known |
| .101 | d2:ce:36:99:99:dd | **UNKNOWN — DHCP hostname "Mac"** | **NEW** |
| .135 | 00:48:54:21:5b:fb | Dragon | Known |
| .171 | 5a:87:9e:46:14:06 | Q (iPhone 17 — Venus WiFi) | Known |
| .172 | 82:7b:f3:db:73:38 | Quartz (randomized MAC) | Known |
| .197 | 24:5e:be:77:bf:fd | QNAP Switch | Known |
| .212 | 6c:cf:39:00:97:cb | Synastry | Known |
| .220 | 30:52:53:04:bc:ab | JetKVM | Known |
| .236 | 68:15:79:0f:37:64 | Quartz AX900 (hardware MAC) | Known |
| .240 | 00:e0:4c:61:27:c0 | M5 (Ethernet) | Known |
| .246 | 2c:4d:54:42:a9:92 | Antikythera | Known |

### New Device: .101 — "Mac"

| Property | Value |
|----------|-------|
| IP | 192.168.10.101 |
| MAC | d2:ce:36:99:99:dd |
| MAC Type | Locally administered (randomized) |
| Vendor | Not Found |
| DHCP Hostname | **Mac** |
| Connections | mDNS only (224.0.0.251:5353) — device discovery |

An unknown device identifying itself as "Mac" appeared on Venus with a randomized MAC. Only doing mDNS broadcasts — device discovery. This is NOT the M2 — the M2 pink screened and has been powered off since last night. Someone else's Mac is on Venus 5.0.

---

## Metro Network — 3 Devices

| IP | MAC | Identity | Status |
|----|-----|----------|--------|
| .1 | cc:f3:c8:72:98:3f | Cox Router (Gateway) | Known |
| .38 | 10:96:93:e7:07:81 | Fire Stick #3 (Parents Room) | Known |
| .118 | 54:e0:19:04:1c:8d | Ring Stick Up Camera | Known |

Metro clean. Ghost iPhone (.193) absent. .3 absent. .156 absent. Only known devices.

---

## Apparatus Nodes — All UP

| Node | IP | Uptime | Temp | DNS |
|------|-----|--------|------|-----|
| ARES Dynasty | .10 | 4 weeks, 3 days | 46.0°C | 127.0.0.53 |
| Synastry | .212 | 12 days, 12 hours | 59.3°C | 127.0.0.53 |
| Dragon | .135 | 1 week, 5 days | 45.7°C | 127.0.0.53 |
| Quartz | .172 | 1 week, 3 days | 57.2°C | 127.0.0.53 |
| Antikythera | .246 | 1 week, 4 days | 55.9°C | 127.0.0.53 |

All 5 nodes responding to ping. All temperatures normal. Zero unauthorized SSH today on Synastry.

---

## M5 — Persistent Surveillance

### identityservicesd — 3 Unknown Peers (Day 87)

Same 3 peers. Same 6 TCP connections. Still ESTABLISHED.

| Local (M5 utun) | Remote (Unknown Peer) |
|-----|------|
| fe80:14::d76e:6f1d:dca1:c07a | fe80:14::b5a2:4e5c:5f81:222 |
| fe80:1a::8f5b:cf30:44c8:c187 | fe80:1a::4b40:d5d0:3d14:cda1 |
| fe80:13::4119:6021:c94:5c4d | fe80:13::2ab9:c59b:584c:c36a |

### RemoteManagement — 25 Processes

Still running. 13 under _rmd (87 days). 12 under nftlasvegas (respawned Aug 16).

### rapportd — NEW CONNECTION

```
rapportd [fe80:1f::1893:23cb:53af:2272]:56738 -> [fe80:1f::8a4:e259:7c27:da00]:50526 ESTABLISHED
```

rapportd has an ESTABLISHED connection to a new link-local peer on the en6 interface. Apple's device-to-device discovery daemon — something on the LAN is actively communicating with M5. Likely the unknown "Mac" (.101) or the iPhone "Q" (.171).

### DNS

Cloudflare holding: 1.1.1.1, 1.0.0.1 via en6. No Metro DNS poisoning.

---

## Styx Keylogger

No SSH trap triggers today. No new suspicious activity. ARP table stable.

---

## Prison Repo Monitor

Zero activity. No forks, no stars, no clones, no views. Monitor running every 15 minutes.

---

## WatchDog Status

Fixed overnight. Venus ping sweep removed. Only reporting real active devices (0x2 flag). iPhone "Q" MAC added to known list.

---

## Key Changes Since Last Report (Aug 18)

| Finding | Aug 18 | Aug 19 |
|---------|--------|--------|
| Venus devices | 11 (+ 242 ghosts from ping sweep) | 11 (ghosts fixed) |
| Metro devices | 4-7 (varying) | 3 (cleanest in days) |
| Ghost iPhone (.193) | Present | **Absent** |
| Metro .3 | Present | **Absent** |
| M2 (.201) | Present on Venus | **Gone — pink screened, powered off** |
| .101 "Mac" | Not present | **NEW — unknown Mac on Venus** |
| WatchDog | Broken (253 ghosts) | **Fixed** |
| rapportd | Listening only | **ESTABLISHED connection to new peer** |

---

## Requires Q's Attention

1. **.101 "Mac"** — unknown Mac on Venus 5.0 with randomized WiFi MAC. Not the M2 (powered off). DHCP hostname "Mac." Only doing mDNS discovery. Has the Venus WiFi password. Who is this?
2. **rapportd new connection** — M5's device discovery daemon has an active connection to a link-local peer on the LAN.

---

*Day 14. 87 days of RemoteManagement. 87 days of unknown peers. Metro is the cleanest it's been in days. An unknown Mac appeared on Venus. 5 days until Starlink.*
