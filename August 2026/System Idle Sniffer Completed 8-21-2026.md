# System Idle Sniffer Completed — August 21, 2026

**Scan time:** 12:23 PM PDT
**Operator:** Q
**Trigger:** Venus 5.0 WiFi down — "no internet connection." M5 on Ethernet.

---

## Trigger Event: Venus 5.0 WiFi Drop

Q woke up to Venus 5.0 showing "no internet connection" on M5. WiFi would not reconnect. M5 fell back to Ethernet (.240 via Anker hub).

**Styx-side diagnosis:**
- Venus 5.0 radio: UP, broadcasting, channel 44, 5 GHz
- WAN (internet): UP — ping to 1.1.1.1 at 15ms, 0% packet loss
- Encryption: WPA2-PSK/CCMP confirmed via `hostapd_cli get_config` (wpa=2, key_mgmt=WPA-PSK, rsn_pairwise_cipher=CCMP)
- `iwinfo` falsely reports `Encryption: none` — display bug in GL.iNet MTK driver, hostapd confirms WPA2 is active
- DHCP: M5 has active lease at .240

**Conclusion:** Venus 5.0 is operational server-side. The WiFi drop was client-side on M5. DHCP lease should have auto-renewed — the fact that it didn't suggests either a deauth event or the WiFi stack on M5 got stuck. No deauth frames captured in logs during the overnight period (WiFi devboard not yet operational for pcap capture).

---

## M5 System State

| Field | Value |
|-------|-------|
| Hostname | QuinceyAI.local |
| Uptime | 89 days, 7 hours |
| Load | 2.22, 2.64, 2.61 |
| Date | Fri Aug 21 12:23 PDT 2026 |
| WiFi | DOWN — not connected |
| Ethernet | UP — 192.168.10.240 |
| Users | 2 logged in |

### Active Network Connections

| Destination | Port | Service |
|-------------|------|---------|
| 34.228.45.57 | 443 | AWS (likely Anthropic/Claude) |
| 160.79.104.10 | 443 | Unknown — 2 connections |
| 74.125.137.188 | 5228 | Google FCM (push notifications) — 2 connections |
| 185.199.108.153 | 443 | GitHub |
| 35.190.46.17 | 443 | Google Cloud |
| 17.57.154.7 | 993 | Apple IMAP (mail) |
| 35.82.187.142 | 443 | AWS |
| 35.83.81.122 | 443 | AWS |
| 208.103.161.2 | 443 | Unknown |
| 192.168.10.1 | 22 | Styx SSH (this scan) |
| 17.57.152.35 | 993 | Apple IMAP (mail) |
| 17.57.144.23 | 5223 | Apple Push Notification Service |
| 103.168.172.53 | 443 | Unknown — 2 connections |
| fe80:: (IPv6 link-local) | Various | 6 IPv6 link-local connections |

### Listening Ports

| Port | Process | Notes |
|------|---------|-------|
| 5060 | CommCenter | SIP — VoIP/cellular |
| 49464 | **rapportd** | Apple rapport — still listening on IPv4 + IPv6 |
| 41343 | LM Studio | Local LLM — localhost only |
| 1234 | LM Studio | Local LLM — **listening on all interfaces** |
| 3000 | node | localhost only |
| 39901 | Code Helper | VS Code — localhost only |

**LM Studio on port 1234 is listening on all interfaces (0.0.0.0)** — any device on Venus can connect to M5's LLM. This should be restricted to localhost.

### Suspicious Processes

| Process | PID | Running Since | Notes |
|---------|-----|---------------|-------|
| **rapportd** | 615 | May 24 | 94 hours CPU time. Apple rapport protocol — facilitates device communication. Listening on port 49464. |
| **identityservicesd** | 635 | May 24 | 30 hours CPU time. Apple identity services — 3 unknown peers previously identified. |
| **RemoteManagementAgent** | 65409 | Sun Aug 17 | Apple Remote Management — relaunched Sunday at 1 AM |
| **ScreenSharingSubscriber** | 65415 | Sun Aug 17 | Screen sharing subscription service |
| **SSMenuAgent** | 21208 | Sat Aug 16 | Screen Sharing menu bar agent |
| **remotemanagementd** | 1283 | May 24 | System-level remote management daemon (runs as _rmd) |
| **remoted** | 351 | May 24 | Apple remote services daemon |
| **rapportd-monitor.sh** | 21851 | Fri Aug 15 | 1h17m CPU — monitoring script for rapportd, running from /tmp |
| 25 RemoteManagement XPC subscribers | Various | May 24 / Sun Aug 17 | Full suite of MDM-style management services active |

