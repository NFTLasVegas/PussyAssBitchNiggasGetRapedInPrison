# Metro WatchDog Proposal — August 11, 2026 (Revised per Codex Review)

**Purpose:** Monitor activity on Metro1/2/3 (192.168.0.0/24) visible from the Styx until the Metro2 PSK is rotated by Cox. The Metro2 PSK was exposed during this investigation session. Until Cox changes it, any device with the exposed password can join the network.

**Alert recipients:** QuinceyLee@NFTLasVegas.io AND Q@Quincey.ai
**Monitoring period:** Continuous until Metro2 PSK is rotated and verified
**Runs on:** Antikythera (192.168.10.246) — pulls data from Styx via SSH
**Honeypot:** "Come Out And Play" — open guest network on Styx rai1

---

## Known Limitations

The Styx is a Wi-Fi client on Metro2 — NOT the Cox AP. The watchdog can only see:
- Devices that appear in the Styx's ARP neighbor table (those that communicate through or near the Styx)
- SSH SYN packets that traverse the Styx's `apclii0` interface

The watchdog CANNOT see:
- Metro peer-to-peer traffic that doesn't pass through the Styx
- Devices using static IP/IPv6 that never trigger ARP
- Quiet devices that join Metro but don't communicate through the Styx
- Traffic through VPN/DoH/tunnels from Metro devices
- Cox AP association events (requires Cox admin access)

**This is partial reconnaissance, not omniscient monitoring.** It is better than the current state of ZERO monitoring on Metro.

---

## Threat Model

The Metro2 PSK was displayed in this Claude Code chat session. If the chat session, the M5's memory, or any intermediary system is compromised, an attacker could use the exposed PSK to:

1. Join Metro1/2 Wi-Fi and access the home network
2. Intercept traffic from devices on Metro1/2
3. Position themselves as a man-in-the-middle
4. Access any device on Metro1/2 with open ports

---

## Component 1: Device Census (Every 2 Minutes)

**Script:** `/usr/local/bin/metro-watchdog.py` on Antikythera
**Data source:** Styx ARP neighbor table via `ssh root@192.168.10.1 "cat /proc/net/arp | grep apclii0"`

**Detection confidence:** ARP observation only. A device appearing in ARP confirms it communicated on Metro. Absence from ARP does NOT confirm absence from Metro.

### Baseline Devices (known at time of proposal)

| MAC | Identity | Status | Reappearance Rule |
|-----|----------|--------|-------------------|
| cc:f3:c8:72:98:3f | Cox Router (gateway) | KNOWN | Normal |
| 88:a2:9e:4c:54:7a | RasQberry | KNOWN | Normal |
| a4:02:b7:d6:f4:73 | Fire Stick #1 (dad's) | UNPLUGGED | **CRITICAL — MAC was SPOOFED on Aug 10. Any reappearance = investigation** |
| c4:1c:ff:bf:56:c9 | Vizio TV | UNPLUGGED | **WARNING — was running with telemetry ON while "off"** |
| 54:e0:19:04:1c:8d | Ring Stick Up Camera | KNOWN | Normal |
| a0:fb:c5:58:e5:da | JoAnn's iPhone | KNOWN | Normal |
| 14:b5:cd:eb:0e:4d | Sovereign Door | **DESTROYED** | **CRITICAL — device is physically smashed. Any reappearance = MAC spoofing** |
| cc:f7:35:f4:47:e5 | Fire Stick #2 (hijacked) | UNPLUGGED | **CRITICAL — BrightData proxy device. Any reappearance = investigation** |

### Alert Triggers

| Trigger | Severity | Action |
|---------|----------|--------|
| NEW MAC appears on Metro | **CRITICAL** | Email BOTH addresses immediately |
| DESTROYED/UNPLUGGED device MAC reappears | **CRITICAL** | Email BOTH addresses — confirmed MAC spoofing |
| Known MAC reappears after being offline >1h | **WARNING** | Email BOTH addresses |
| Device with SSH (port 22) open detected | **CRITICAL** | Email BOTH addresses + log full port scan |
| Device with ADB (port 5555) open detected | **CRITICAL** | Email BOTH addresses + log full port scan |
| Device with DNS (port 53) open detected | **CRITICAL** | Email BOTH addresses |
| Device with any proxy port (1080, 8888, 9080) open | **CRITICAL** | Email BOTH addresses |
| >15 devices on Metro simultaneously | **WARNING** | Email BOTH addresses |
| Honeypot "Come Out And Play" client connects | **CRITICAL** | Email BOTH addresses + full device fingerprint |

### Data Collected Per Device Per Scan

- IP address
- MAC address
- MAC type (hardware vs randomized — check bit 1 of first octet)
- OUI manufacturer lookup
- Hostname (reverse DNS from Cox router)
- Open ports (22, 53, 80, 443, 1080, 3000, 5555, 8008, 8009, 8080, 8443, 8888, 9080, 55443, 62078)
- First seen timestamp (UTC + PDT)
- Last seen timestamp (UTC + PDT)
- Connection duration
- Detection confidence (ARP observation)

---

