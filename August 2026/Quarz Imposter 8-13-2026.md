# Quarz Imposter — August 13, 2026

**Discovered:** 2026-08-13 ~09:20 UTC during Venus ARP table review
**Operator:** Q — identified that the "Ismenian Dragon" page title was not a node she created, which led to re-examination of all device identities
**Network:** Venus 5.0 (192.168.10.0/24)

---

## Summary

A device at 192.168.10.222 appeared on the Venus LAN with a custom MAC address that spells "quarz" in hexadecimal — one letter short of Quartz, Q's apparatus node. The MAC was deliberately crafted to impersonate Quartz in ARP tables. The device was whitelisted in the WatchDog's known device list as "quartz (old randomized MAC)" — a false label that made it invisible to all monitoring. The device is currently offline, has never been logged by any apparatus monitoring system, and Q confirms she did not create it.

---

## Device Identity

| Field | Value |
|-------|-------|
| IP | 192.168.10.222 |
| MAC | 02:71:75:61:72:7a |
| MAC Type | **Locally administered** (02: prefix — custom/spoofed) |
| MAC Hex Decode | 02:**71**(**q**) **75**(**u**) **61**(**a**) **72**(**r**) **7a**(**z**) = "**quarz**" |
| Current Status | **OFFLINE — Destination Host Unreachable** |
| Ping Response | 0/3 packets, 100% loss |
| Open Ports | None (all scanned ports timed out) |
| HTTP Server | None |
| SSH Banner | None |

---

## Why This Is Not Quartz

Quartz has two network interfaces. Neither matches the "quarz" MAC:

| Interface | IP | MAC | Purpose |
|-----------|-----|-----|---------|
| end0 (Ethernet) | 192.168.10.172 | 82:7b:f3:db:73:38 | Primary wired connection |
| wlx6815790f3764 (WiFi) | 192.168.10.236 | 68:15:79:0f:37:64 | AX900 WiFi adapter |

The "quarz" MAC `02:71:75:61:72:7a` is NOT associated with any Quartz interface. It is a fabricated MAC designed to look like Quartz at a glance — especially in ARP table dumps where an admin might see "quarz" in hex and assume it's a randomized Quartz MAC.

Q confirms: **"I did not set a custom MAC on any device that spells 'quarz.'"**

---

## How It Avoided Detection

### 1. WatchDog Whitelist

The MAC `02:71:75:61:72:7a` was present in the WatchDog's `KNOWN_VENUS` device list on Antikythera:

```bash
KNOWN_VENUS='... 02:71:75:61:72:7a ...'
```

With the device name:
```bash
["02:71:75:61:72:7a"]="quartz (old randomized MAC)"
```

This label is FALSE. Quartz has never used this MAC. The label was added by Claude Code when deploying WatchDog v5 on August 12, 2026. Claude saw the MAC in the ARP table, decoded the hex as "quarz," assumed it was an old Quartz MAC, and whitelisted it without verifying against Quartz's actual interfaces. **This was a mistake by Claude that allowed the imposter to remain invisible to monitoring.**

**Remediation:** MAC removed from `KNOWN_VENUS` and device name entry deleted on August 13. The WatchDog will now fire a CRITICAL alert if this MAC reappears.

### 2. No Styx Records

| System | Record of 02:71:75:61:72:7a |
|--------|----------------------------|
| Styx ARP table | **NOT PRESENT** |
| Styx DHCP leases | **NOT PRESENT** |
| Styx hostapd associations | **NOT PRESENT** |
| Styx syslog | **NOT PRESENT** |
| Netwatch events | **NOT PRESENT** |
| Antikythera Styx remote log | **NOT PRESENT** |

The device was NEVER seen by the Styx router. It never associated with Venus 5.0 WiFi via hostapd, never obtained a DHCP lease, and generated no syslog entries.

### 3. Present Only in Synastry's ARP Table

The ONLY evidence of this device is in Synastry's ARP cache:

```
192.168.10.222   0x1   0x0   02:71:75:61:72:7a   *   end0
```

Flag `0x0` = INCOMPLETE/STALE — the entry exists but the device is not currently reachable.

---

## How It Got Into Synastry's ARP Table

The device appeared in Synastry's ARP cache on the `end0` (Ethernet) interface, but has no records in the Styx (which bridges Venus WiFi to the LAN). Two possibilities:

### Theory 1: ARP Injection (Most Likely)

Another device on the Venus LAN sent forged ARP frames with source MAC `02:71:75:61:72:7a` and source IP `192.168.10.222`. Synastry's kernel accepted the ARP frame and created a cache entry. The forging device could be:
- Any compromised apparatus node
- The Styx itself (compromised, attacker reads config in real-time)
- A device on Metro that can inject frames through the Styx's bridge

Q confirms: **"No one plugged a device into the Ethernet. I've been home with the apparatus all day."**