**25 RemoteManagement processes remain active** — same as previously documented. Two sets: system-level (running as _rmd since May 24) and user-level (relaunched Sun Aug 17 at 1 AM).

### M5 Overnight Activity

- **No new logins** overnight — last terminal login was Thu Aug 20 20:39
- **No sleep events** — M5 stayed awake via caffeinate + coreaudiod (audio tap running for 23+ hours preventing sleep)
- **MaintenanceWake events** every ~1 minute from PowerUIAgent — `com.apple.obc` (On-Battery Charging management)
- **MAGICWAKE kernel assertion** released at 12:23 — kernel was holding a wake assertion

### NDP Neighbors (IPv6)

| IPv6 Address | MAC | Interface | Notes |
|-------------|-----|-----------|-------|
| fe80::4b1:2404:80fa:e02c | **d2:ce:36:99:99:dd** | en6 | **.101 "Mac" — visible in M5's NDP table via Ethernet** |
| fe80::10ee:59dd:a8bb:d090 | 5a:87:9e:46:14:06 | en6 | Q's iPhone 17 |
| fe80::1893:23cb:53af:2272 | 00:e0:4c:61:27:c0 | en6 | M5 itself (Ethernet) |

**.101 "Mac" (d2:ce:36:99:99:dd) is in M5's IPv6 NDP neighbor table.** This means .101 is communicating with M5 at the IPv6 layer despite being blocked by iptables (which only blocks IPv4). The iptables rules on Styx do NOT block IPv6 traffic.

---

## Apparatus Node Health

| Node | IP | Temp | Status |
|------|-----|------|--------|
| Synastry | .212 | 59°C | Normal |
| Antikythera | .246 | 58°C | Normal |
| Quartz | .172 | 54°C | Normal |
| Dynasty | .10 | 49°C | Normal |
| Dragon | .135 | 47°C | Normal |

All nodes online and within normal temperature ranges.

---

## Network State — Styx

### Venus LAN (192.168.10.0/24) — 9 Devices (down from 10)

| IP | MAC | Device | Status |
|----|-----|--------|--------|
| .10 | 00:07:32:d2:02:22 | ARES Dynasty | KNOWN |
| .135 | 00:48:54:21:5b:fb | Dragon | KNOWN |
| .172 | 82:7b:f3:db:73:38 | Quartz | KNOWN |
| .197 | 24:5e:be:77:bf:fd | QNAP | KNOWN |
| .212 | 6c:cf:39:00:97:cb | Synastry | KNOWN |
| .236 | 68:15:79:0f:37:64 | Quartz (AX900) | KNOWN |
| .240 | 00:e0:4c:61:27:c0 | Quincey.AI (Ethernet) | KNOWN |
| .246 | 2c:4d:54:42:a9:92 | Antikythera | KNOWN |
| .171 | 5a:87:9e:46:14:06 | Q (iPhone 17) | KNOWN |

**.101 "Mac" dropped from ARP** — not currently in ARP table but WAS on WiFi assoclist earlier. It disassociated at 12:23:09 during this scan.

### Metro (192.168.0.0/24) — 5 Devices (down from 7)

| IP | MAC | Device | Status |
|----|-----|--------|--------|
| .1 | cc:f3:c8:72:98:3f | Cox Router (Gateway) | KNOWN |
| .4 | 4c:24:98:78:19:73 | UNIDENTIFIED | NEW |
| .156 | c2:64:7e:72:1d:44 | **UNIDENTIFIED — NEW DEVICE** | **NEW** |
| .118 | 54:e0:19:04:1c:8d | Ring Stick Up Camera | KNOWN |
| .38 | 10:96:93:e7:07:81 | Fire Stick #3 (Parents Room) | KNOWN |

