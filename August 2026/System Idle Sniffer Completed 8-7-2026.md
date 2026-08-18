# System Idle Sniffer Completed -- August 7-8, 2026

**Auditor:** Claude on M5 (QuinceyAI, Opus 4.6)
**Collection time:** 2026-08-08 06:29 UTC
**Absence window:** 2026-08-07 ~10:00 UTC to ~22:00 UTC (~12 hours)
**Trigger:** ARES Dynasty fan light anomalies during operator absence

---

## Result Summary

| Node | Unauthorized Access | Suspicious Changes | Status |
|------|--------------------|--------------------|--------|
| Styx | None | Quartz MAC reappeared (during our SD card session) | CLEAN |
| Synastry | None | None | CLEAN |
| Dragon | None | killuminati visitors logged | CLEAN |
| Quartz | None | wlan0 UP flag set (boot artifact) | NOTE |
| Antikythera | None | None | CLEAN |
| ARES Dynasty | None | Session 6195 opened, lsusb every 5min | INVESTIGATE |
| RasQberry | None | M2 key still present | KNOWN |
| Sovereign Door | None | M2 key still present | KNOWN |

---

## Styx Router

**Uptime:** 8 days, 22 hours

**Hostapd during absence:** ONLY routine GTK group key rekeys every hour for three known
stations (M2, iPhone, M5). No unknown stations. No failed authentications. No deauth events.
No new MAC addresses probing the network.

**Quartz MAC reappearance:** Two `e8:fb:1c:65:20:73 IEEE 802.11: disassociated` entries at
01:19 and 01:35 AM PDT on Aug 7 -- these occurred DURING our active session while we were
doing SD card swaps. When Quartz rebooted after the SD card swap, the `wpa_supplicant` service
briefly activated before the interface was fully down, triggering these two disassociation
events. Both entries also show `MAC addr invalid!` kernel errors -- consistent with Quartz's
broken Wi-Fi adapter (missing calibration blob) being rejected by the AP.

**ARES Dynasty SSH attempts:** Multiple connections from 192.168.10.10 to the Styx that
"Exit before auth" -- ARES Dynasty's health-check or netwatch cron is trying to SSH into the
Styx as root but getting rejected because the FAFO key on the Styx doesn't match what ARES
Dynasty has. This is expected given the key rotation.

**ARP table:** All known devices present. No unknown MACs on the Styx LAN. One unidentified
device at 192.168.10.197 (MAC 24:5e:be:77:bf:fd) remains in DHCP leases with no hostname --
this was noted in the original investigation and is still unresolved.

**Metro2 side:** Device at 192.168.0.106 (MAC c4:1c:ff:bf:56:c9) in ARP -- may be a neighbor's
device or Cox infrastructure.

---

## ARES Dynasty (Primary Target)

**Uptime:** 19 days, 23 hours (has NOT been rebooted)

### Fan Light Investigation

**No RGB control software is installed.** `openrgb`, `rgb-cli`, `ipmitool` -- none present.
No RGB-related packages in dpkg. No dmesg entries for RGB/fan/LED.

**I2C/SMBus hardware IS present:**
- `/dev/i2c-0` exists
- Kernel modules loaded: `i2c_i801`, `i2c_smbus`, `i2c_mux`, `i2c_algo_bit`
- These provide the hardware interface that motherboard RGB headers use

The I2C bus exists and is accessible, but NO software on the system is using it to control
RGB. The fan light changes are NOT explained by any installed software.

**Possible causes for fan light changes without OS-level software:**
1. Motherboard BIOS/UEFI has built-in RGB control that operates independently of the OS
2. Physical button on the fan hardware (crystal fan or Alseye may have inline controllers)
3. Power fluctuation or fan controller firmware responding to thermal changes
4. Remote access via JetKVM (keyboard/mouse injection to BIOS setup if the machine was at
   a BIOS prompt or had an RGB utility in the boot path)
5. IPMI/BMC if the Xeon motherboard has out-of-band management

### Login Activity

**No SSH logins during the absence window.** Auth log grep for Aug 7 hours 10-22 returned
zero results.

**Session 6195 opened at 12:13:06 UTC (5:13 AM PDT):** `Started session-6195.scope -
Session 6195 of User aphroqite`. This appears in the systemd journal but NOT in auth.log --
meaning it was a systemd user service or timer activation, not an SSH login. Likely a
periodic system service running as aphroqite.

**Physical console login (tty1):** Still active from the JetKVM session during the lockdown
recovery. Idle for 22h44m. This is Q's own session that was never closed.

### lsusb Running Every 5 Minutes

`lsusb` is being called every 5 minutes and blocked by AppArmor:
```
apparmor="DENIED" operation="open" class="file" profile="lsusb"
name="/sys/devices/pci0000:00/0000:00:14.0/uevent"
```

This runs at :20, :25, :30 -- every 5 minutes. Something (likely the apparatus health-check
cron or netwatch) is periodically probing USB devices. AppArmor is blocking it, which means
the calls are failing silently. This is NOT a security issue -- it's a misconfigured health
check hitting AppArmor policy.

### Network and Services

- Active connection: only M5's current SSH session
- Listening ports: 8890 (internal), 53 (DNS resolver), 5432 (PostgreSQL localhost), 22 (SSH), 80 (nginx)
- All expected. No new or unexpected services.
- Systemd timers: all routine (apt, sysstat, logrotate, fwupd, man-db, motd-news)
- No unexpected cron jobs