## Component 2: SSH Attempt Monitor (Continuous)

**Script:** `/usr/local/bin/metro-ssh-watch.sh` on Styx
**Method:** `tcpdump` on `apclii0` capturing SYN packets to port 22 (confirmed: tcpdump 4.9.3 installed, filter parses successfully)

```
tcpdump -i apclii0 'tcp[tcpflags] & tcp-syn != 0 and dst port 22' -l --immediate-mode
```

**Detection scope:** Captures SSH SYN packets that traverse the Styx's `apclii0` interface. Metro peer-to-peer SSH that doesn't route through the Styx is NOT captured. This is a hardware limitation of the Styx being a client, not the AP.

### Data Logged Per SSH Attempt

- Event ID (sequential, unique per alert)
- Timestamp (UTC + PDT)
- Source IP
- Source MAC (from ARP correlation — noted as unreliable when ARP entries are INCOMPLETE/FAILED)
- Destination IP
- Direction (inbound to Metro device or outbound from Metro device)
- OUI of source MAC
- Detection confidence (tcpdump capture on apclii0)

### Alert Trigger

**ANY SSH SYN packet visible on Styx apclii0 triggers an immediate email alert** with full packet details. There should be ZERO SSH traffic on Metro — apparatus SSH goes through the Styx LAN, not Metro.

---

## Component 3: Command Execution Logging

### On the Styx Router

The Styx's `logread` captures all dropbear SSH sessions. The watchdog polls this every 60 seconds:

```
ssh root@192.168.10.1 "logread | grep dropbear"
```

Any NEW dropbear entry that doesn't originate from M5 (192.168.10.202 or 192.168.10.240) triggers an alert.

### On Apparatus Nodes (Dragon, Synastry, Quartz, Antikythera, ARES Dynasty)

Each node's `auth.log` is polled every 60 seconds for non-M5 SSH sessions:

```
grep -v '192.168.10.202\|192.168.10.240' /var/log/auth.log | grep -iE 'accept|session|opened'
```

Any match triggers an alert with the full log line.

### On the RasQberry

Q manages SSH access directly. Watchdog does not connect to the RasQberry via SSH. If Q grants Antikythera SSH access, auth.log polling can be added.

---

## Component 4: Email Alerts

**Transport:** Python SMTP via existing `netwatch-mail.py` on Antikythera (NOT msmtp — use the transport already deployed and tested)
**Recipients:** QuinceyLee@NFTLasVegas.io, Q@Quincey.ai
**Credentials:** FastMail SMTP at `/etc/netwatch/smtp.env`

### Email Format

```
Subject: [METRO WATCHDOG] CRITICAL — New device on Metro2

Event ID: MW-2026-08-11-0001
Timestamp: 2026-08-11 02:15:33 UTC / 2026-08-10 19:15:33 PDT
Collector: Antikythera (192.168.10.246)
Source: Styx ARP observation on apclii0
Detection Confidence: ARP neighbor entry (REACHABLE)

Alert Type: NEW DEVICE
Severity: CRITICAL

Device:
  IP: 192.168.0.xxx
  MAC: aa:bb:cc:dd:ee:ff
  MAC Type: hardware / randomized
  OUI: [manufacturer or "private (randomized)"]
  Hostname: [from Cox DNS or "unresolvable"]
  Open Ports: [list with service names]

Baseline Match: NOT IN BASELINE — unknown device
Spoofing Check: MAC does not match any destroyed/unplugged device

Raw Evidence:
  ARP entry: "192.168.0.xxx 0x1 0x2 aa:bb:cc:dd:ee:ff * apclii0"
  Scan timestamp: 2026-08-11T02:15:33Z

Action Required: Verify this device or investigate.

PSK rotation status: PENDING (Cox call required)
Watchdog uptime: 3h 22m
Next heartbeat: 2026-08-11 03:00:00 PDT

---
Metro WatchDog on Antikythera | Event MW-2026-08-11-0001
```

### Alert Rate Limiting

Rate limiting is per **(event type, MAC, destination port)** tuple — not per MAC alone:
- Same (type, MAC, port) tuple: maximum 1 alert per 10 minutes
- This ensures a NEW DEVICE alert does NOT suppress a subsequent SSH or ADB alert for the same MAC
- Summary digest: every 6 hours, send a full device census regardless of alerts
- Heartbeat: every 1 hour, send a "watchdog alive" confirmation email

### Heartbeat Monitoring

If a heartbeat email fails to send:
- Log the failure to `/var/log/metro-watchdog/heartbeat-failures.log`
- Retry every 5 minutes for up to 30 minutes
- If all retries fail, log WATCHDOG DELIVERY FAILURE and continue monitoring (do not stop collecting data because email is down)

Q should independently verify heartbeat arrival — if no heartbeat for >2 hours, the watchdog or SMTP is down.

---

## Component 5: Log Retention

**Storage:** Persistent filesystem on Antikythera — NOT `/var/log` (which is volatile zram on Antikythera). Logs written to `/var/www/antikythera/metro-watchdog/` (persistent storage, same partition as netwatch dashboard).