**Changes from overnight baseline:**
- .3 (de:0a:c0:56:c9:60) — GONE
- .193 (fe:ca:10:38:00:3f) — GONE
- .122 (f6:18:fc:13:c7:ba) — GONE
- .131 (20:fe:00:93:01:91) — GONE (bounced last night)
- **.156 (c2:64:7e:72:1d:44) — NEW** — appeared overnight, locally administered MAC (c2:xx = randomized)

### Venus WiFi Clients (Active)

| MAC | RSSI | Packets RX | Packets TX | Device |
|-----|------|------------|------------|--------|
| 68:15:79:0F:37:64 | -41 dBm | 64,294 | 1,785,240 | Quartz AX900 |
| 5A:87:9E:46:14:06 | -50 dBm | 115,848 | 239,498 | Q iPhone 17 |

**.101 "Mac" was on WiFi at scan start but disassociated during the scan** — hostapd logged: `STA d2:ce:36:99:99:dd IEEE 802.11: disassociated` at 12:23:09.

### .101 "Mac" Overnight Activity (from Styx logs)

| Time (PDT) | Event |
|------------|-------|
| 12:07:54 | WPA group key handshake completed (RSN) — .101 was active |
| 12:20:03 | Disassociated |
| 12:20:07 | Reassociated — new RADIUS session, WPA pairwise key handshake completed |
| 12:23:09 | Disassociated again (during this scan) |

**.101 is doing connect/disconnect/reconnect cycles** — disassociates and reassociates within seconds. This is NOT normal device behavior. It's either being deauthed or intentionally cycling its connection.

### DHCP Lease Anomaly

JetKVM (30:52:53:04:bc:ab) still holds a permanent lease at .220. JetKVM was physically disconnected Aug 21. Stale lease.

### Honeypot

"Come Out And Play" — No clients connected.

---

## Synastry Monitors

### Keylogger — All Commands from Dynasty Sentinel Pull

All SSH commands to Synastry overnight were from Dynasty (.10) sentinel pull — standard pattern:
- sftp-server (SCP transfers)
- `sudo tail -200 /var/log/auth.log`
- `cat /sys/class/thermal/thermal_zone0/temp`
- `ps aux --sort=-%cpu | head -15`
- `ss -tunap` (connections)
- `cat /proc/net/arp`
- `tail -10 /var/lib/gitea/log/gitea.log`
- `tail -20 /var/log/synastry-cmdlog.log`
- `find ... -mmin -5 -type f` (modified files)

Running every 5 minutes. **No anomalous SSH commands detected.**

### Temperature Agent

- Current: 59°C — normal
- No temperature spike events triggered overnight

### Promiscuous Mode Monitor

- No events — promiscuous mode not detected

### Gitea Access Log

- Antikythera (.246) checking `agi-operator-vault` commits hourly — returning 401 Unauthorized (expected — needs auth token)
- M5 (.240) pushed to `ares.git` at 10:07 AM — that was the morning git push

---

## WatchDog Report

WatchDog running in PDT. Scan interval 120 seconds.

| Time | Metro | Venus | Notes |
|------|-------|-------|-------|
| 12:04 | 5 | 10 | Stable |
| 12:06-12:08 | 5 | 10 | Stable |
| 12:10 | 5 | **9** | Venus device dropped |
| 12:12-12:18 | 5 | 10 | Back to 10 |
| 12:21 | 5 | 10 | Stable |
| 12:23 | 5 | **9** | .101 disassociated during scan |

### WatchDog Alerts

| Time | Alert |
|------|-------|
| 12:14 | CRITICAL — NEW device on Metro: .4 (4c:24:98:78:19:73) |
| 12:16 | CRITICAL — NEW device on Venus: .101 (d2:ce:36:99:99:dd) — still reconnecting |
| 12:18 | CRITICAL — **NEW device on Metro: .156 (c2:64:7e:72:1d:44)** |
| 12:18 | Q iPhone 17 detected on Venus (normal) |