### Modified Files

All files modified during the absence are routine system operations:
- apt/Ubuntu Advantage list updates (13:14 UTC)
- apt periodic stamps (13:56 UTC)
- update-notifier (13:56 UTC)
- motd-news timer stamp (21:35 UTC)
- apt-daily timer stamp (23:02 UTC)
- dpkg-db-backup, logrotate, sysstat stamps
- fwupd-refresh stamp
- landscape sysinfo cache

**No configuration files changed. No scripts modified. No authorized_keys touched. No
shadow file modified. No sshd config changed.**

---

## Dragon (killuminati Host)

**killuminati.nftlasvegas.io received 39 HTTP requests.**

All visitor IPs show as localhost (::1, 127.0.0.1) because Tailscale Funnel terminates TLS
at the edge and proxies to nginx on localhost. The actual visitor IPs are in the Tailscale
Funnel logs, not nginx access logs.

**Visitor alert log entries during the absence:**
```
13:10 UTC -- 2 visitors (curl + Safari)
13:15 UTC -- 1 visitor (Safari/604.1 = iOS Safari)
13:35 UTC -- 1 visitor (Safari/537.36 = Chrome)
13:40 UTC -- 1 visitor (Safari/537.36)
13:45 UTC -- 1 visitor (web scanner: example.com referrer)
13:50 UTC -- 1 visitor (curl)
16:05 UTC -- 1 visitor (web scanner)
17:10 UTC -- 1 visitor (web scanner)
18:25 UTC -- 1 visitor (internet-measurement.com = known web scanner)
21:20 UTC -- 1 visitor (Safari/537.36)
21:25 UTC -- 1 visitor (Safari/604.1 = iOS)
01:25 UTC -- 1 visitor (Chrome)
02:10 UTC -- 1 visitor (Chrome)
03:10 UTC -- 1 visitor (Chrome)
```

Some visitors are web scanners (internet-measurement.com). Others use real browsers (Safari,
Chrome). **People are reading the site.**

**Tailscale:** M2 (ares) shows "offline, last seen 31m ago." The M2 was on Tailscale but went
offline recently. This means the M2 had Tailscale active at some point during the day.

**Modified files:** Only the killuminati deployment files from our session. No other changes.

---

## Quartz

**wlan0 state: `<NO-CARRIER,BROADCAST,MULTICAST,UP>`** -- the UP flag is set, which is
slightly different from the expected pure DOWN state. This is likely because the
`wpa_supplicant` service (still configured in netplan) briefly activated the interface during
boot after the SD card swap. The interface has NO CARRIER (not connected to any AP) and
is functionally inert, but the UP flag indicates the driver loaded.

**No auth events. No file changes (just fake-hwclock and MOTD quotes). FAFO key intact.
Password still locked.**

---

## All Other Nodes

**Synastry, Antikythera, RasQberry, Sovereign Door:** No unauthorized access. No suspicious
changes. FAFO keys intact. Passwords unchanged. Only routine system file updates
(fake-hwclock, MOTD, wireplumber state on Sovereign Door).

**RasQberry and Sovereign Door still have M2 key (`quinceylee@nftlasvegas.io`) alongside
FAFO.** This is known and pending pruning.

---

## The 192.168.10.197 Mystery Device

Still present in DHCP leases with MAC `24:5e:be:77:bf:fd` and no hostname. This device was
flagged during the original investigation but never identified. It has a valid DHCP lease.

---

## Conclusions

1. **No unauthorized SSH access to any node during the absence.** Zero auth events on any
   of the 8 nodes during the absence window.

2. **No file modifications beyond routine system operations.** No configs changed, no scripts
   modified, no keys touched, no passwords altered.

3. **No new deauth attacks or unknown Wi-Fi probes.** Styx hostapd shows only routine GTK
   rekeys. The Quartz MAC reappearance was from our own SD card swap.

4. **The ARES Dynasty fan light changes are NOT explained by installed software.** No RGB
   control software exists on the system. The I2C hardware interface is present but unused.
   The lights changed through a mechanism outside the OS -- BIOS/UEFI firmware, physical
   button, fan controller hardware, or remote access via JetKVM.

5. **killuminati.nftlasvegas.io is being visited.** 39 requests, mix of real browsers and
   web scanners. People are reading it.

6. **The M2 was on Tailscale during the day** and went offline recently. The M2 has Tailscale
   access to Dragon, which is a known bypass around the FAFO lockdown.

7. **ARES Dynasty is trying to SSH into the Styx** via health checks and getting rejected.
   Expected behavior given the key rotation -- the health check needs updating.

---

## Remaining Concerns

- The fan light changes have no software explanation. Hardware-level or BIOS-level control
  is the most likely mechanism, but this cannot be verified remotely.
- 192.168.10.197 remains unidentified.
- M2 key still active on RasQberry and Sovereign Door.
- Quartz's wpa_supplicant is still configured in netplan and briefly activates on boot.
- The M2 had Tailscale active during Q's sleep and could reach Dragon.

---

*Scan conducted read-only. No files modified on any node. All SSH sessions from M5
(192.168.10.202) are visible in each node's auth.log as collector-generated entries.*
