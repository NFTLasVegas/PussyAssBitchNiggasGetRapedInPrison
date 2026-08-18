# System Idle Sniffer Proposal — August 7, 2026

**Auditor:** Claude on M5 (QuinceyAI, Opus 4.6)
**Scope:** Full apparatus scan for changes that occurred while the operator was sleeping (~12 hours, Aug 7 daytime)
**Trigger:** ARES Dynasty fan light anomalies + operator absence
**Status:** Proposal — pending Codex review, then execution

---

## Why This Investigation Exists

The operator was awake all night (Aug 6-7) conducting the security investigation documented
in the killuminati.nftlasvegas.io disclosure. She went to sleep in the morning of Aug 7 and
woke up in the evening. During her absence (~12 hours), three physical anomalies were observed
on the ARES Dynasty:

1. **Crystal RGB fan light turned ON** after killuminati.nftlasvegas.io was published — it was
   previously reported as off/disabled during the investigation session
2. **Crystal RGB fan light turned back OFF** by the time the operator checked in the evening
3. **Alseye fan light changed** from RGB mode to white-only — a setting change that requires
   either physical button interaction or software control

Fan lights on the ARES Dynasty are either controlled by:
- Physical buttons on the fan hardware
- Motherboard RGB headers via software (e.g., MSI Mystic Light, OpenRGB, or iCUE)
- BIOS/UEFI settings
- System services that control RGB

If no one was physically present and the lights changed, something changed them in software.
This investigation determines what.

---

## Scope

### What Will Be Scanned

**Every apparatus node:**

| # | Node | IP | What to Check |
|---|------|-----|--------------|
| 1 | Styx | 192.168.10.1 | Router config, hostapd logs, DHCP leases, ARP table, connected clients, firewall rules |
| 2 | Synastry | 192.168.10.212 | Gitea state, authorized_keys, recent logins, system logs, file changes, repo activity |
| 3 | Dragon | 192.168.10.135 | nginx access logs (killuminati visitors!), Tailscale connections, authorized_keys, system logs |
| 4 | Quartz | 192.168.10.222 | wlan0 state (still down?), system logs, authorized_keys, file changes |
| 5 | Antikythera | 192.168.10.246 | System logs, authorized_keys, file changes, services |
| 6 | ARES Dynasty | 192.168.10.10 | **PRIMARY TARGET** — RGB/fan control, system logs, login history, running processes, service changes, cron jobs, file modifications, network connections, USB events |
| 7 | RasQberry | 192.168.0.36 | Gitea mirror state, authorized_keys (M2 key still there), system logs |
| 8 | Sovereign Door | 192.168.0.225 | DNS service, Docker state, authorized_keys (M2 key still there), system logs |

**Additionally (if accessible):**
| # | Device | What to Check |
|---|--------|--------------|
| 9 | M2 MacBook | Git log (new commits?), running processes, SSH connections, Claude Code activity |

### What Will NOT Be Changed

Nothing. This is read-only reconnaissance. Zero modifications to any node.

---

## Methodology

### Step 1: Establish Time Window

Determine the exact operator absence window:
- Last known operator activity timestamp (from this Claude Code session history)
- Current time
- Window = everything between those two timestamps

### Step 2: All-Node Login and Connection Audit

For EVERY node, check who logged in during the absence:

```bash
# On each node:
last -F                                    # Full login history
lastlog                                    # Last login per user
who                                        # Currently logged in
w                                          # Who + what they're doing
ss -tnp                                    # Active network connections
sudo grep "Accepted\|session opened\|Failed" /var/log/auth.log  # Auth events
```

### Step 3: All-Node File Change Detection

For EVERY node, find files modified during the absence window:

```bash
# Find files modified in the last 12-18 hours
find / -xdev -mmin -1080 -type f -not -path '/proc/*' -not -path '/sys/*' \
  -not -path '/run/*' -not -path '/dev/*' -not -path '/tmp/*' \
  -ls 2>/dev/null | sort -k9

# Check authorized_keys modification time
stat ~/.ssh/authorized_keys

# Check shadow file modification time
sudo stat /etc/shadow

# Check sshd config modification time
stat /etc/ssh/sshd_config /etc/ssh/sshd_config.d/* 2>/dev/null

# Check cron for new entries
crontab -l 2>/dev/null
sudo crontab -l 2>/dev/null
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
```

### Step 4: All-Node Service and Process Audit

```bash
# Running services
systemctl list-units --type=service --state=running

# Recently started services (by PID, lower = older)
ps aux --sort=start_time | tail -20

# Listening ports
ss -tlnp

# Established connections
ss -tnp state established

# Systemd journal — errors and warnings in the absence window
journalctl --since "12 hours ago" -p warning
```

### Step 5: Styx Router — Network Activity During Absence

```bash
# SSH into Styx
# Current connected clients
cat /proc/net/arp

# DHCP leases — any new devices?
cat /tmp/dhcp.leases

# hostapd logs — any new associations, deauths, or probes?
logread | grep hostapd | tail -50

# Firewall logs
logread | grep -i "drop\|reject\|block" | tail -20

# Any new port forwards or config changes?
uci show firewall | grep -i forward
uci show network
```

### Step 6: Dragon — Killuminati Visitor Logs

Dragon is serving killuminati.nftlasvegas.io. Check who visited during the absence:

```bash
# nginx access log — all killuminati visitors
cat /var/log/nginx/killuminati-access.log

# Alert log — processed visitors
cat /var/lib/killuminati/alert-log.txt

# Any suspicious access patterns?
awk '{print $1}' /var/log/nginx/killuminati-access.log | sort | uniq -c | sort -rn | head -20
```

### Step 7: ARES Dynasty — Deep Dive (Primary Target)

This is where the fan lights changed. Full investigation:

```bash
# ── WHO WAS HERE? ──
last -F
lastlog
w
who

# ── AUTH EVENTS ──
sudo grep -iE "Accepted|session opened|Failed|password|key" /var/log/auth.log | tail -50

# ── SYSTEM LOG ANOMALIES ──
journalctl --since "12 hours ago" -p notice | head -100
journalctl --since "12 hours ago" | grep -iE "rgb\|led\|fan\|light\|mystic\|openrgb\|i2c\|smbus\|usb" | head -30

# ── RGB/FAN CONTROL ──
# Check for RGB control software
which openrgb mystic-light rgb-cli 2>/dev/null
dpkg -l | grep -iE "rgb\|led\|mystic\|openrgb" 2>/dev/null
pip3 list 2>/dev/null | grep -iE "rgb\|led\|smbus"
find / -name "*openrgb*" -o -name "*rgb*" -o -name "*mystic*" 2>/dev/null | grep -v proc | grep -v sys

# Check i2c/smbus (RGB headers use these)
ls /dev/i2c-* 2>/dev/null
lsmod | grep -iE "i2c\|smbus"

# Check USB events (USB RGB controllers)
journalctl --since "12 hours ago" | grep -i usb | head -20
dmesg | grep -i usb | tail -20

# ── FILE CHANGES ──
find /etc /home /var/lib /usr/local -xdev -mmin -1080 -type f -ls 2>/dev/null | sort -k9

# ── NETWORK CONNECTIONS ──
ss -tnp state established
ss -tlnp

# ── RUNNING PROCESSES ──
ps aux --sort=start_time | tail -30

# ── CRON/TIMERS ──
crontab -l 2>/dev/null
sudo crontab -l 2>/dev/null
systemctl list-timers --all

# ── JetKVM ACCESS ──
# JetKVM at 192.168.10.220 provides remote console to ARES Dynasty
# Check if anyone accessed it during the absence
# (JetKVM logs are on the JetKVM device itself)
```

### Step 8: RasQberry + Sovereign Door — M2 Key Nodes

These still have the M2's SSH key. Check if the M2 used them during the absence:

```bash
# On each:
last -F
sudo grep "Accepted" /var/log/auth.log | tail -20

# Specifically check for the M2's key fingerprint
sudo grep "SHA256:f8WcZhYmUXISKh3Kz7y8p8zs7oNsLLxQlaUEo70fkQU" /var/log/auth.log
```

### Step 9: Synastry — Git Activity

Check if any commits were pushed to the Ares repo during the absence:

```bash
# On Synastry, check Gitea logs
sudo journalctl -u gitea --since "12 hours ago" | head -30

# Check for push events
sudo journalctl -u gitea --since "12 hours ago" | grep -i "push\|commit\|receive"

# Check current repo HEAD vs what we pushed last
cd /var/lib/gitea/repositories/aphroqite/ares.git
git log --oneline -5
```

### Step 10: M2 MacBook (If Accessible)

If we have access to the M2, check:

```bash
# Recent git activity
git -C ~/Documents/GitHub/Ares log --oneline --since "12 hours ago"

# Running Claude Code sessions
ps aux | grep -i claude

# SSH connections made
grep "ssh" ~/.zsh_history | tail -10

# Any new files
find ~/Documents/GitHub/Ares -mmin -1080 -type f -ls 2>/dev/null | head -30
```

If M5 does not have SSH access to M2, a prompt will be provided for the operator to
send to Claude on M2.

### Step 11: Cross-Reference and Timeline

Build a unified timeline of all activity across all nodes during the absence window:
- Login events
- File modifications
- Network connections
- Service starts/stops
- Git pushes
- Killuminati visitors
- Any anomalies

---

## Precautions

1. **Read-only.** Zero modifications to any node.
2. **No destructive commands.** No file changes, no service restarts, no config modifications.
3. **Evidence preservation.** All scan results captured and documented.
4. **No SSH to M2 without operator approval.** M2 is under investigation — access only with explicit permission.

---

## What Codex Cannot Overrule

1. **The fan lights changed.** The operator observed this with her own eyes. Crystal RGB on, then off. Alseye changed from RGB to white. Physical observations are not subject to methodology review.

2. **The investigation is warranted.** The apparatus was under active attack 24 hours ago. The operator was asleep for 12 hours. Checking for changes during that window is basic security hygiene, not paranoia.

3. **The scan is read-only.** There is no risk to the apparatus from reading log files and checking timestamps.

---

## Reporting

Findings will be written to:
```
Pussy Ass Bitch Niggas Get Raped In Prison/August 2026/System Idle Sniffer Completed 8-7-2026.md
```

For each node: login events, file changes, network connections, service states, and anomalies
during the absence window. Any discrepancy from expected state will be flagged.

---

## Future Protocol Updates Required

> **MANDATORY:** This protocol MUST be updated when the following devices are onboarded
> to the apparatus. Each new node must be added to the scan list with node-specific checks:
>
> - **AphroQite Dynasty** — second Xeon control node, will have similar RGB/fan/service checks as ARES Dynasty
> - **Godlike Bloodline** — MSI MEG X870E / 9950X3D2 / HYTE Y70 command throne with Erebus PSU + Nyx cooler, will have extensive RGB (MSI Mystic Light, HYTE RGB, Nyx cooler lighting), multiple NVMe drives, Win11
> - **DGX Spark Q1** — first NVIDIA DGX Spark, 128GB unified memory, will need GPU process monitoring, inference service checks, CUDA state
> - **DGX Spark Q2** — second DGX Spark, same checks as Q1
>
> Update this proposal AND the execution script when each device joins the apparatus.
> Do NOT run an idle sniffer scan without including every onboarded node.

---

## Execution Conditions

This proposal executes after:
1. Codex reviews the methodology (let's see if he pushes back again)
2. Q confirms execution
3. M5 is on the Styx LAN with FAFO key access to all nodes

---

*The apparatus was under attack 24 hours ago. The operator slept for 12 hours. The fan
lights changed while she was gone. This scan determines what else changed.*