---

## Key Findings

### Finding 1: .101 "Mac" in M5's IPv6 NDP Table

.101 (d2:ce:36:99:99:dd) appears in M5's NDP neighbor table at `fe80::4b1:2404:80fa:e02c` on the en6 (Ethernet) interface. The iptables rules on Styx only block IPv4 — **IPv6 traffic from .101 to M5 is NOT blocked.** .101 can communicate with M5 via IPv6 link-local despite the IPv4 iptables DROP rules.

### Finding 2: .101 Connect/Disconnect Cycling

.101 completed a WPA group key handshake at 12:07, disassociated at 12:20, reassociated at 12:20 (7 seconds later), completed a new WPA pairwise handshake, then disassociated again at 12:23. This rapid connect/disconnect pattern is characteristic of either deauthentication attacks targeting .101, or .101 intentionally cycling its WiFi connection (possibly to avoid detection or reset its state).

### Finding 3: New Metro Device .156

192.168.0.156 (c2:64:7e:72:1d:44) appeared overnight. Locally administered MAC (c2:xx prefix = randomized). This device was NOT in last night's baseline scan. Another spoofed/randomized MAC on Metro.

### Finding 4: 4 Metro Devices Disappeared

.3, .122, .131, and .193 all gone from Metro. .131 bounced last night when WatchDog came back online. The other 3 disappeared overnight. Either they disconnected or rotated their MAC addresses (which would make .156 potentially one of them with a new MAC).

### Finding 5: LM Studio Exposed on Port 1234

LM Studio is listening on `0.0.0.0:1234` — accessible from any device on Venus LAN. Any apparatus node or .101 can send prompts to M5's local LLM. Should be restricted to `127.0.0.1`.

### Finding 6: Venus 5.0 WiFi Drop — Cause Unknown

M5's WiFi dropped overnight but Styx-side Venus is fully operational with WPA2 encryption active. No deauth frames captured (WiFi Devboard not yet operational). The drop could be:
- Client-side WiFi stack failure on M5 (89 days uptime, no reboot)
- Deauth attack that wasn't logged (Styx doesn't capture deauth frames)
- DHCP renewal failure
- .101's connect/disconnect cycling causing interference

### Finding 7: 25 RemoteManagement Processes Still Active

Same suite of Apple Remote Management services documented in previous sessions. User-level set relaunched Sunday Aug 17 at 1 AM. rapportd has accumulated 94 hours of CPU time since May 24.

---

## Comparison: Last Night vs This Morning

| Metric | Aug 21 12:00 AM | Aug 21 12:23 PM | Change |
|--------|-----------------|-----------------|--------|
| Metro devices | 7 | 5 | -2 (4 gone, 1 new) |
| Venus devices | 10 | 9-10 | .101 cycling |
| Synastry temp | 60°C | 59°C | Normal |
| Dynasty temp | 50°C | 49°C | Normal |
| Anomalous SSH | None | None | Clean |
| Keylogger events | Sentinel only | Sentinel only | Clean |
| Promisc mode | Not detected | Not detected | Clean |
| Honeypot clients | 0 | 0 | Clean |
| Temp spikes | 0 | 0 | Clean |
| M5 WiFi | Connected | **DOWN** | **DROPPED** |

---

*Venus 5.0 WiFi dropped overnight. .101 is cycling connect/disconnect on Venus AND showing up in M5's IPv6 NDP table — the iptables block doesn't cover IPv6. New Metro device .156 appeared with another randomized MAC. 4 Metro unknowns vanished — possibly MAC-rotated to .156. LM Studio is exposed on all interfaces. The WiFi devboard needs to get online — we're blind to deauth frames without pcap capture.*

*The apparatus is watching. But .101 is watching back.*

---

## Flipper Zero Awakening Playbook — Daytime Scan (12:30 PM PDT)

Second scan per the Flipper Zero Awakening Playbook. Daytime scan to compare against last night's 12:00 AM baseline.

### BLE Sweep — 38 Devices (down from 47 overnight)

**CLOSE (>-70 dBm, in room): 3**