| Log File | Contents | Rotation |
|----------|----------|----------|
| `/var/www/antikythera/metro-watchdog/census.log` | All device census snapshots | Daily, keep 30 days |
| `/var/www/antikythera/metro-watchdog/alerts.log` | All triggered alerts with event IDs | Daily, keep 90 days |
| `/var/www/antikythera/metro-watchdog/ssh-attempts.log` | All SSH SYN packets on apclii0 | Daily, keep 90 days |
| `/var/www/antikythera/metro-watchdog/styx-ssh.log` | Styx dropbear entries | Daily, keep 30 days |
| `/var/www/antikythera/metro-watchdog/node-auth.log` | Apparatus node auth events | Daily, keep 30 days |
| `/var/www/antikythera/metro-watchdog/heartbeat.log` | Watchdog alive confirmations | Daily, keep 7 days |
| `/var/www/antikythera/metro-watchdog/heartbeat-failures.log` | Failed email deliveries | Keep 90 days |

Logrotate policy to be created at `/etc/logrotate.d/metro-watchdog`.

---

## Component 6: Dashboard Integration

Extend the existing netwatch dashboard on Antikythera with a Metro section:

**File:** `/var/www/antikythera/netwatch/metro-watchdog.json`

Updated every 2 minutes with:
- Current device list with all metadata
- Alert history (last 24 hours) with event IDs
- SSH attempt log (last 24 hours)
- PSK rotation status (PENDING / COMPLETED)
- Watchdog uptime and last scan time
- Collection health (last successful Styx SSH, last successful email delivery)

---

## Component 7: Honeypot — "Come Out And Play"

**SSID:** Come Out And Play
**Security:** OPEN (no password)
**Radio:** rai1 on Styx (5GHz guest radio — cannot be disabled due to firmware bug, weaponized instead)
**Network:** Isolated guest subnet (no access to Styx LAN or apparatus)

Any device connecting to the honeypot triggers:
- **CRITICAL email alert** to both addresses with full device fingerprint
- Full port scan of the connecting device
- Continuous monitoring of the device's traffic for the duration of connection
- ARP/MAC capture and OUI lookup

The honeypot provides internet access but ZERO access to the apparatus. The attacker gets a working connection while Q gets their device fingerprint.

---

## Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| SSH from Antikythera to Styx | Verify + pin host key | Do NOT accept blindly — verify key fingerprint first |
| tcpdump on Styx | ✅ Installed (4.9.3) | Filter confirmed parsing successfully |
| Python SMTP (netwatch-mail.py) | ✅ Available | Use existing transport — do NOT install msmtp |
| FastMail SMTP credentials | ✅ Available | `/etc/netwatch/smtp.env` on Antikythera |
| OUI database | ✅ Available | `/var/lib/netwatch/oui.csv` on Antikythera |
| Persistent storage on Antikythera | Use `/var/www/antikythera/` | NOT `/var/log` (volatile zram) |

---

## Implementation Steps

1. Pin Styx SSH host key on Antikythera (verify fingerprint, add to `known_hosts`)
2. Deploy `metro-ssh-watch.sh` on Styx (continuous tcpdump logger)
3. Deploy `metro-watchdog.py` on Antikythera (census + alerts + polling) using existing `netwatch-mail.py` for email
4. Create log directory at `/var/www/antikythera/metro-watchdog/`
5. Create logrotate policy at `/etc/logrotate.d/metro-watchdog`
6. Add cron entry on Antikythera (every 2 minutes)
7. Send test alert to BOTH email addresses — verify delivery
8. Verify heartbeat emails arrive
9. Run go-live tests:
   - Verify a new device appearing in ARP triggers CRITICAL alert
   - Verify Styx SSH from non-M5 source triggers alert
   - Verify honeypot connection triggers CRITICAL alert
   - Verify Styx reboot recovery (watchdog reconnects after Styx comes back)
   - Verify email failure retry (temporarily break SMTP, confirm retry + logging)
10. Monitor until Cox rotates Metro2 PSK

---

## Deactivation Criteria

The Metro WatchDog runs until ALL of the following are true:

1. Cox has changed the Metro2 PSK to a value never exposed in any chat or log
2. Q has verified the new PSK works on her devices
3. Q has verified the OLD PSK is **rejected** (controlled test: attempt connection with old PSK, confirm failure)
4. A full device census shows only known devices on Metro
5. All UNKNOWN/Pussy Ass Bitch devices have been triaged (identified or blocked)
6. 24 hours have passed with demonstrably healthy collection AND successful alert delivery (not merely "zero alerts" — the watchdog must prove it's working, not just silent)
7. Closure evidence preserved: final census snapshot, alert log hash, confirmation of PSK rejection test

---

*This watchdog monitors the Metro1/2/3 network for unauthorized access following PSK exposure during an active security investigation. It provides device census via Styx ARP observation, SSH capture via tcpdump, email alerts via existing Python SMTP transport, honeypot monitoring via "Come Out And Play" open network, and persistent log retention on Antikythera. Detection scope is limited to traffic visible from the Styx — partial reconnaissance that is better than the current state of zero Metro monitoring.*
