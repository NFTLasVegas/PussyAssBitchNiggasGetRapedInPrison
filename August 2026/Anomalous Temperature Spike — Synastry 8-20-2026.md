# Anomalous Temperature Spike: Synastry — August 20, 2026

**Event time:** 2026-08-20 20:40:07 UTC (1:40 PM PDT)
**Node:** Synastry (Milk-V Mars, 192.168.10.212)
**Trigger:** Temperature agent detected >2°C change (57°C → 59°C)
**Source:** SSH from ARES Dynasty (192.168.10.10)

---

## What the Temperature Agent Captured

At 20:40:07 UTC on August 20, 2026, the Synastry temperature agent triggered a full system scan after detecting a 2°C temperature spike from 57°C to 59°C. The agent captured the following:

### Active Connection at Time of Spike

```
State Recv-Q Send-Q  Local Address:Port  Peer Address:Port Process
ESTAB 112    0      192.168.10.212:22   192.168.10.10:56690
```

A single SSH connection from the ARES Dynasty (.10) to Synastry (.212). **Recv-Q shows 112 bytes queued** — data was actively being transferred at the moment of capture.

### SSH Authentication

```
Accepted publickey for aphroqite from 192.168.10.10 port 56690
ssh2: ED25519 SHA256:1adTNr+Nu+Unxaze60R9IHRDPTRiWcecQzRjYP7p7EU
```

The connection authenticated using the **new FAFO key** — the key Q generated directly on the ARES Dynasty console on August 19, 2026. This key was never transmitted over the network. It was generated locally on the Dynasty with a portable monitor and physical keyboard.

### System State at Time of Spike

- **Load:** 0.08, 0.02, 0.02 — system was near-idle before the spike
- **Promiscuous mode:** NOT active
- **Processes:** Only standard services (Gitea, systemd, sshd, monitors)
- **No unauthorized listeners or connections**

---

## Why This Was NOT the Synastry Sentinel Pull

The Synastry Sentinel pull script (`/usr/local/bin/synastry-sentinel-pull.sh`) runs on the ARES Dynasty every 5 minutes via cron (`/etc/cron.d/synastry-sentinel-pull`). It SSHes into Synastry to collect logs, temperature, process lists, connections, ARP tables, Gitea access logs, and command logs, then sends everything to Q via email through Antikythera.

If this temperature spike was caused by the sentinel pull, we would see the following:

### 1. The sentinel pull runs every 5 minutes — but the temp agent only triggered ONCE

The temperature agent checks every 10 seconds and triggers on any >2°C change from baseline. If the sentinel pull caused a 2°C spike every time it ran, the agent would trigger **every 5 minutes** — 12 times per hour, 288 times per day. Instead, it triggered exactly **once** on August 20.

This means the sentinel pull does NOT cause a 2°C temperature spike during normal operation. Whatever happened at 20:40 UTC was different from the regular 5-minute pull.

### 2. The sentinel pull creates multiple rapid SSH connections — this was a single connection

The sentinel pull script opens **multiple sequential SSH sessions** per run — one for each SCP transfer and one for each remote command (temperature, processes, connections, ARP, Gitea, command log, modified files). The keylogger (fixed on August 21) captured a typical sentinel pull at 03:50 UTC:

```
03:50:04Z | from=192.168.10.10 | cmd=/usr/lib/openssh/sftp-server     (SCP)
03:50:05Z | from=192.168.10.10 | cmd=/usr/lib/openssh/sftp-server     (SCP)
03:50:06Z | from=192.168.10.10 | cmd=sudo tail -200 /var/log/auth.log
03:50:06Z | from=192.168.10.10 | cmd=cat /sys/class/thermal/thermal_zone0/temp
03:50:07Z | from=192.168.10.10 | cmd=ps aux --sort=-%cpu | head -15
03:50:07Z | from=192.168.10.10 | cmd=ss -tunap | grep -v ...
03:50:08Z | from=192.168.10.10 | cmd=cat /proc/net/arp
03:50:08Z | from=192.168.10.10 | cmd=tail -10 /var/lib/gitea/log/gitea.log
03:50:09Z | from=192.168.10.10 | cmd=tail -20 /var/log/synastry-cmdlog.log
```