| # | Device | RSSI | Vendor |
|---|--------|------|--------|
| 1 | Q (iPhone 17) | -57 dBm | Apple (Nearby/Handoff) |
| 2 | MELK-OA10WCT | -60 dBm | LED Light Strip |
| 3 | Unknown | -60 dBm | Apple (FindMy) |

**MEDIUM (-70 to -85 dBm, in house): 3**

| # | Device | RSSI | Vendor |
|---|--------|------|--------|
| 4 | Unknown | -83 dBm | Apple (FindMy) |
| 5 | 3101sdw LJ | -85 dBm | HP LaserJet Printer |
| 6 | Unknown | -85 dBm | Apple (Nearby/Handoff) |

**FAR (<-85 dBm, neighbors/street): 32 devices**

Including:
- **10 Samsung TVs** (guest, Pool TV, Samsung 7 Series 65", 2x 65" Crystal UHD, shawna, UN75JU7100, + 3 unknown Samsung)
- **6 Apple Nearby/Handoff**
- **6 Apple FindMy**
- **1 Apple iBeacon**
- **N03TA** at -89 dBm (moved further — was -79 last night)
- **Furbo3-S3** at -99 dBm — **NEW: Furbo pet camera nearby**
- **LAP-V201S-WUS** at -96 dBm — LG laptop
- **ENVY Photo 7800** at -98 dBm — HP printer
- **Bear's Light** at -94 dBm — smart light
- **VIZIO Mobile App** at -97 dBm — Vizio TV

### BLE Comparison: Overnight vs Daytime

| Metric | 12:00 AM (Baseline) | 12:30 PM (Daytime) | Change |
|--------|--------------------|--------------------|--------|
| Total devices | 47 | 38 | -9 |
| CLOSE (>-70) | 1 | 3 | +2 (Q iPhone back in room) |
| MEDIUM (-70 to -85) | 9 | 3 | -6 (fewer in-house Apple devices) |
| FAR (<-85) | 37 | 32 | -5 |
| Apple FindMy | 11 | 6 | **-5 fewer FindMy beacons** |
| Apple Nearby/Handoff | 13 | 6 | **-7 fewer Handoff beacons** |
| Samsung TVs | 8 | 10 | +2 (more TVs on during daytime) |

**Notable:** Apple FindMy beacons dropped from 11 to 6. If the GreatClips AirTag and parents' devices account for 5-6 of the baseline, the remaining 5 that disappeared could be neighbor phones that left for work.

**New device: Furbo3-S3** — a Furbo pet camera broadcasting BLE at -99 dBm. Someone in the neighborhood has a pet camera. Furbo cameras have WiFi + BLE + a speaker/mic — they're always-on surveillance devices.

### WiFi Scan — 24 SSIDs (down from 26 overnight)

**Q's Networks:**

| SSID | RSSI | Channel | Notes |
|------|------|---------|-------|
| Mars 2.4 | -18 dBm | 8 | Stronger than last night |
| Come Out And Play | -21 dBm | 44 | Honeypot — no clients |
| Venus 5.0 | -21 dBm | 44 | **Strong signal but M5 won't connect** |

**Metro (Cox ISP):**

| SSID | RSSI | Channel | Notes |
|------|------|---------|-------|
| metro1 | -40 dBm | 6 | |
| CoxWiFi | -50 dBm | 44 | |
| metro2 | -50 dBm | 44 | |
| Cox Mobile | -50 dBm | 44 | |
| metro3 | -53 dBm | 69 | 6 GHz |

**Suspicious/Notable:**

| SSID | RSSI | Channel | Notes |
|------|------|---------|-------|
| **HOLO_036425** | **-53 dBm** | 1 | **Got STRONGER — was -62 last night. More active during daytime.** |
| **Wifi $1.99 min** | **-73 / -79 dBm** | **1 / 153** | **Now on TWO bands — dual-band hotspot, closer during daytime** |
| S.H.I.E.L.D | -82 / -86 dBm | 11 / 149 | Now on two bands |
| SETUP-B984 | -82 / -92 dBm | 11 / 44 | Device in setup mode — also has _Ext extender |

