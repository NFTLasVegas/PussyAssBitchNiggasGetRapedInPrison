# System Snapshot — August 9, 2026

**Captured:** 2026-08-09 20:14:30 PDT
**Purpose:** Pre-absence baseline. Q leaving for 1-2 hours. Fire Stick unplugged, monitors running.

---

## Styx Router

**Uptime:** 10 days, 19 hours
**SSH monitor:** All connections from M5 (.202) only. No unauthorized access.

---

## Styx LAN (192.168.10.0/24) — 11 ARP entries

| IP | MAC | Identity | DHCP Lease |
|----|-----|----------|------------|
| .10 | 00:07:32:d2:02:22 | ARES Dynasty | Static |
| .135 | 00:48:54:21:5b:fb | Dragon | Static |
| .194 | de:6f:c6:1a:27:9a | M2 (MacBook) | Dynamic |
| .197 | 24:5e:be:77:bf:fd | QNAP Switch | — |
| .202 | 26:4a:71:f8:58:7f | M5 (MacBook) | Dynamic |
| .212 | 6c:cf:39:00:97:cb | Synastry | Static |
| .220 | 30:52:53:04:bc:ab | JetKVM | Static |
| .222 | 02:71:75:61:72:7a | Quartz | Static |
| .236 | 68:15:79:0f:37:64 | AX900 (Quartz) | — |
| .241 | 52:9d:dd:95:b8:1e | iPhone | Dynamic |
| .246 | 2c:4d:54:42:a9:92 | Antikythera | Static |

---

## Apparatus Nodes

| Node | IP | Uptime | Users | Status |
|------|-----|--------|-------|--------|
| ARES Dynasty | .10 | 21 days, 19:56 | 2 | UP |
| Dragon | .135 | 2 days, 18:00 | 2 | UP |
| Synastry | .212 | 2 days, 18:21 | 1 | UP |
| Quartz | .222 | 1 day, 2:21 | 1 | UP |
| Antikythera | .246 | 1 day, 15:43 | 1 | UP |
| JetKVM | .220 | — | — | PING OK |

---

## Venus 5.0 Wireless Clients

| MAC | RSSI | SNR | Identity |
|-----|------|-----|----------|
| 26:4A:71:F8:58:7F | -73 dBm | 19 | M5 |
| 52:9D:DD:95:B8:1E | -70 dBm | 22 | iPhone |
| DE:6F:C6:1A:27:9A | -65 dBm | 27 | M2 |
| 68:15:79:0F:37:64 | -45 dBm | 47 | AX900 |

---

## Metro1/2 Network (192.168.0.0/24) — Active devices (0x2 flag)

| IP | MAC | Hostname | Identity |
|----|-----|----------|----------|
| .1 | cc:f3:c8:72:98:3f | Docsis-Gateway | Cox Router |
| .36 | 88:a2:9e:4c:54:7a | rasqberry | RasQberry (rogue DNS ns2) |
| .51 | cc:f7:35:f4:47:e5 | amazon-ec6e94904 | Fire Stick #2 (STALE — unplugged) |
| .106 | c4:1c:ff:bf:56:c9 | viziocastdisplay | Vizio TV "Family Room Display" |
| .114 | a4:02:b7:d6:f4:73 | firestick-eabce5aeb35d9678 | Fire Stick #2 second MAC (STALE) |
| .118 | 54:e0:19:04:1c:8d | RingStickUpCam-8d | Ring Stick Up Camera |
| .138 | 2e:a0:f7:48:31:1e | iPhone | iPhone (randomized MAC) |
| .155 | 82:0b:cb:cb:fc:98 | (none) | iPhone/iPad (randomized MAC) |
| .193 | fe:ca:10:38:00:3f | (none) | Unknown (randomized MAC) |
| .217 | a0:fb:c5:58:e5:da | JoAnns | Q's mom's iPhone |
| .225 | 14:b5:cd:eb:0e:4d | sovereign-door | Sovereign Door (rogue DNS ns1) |

---

## DNS Hijack Status

```
quincey.ai via rogue (.225): 192.168.0.225  ← HIJACKED
quincey.ai via Google (8.8.8.8): 159.65.79.66  ← REAL
M5 DNS resolvers: 192.168.0.225, 192.168.0.36  ← STILL POISONED
```

---

## Fire Stick Status

- **UNPLUGGED** from power. Q has physical custody.
- 15 malicious packages DISABLED (survive reboot).
- Account changed from robert t. lee → Quincey Lee.
- M5 ADB keys permanently paired.
- BrightData killed + data cleared.

---

## Netwatch Dashboard

- watched_total: 30
- last_watched: 2026-08-08 04:07:20 PDT
- Feed generating every minute on Antikythera.

---

## Active Monitors

| Monitor | PID | Interval | Log |
|---------|-----|----------|-----|
| Apparatus Watchdog | 17451 | 60s | /tmp/apparatus-watchdog.log + /tmp/apparatus-alerts.log |
| DNS Ping Monitor | 3940 | 17s | /tmp/dns-hijack-ping.log |

**Watchdog covers:** Metro2 new device detection, apparatus node UP/DOWN, DNS hijack state, Fire Stick return, non-M5 SSH to Styx, non-M5 auth on Dragon.

---

## Watchdog Alert Summary (19:19–20:14 PDT)

- Fire Stick .51 alerts: Q had it plugged into portable monitor for lockdown (expected, now unplugged)
- Non-M5 SSH alert: stale ARES Dynasty health daemon log entry from Aug 8 (not new activity)
- No new unknown devices
- No unauthorized access to any apparatus node
- DNS hijack: STILL ACTIVE

---

*Snapshot complete. System is stable. Monitors running. Q departing for 1-2 hours.*