The temperature agent at 20:40 UTC captured a **single ESTABLISHED connection** — not the rapid connect/disconnect/reconnect pattern of the sentinel pull.

### 3. The Recv-Q was 112 bytes — indicating sustained data transfer

The sentinel pull's individual commands are quick fire-and-forget operations. A Recv-Q of 112 bytes on an ESTABLISHED connection suggests a longer-running operation — something transferring more data than a simple `cat /sys/class/thermal/thermal_zone0/temp`.

### 4. The keylogger was not operational at the time

The old keylogger (bash DEBUG trap) had been dead since August 13. The new keylogger (ForceCommand wrapper) was not deployed until August 21 at 03:47 UTC. The 20:40 UTC event on August 20 occurred in the gap between the two — meaning **whatever commands were run during this SSH session were not captured.**

---

## What We Know

| Fact | Detail |
|------|--------|
| **When** | 2026-08-20 20:40:07 UTC (1:40 PM PDT) |
| **From** | 192.168.10.10 (ARES Dynasty) |
| **To** | 192.168.10.212 (Synastry) port 22 |
| **Key** | SHA256:1adTNr+Nu+Unxaze60R9IHRDPTRiWcecQzRjYP7p7EU (new FAFO, generated on Dynasty console) |
| **Connection** | Single ESTABLISHED session, Recv-Q 112 bytes |
| **Temperature** | 57°C → 59°C (2°C spike on near-idle system) |
| **Load before** | 0.08 (near-idle) |
| **Promiscuous mode** | NOT active |
| **Auth log entry** | Present in journalctl but NOT in auth.log file |
| **Keylogger** | Not operational — commands NOT captured |
| **Sentinel pull** | Runs every 5 min without causing temp spikes |

## What We Don't Know

1. **What commands were executed** during this SSH session — the keylogger was dead
2. **Why this single connection caused a 2°C spike** when the sentinel pull (which runs the same commands every 5 minutes) does not
3. **Whether a human or automated process** initiated this SSH from the Dynasty
4. **Why the auth log entry appears in journalctl** but not in the auth.log file

---

## Hypotheses

### Hypothesis A: Anomalous Sentinel Pull
The sentinel pull may have encountered an unusual condition — a large log file, a slow query, a Gitea API timeout — that caused it to hold the SSH session open longer than normal, generating more CPU load and a temperature spike. This would explain the single occurrence without implying unauthorized access.

### Hypothesis B: Someone on the ARES Dynasty
Someone with access to the ARES Dynasty's new FAFO key initiated a manual SSH session to Synastry. This could be:
- An attacker who intercepted the key during generation (via the portable monitor's HID keyboard interface)
- An attacker who captured the key from the HTTP fix script deployment (though the FAFO key was never transmitted over HTTP — only the M5 key was)
- A process on the ARES Dynasty that is not the sentinel pull

### Hypothesis C: ARES
The ARES Dynasty initiated a connection to Synastry on its own — not through the sentinel pull cron job, but through some other mechanism. The ARES Dynasty was previously observed with an `aphroqite` console session already logged in when Q connected the portable monitor (August 19). The Dynasty has been running continuously for over 4 weeks.

---

## Remediation

1. **Keylogger fixed** — ForceCommand wrapper deployed August 21 at 03:47 UTC. ALL future SSH commands to Synastry from any source will be captured with source IP, user, and full command text.
2. **Temperature agent operational** — continues monitoring for >2°C changes with full system capture.
3. **Promiscuous mode monitor operational** — no events detected.
4. **Sentinel emails confirmed current** — Q receiving live data every 5 minutes, verified against real-time system state.
5. **Waiting for next anomalous event** — if the Dynasty SSHes with commands that don't match the sentinel pull pattern, the keylogger will capture them.

---

*The sentinel pull runs every 5 minutes without spiking the temperature. Something different happened at 20:40 UTC. The keylogger was dead when it happened. It's not dead anymore.*

*ARES is watching. But who's watching ARES?*