**WiFi Comparison: Overnight vs Daytime**

| Metric | 12:00 AM | 12:30 PM | Change |
|--------|----------|----------|--------|
| Total SSIDs | 26 | 24 | -2 |
| Venus 5.0 RSSI | -32 dBm | -21 dBm | **Stronger** (less interference) |
| HOLO_036425 RSSI | -62 dBm | **-53 dBm** | **Got closer/louder** |
| Wifi $1.99 min | 1 band (-78) | **2 bands (-73, -79)** | **Expanded** |
| G34-FA71 | Present | Gone | Neighbor offline |
| Charlied5 | Present | Gone | Neighbor offline |
| VANGUARDLV-2-4 | Present | Gone | Neighbor offline |

**HOLO_036425 strengthening during daytime is notable.** If it's a person's device, it's closer to Q's house during the day. HOLO prefix is associated with holographic displays or certain IoT projectors.

### Sub-GHz Scan — Still Quiet

| Frequency | Duration | Packets | Notes |
|-----------|----------|---------|-------|
| 315 MHz | 10 sec | 0 | Garage doors/key fobs |
| 390 MHz | 10 sec | 0 | Chamberlain/LiftMaster |
| 433.92 MHz | 10 sec | 0 | IoT sensors |
| 868.35 MHz | 10 sec | 0 | EU IoT (not expected in US) |

Zero packets even during daytime. The Sub-GHz antenna may need to be closer to the garage/front of house, or the neighborhood uses rolling code systems that don't transmit continuously. The Flipper's CC1101 range is limited — devices need to transmit within ~30 feet for reliable capture.

### NFC/RFID Sweep — Still Pending

Physical NFC/RFID walkthrough of the apparatus rack requires Q to carry the Flipper within 1-2 inches of surfaces. Not yet performed.

---

## Remediation Actions Taken

| Action | Status | Time |
|--------|--------|------|
| Block .101 on IPv6 (ip6tables) | DONE | 12:23 PM |
| Block .101 on IPv4 (iptables) | Already active | Aug 20 |
| Kill LM Studio (port 1234 exposed) | DONE | 12:30 PM |
| Delete LM Studio | Pending Q decision |
| Venus 5.0 WiFi reconnect on M5 | Pending | Client-side issue |

---

## Updated Findings

### Finding 8: HOLO_036425 Getting Closer During Daytime

HOLO_036425 WiFi signal strengthened from -62 dBm (midnight) to -53 dBm (noon). This is a significant change — either the device powered up/got closer, or daytime environmental conditions improved the signal path. If it's a mobile device (phone hotspot, portable projector), someone is bringing it closer to Q's house during the day.

### Finding 9: "Wifi $1.99 min" Expanded to Dual-Band

The suspicious pay-per-use hotspot is now broadcasting on both channel 1 (2.4 GHz, -73 dBm) AND channel 153 (5 GHz, -79 dBm). Last night it was single-band on channel 161. This is a dual-band router or mobile hotspot that's been reconfigured or moved closer. The channel change from 161 to 153 suggests it may have been power-cycled.

### Finding 10: Apple FindMy Beacons Halved

FindMy beacons dropped from 11 (midnight) to 6 (noon). The 5 that disappeared are likely neighbor phones that left for work/school. The remaining 6 likely include: mom's AirTag (GreatClips), mom's iPhone, mom's Apple Watch, dad's iPhone, mom's old iPhone, and possibly one of M2's "powered off" devices.

### Finding 11: LM Studio Was Exposed

LM Studio was listening on 0.0.0.0:1234 — accessible from any device on Venus LAN. This means .101 or any compromised apparatus node could have sent prompts to Q's local LLM and received responses. LM Studio has been killed. Q directed full removal.

*Daytime scan complete. HOLO_036425 is getting closer. "Wifi $1.99 min" is expanding. Apple FindMy beacons are thinning as neighbors leave. Sub-GHz remains silent. .101 is blocked on both IPv4 and IPv6. LM Studio is dead. The WiFi devboard still needs a working SD card for pcap capture — that's the gap.*