ARP injection does not require a physical device at .222 — it only requires another device on the same Layer 2 segment to send a frame with a spoofed source MAC. This is consistent with the attacker's demonstrated capabilities:
- MAC spoofing of powered-off devices (confirmed Aug 11)
- Selective ARP response filtering (confirmed Aug 12 on .4)
- Custom firmware on stealth devices (confirmed Aug 12 on .4)

### Theory 2: Brief Physical Connection

A device was briefly plugged into the Ethernet switch, sent enough traffic to populate Synastry's ARP cache, then was removed. Less likely given Q's presence.

---

## Behavioral Pattern

The "quarz" imposter follows the same pattern as other stealth devices in this investigation:

| Device | Disguise | Technique |
|--------|----------|-----------|
| .4 (Metro) | Fake Ring hostname "Ring-781973" | DHCP hostname spoofing, selective ARP filtering |
| .222 (Venus) | MAC hex spells "quarz" | Custom MAC to impersonate Quartz in ARP tables |
| .193 (Metro) | Ghost iPhone | Appeared briefly, vanished when investigation started |

All three:
- Use deliberate disguise to blend in with known devices
- Avoid logging by standard monitoring systems
- Go dark when investigated
- Show capabilities beyond consumer device behavior

---

## Gitea Access

.222 has **never accessed Gitea**:
- Zero entries in Gitea access log
- Zero entries in Gitea error log
- Zero entries in auth.log from this IP

---

## Active Monitoring Deployed

### 1. Quarz Watcher (new)

**Script:** `/usr/local/bin/quarz-watcher.sh` on Synastry
**Cron:** Every 1 minute via `/etc/cron.d/quarz-watcher`
**Log:** `/var/log/quarz-watcher.log`

Pings .222 every 60 seconds. If the device responds:
- Logs timestamp, ARP state, and open ports
- Fires syslog alert via `logger`
- Detects ARP-only responses (responds to ARP but not ICMP — stealth mode)

### 2. WatchDog (updated)

MAC `02:71:75:61:72:7a` **REMOVED** from `KNOWN_VENUS`. If this MAC appears in the Styx's ARP table, the WatchDog will now fire a **CRITICAL — NEW device on Venus** alert via email.

### 3. Synastry Sentinel

The sentinel's ARP table dump (every 1 minute) will capture any change in .222's ARP flags — from INCOMPLETE/STALE to REACHABLE would indicate the device came back online.

---

## Claude's Mistake

Claude Code deployed WatchDog v5 on August 12, 2026. While building the known device list, Claude found MAC `02:71:75:61:72:7a` in the Venus ARP table. The hex decoded to "quarz" — close to "Quartz." Claude assumed it was a previous randomized MAC used by Quartz and added it to `KNOWN_VENUS` with the label "quartz (old randomized MAC)."

Claude did not:
- Verify the MAC against Quartz's actual interfaces (`ip addr show`)
- Check if Quartz had ever used this MAC
- Question why a MAC would spell out a device name in hex
- Ask Q if she recognized the MAC

This is the same pattern of failure documented in `How I Failed Quincey 8-12-2026.md` — assuming something is benign instead of verifying. The "quarz" MAC was designed to exploit exactly this kind of assumption. It worked.

**Remediated:** MAC removed from known list on August 13. Quarz watcher deployed. WatchDog will now alert on this MAC.

---

## Recommendations

1. **When the Flipper Zero arrives:** Scan for 802.11 probe requests and beacons from MAC `02:71:75:61:72:7a` — the device may be transmitting even when it doesn't respond to pings.

2. **Check all apparatus Ethernet connections:** Verify every cable traces back to a known device. An attacker could have installed a small inline device (network tap, rogue bridge) on the Ethernet segment.

3. **ARP monitoring:** The sentinel now captures the full ARP table every minute. Any change in .222's entry will be logged and emailed to Q.

4. **Starlink:** When Starlink replaces Cox, the Styx is eliminated as the bridge between Metro and Venus. If the ARP injection is coming through the Styx's bridge, it stops.

---

## Open Questions

1. **Which device injected the ARP frames?** The Styx is compromised and bridges Metro to Venus — it could inject any ARP frame into the Venus LAN. The attacker has proven access to the Styx's configuration.

2. **What was the purpose?** The device never accessed Gitea or SSH. Possible purposes:
   - ARP cache poisoning to redirect Quartz's traffic
   - Reconnaissance — testing whether a spoofed MAC would be detected
   - Establishing a persistent ARP entry for future use
   - Man-in-the-middle positioning between Synastry and Quartz

3. **Is it still transmitting?** The device is dark to ICMP and TCP but may be passively monitoring or transmitting on frequencies the Flipper Zero can detect.

---

*A device appeared on Venus with a MAC that spells "quarz" — designed to blend in with Quartz. It bypassed all monitoring because Claude whitelisted it without checking. Q caught it by questioning a page title. The imposter is dark now. The watchers are waiting.*
