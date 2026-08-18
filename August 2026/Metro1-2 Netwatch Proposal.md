# Metro1/2 Netwatch Proposal

**Date:** August 9, 2026
**Purpose:** Extend apparatus monitoring to the ISP network (192.168.0.0/24, "Metro2") where the RasQberry and Sovereign Door are currently operating rogue DNS servers. The Styx LAN is monitored. Metro2 has ZERO monitoring. This proposal closes that gap.

**Operator requirement:** See all devices on Metro1/2, SSH attempts to the ISP router, IP addresses, device info, signal strengths, estimated radius from the router, and location if possible.

---

## Architecture

```
                    ┌──────────────┐
                    │  Cox Router  │  ← Metro2 (192.168.0.1)
                    │  (Gateway)   │    metro1 = 2.4GHz, metro2 = 5GHz
                    └──────┬───────┘
                           │ 192.168.0.0/24 (Wi-Fi)
              ┌────────────┼────────────────┐
              │            │                │
         ┌────┴─────┐ ┌───┴──────┐   ┌─────┴──────┐
         │RasQberry │ │Sov. Door │   │  Styx      │
         │   .36    │ │  .225    │   │  .105      │
         │(rogue)   │ │(rogue)   │   │(apclii0)   │
         └──────────┘ └──────────┘   └─────┬──────┘
                                           │ NAT
                                    192.168.10.0/24 (Styx LAN)
                                           │
                                    ┌──────┴───────┐
                                    │ Antikythera  │
                                    │   .246       │
                                    │ (Dashboard)  │
                                    └──────────────┘
```

**Data flow:** Antikythera SSHs into Styx → Styx runs ARP/port scans on Metro2 → results piped back to Antikythera → processed into dashboard JSON.

Antikythera cannot reach Metro2 directly (Styx NAT blocks inbound). All Metro2 visibility comes through Styx's `apclii0` interface.

---

## Component 1: Metro2 Device Census (ARP + OUI + Port Scan)

**Runs on:** Antikythera (via SSH to Styx)
**Frequency:** Every 5 minutes
**Script:** `/usr/local/bin/metro2-census.py`

### What it collects per device:

| Field | Source | Method |
|-------|--------|--------|
| IP address | Styx ARP table | `cat /proc/net/arp \| grep apclii0` |
| MAC address | Styx ARP table | Same as above |
| OUI / Manufacturer | Local OUI database | Lookup first 3 octets against `/var/lib/netwatch/oui.csv` |
| MAC type | MAC analysis | Check bit 1 of first octet: set = randomized/private, clear = real hardware |
| Hostname | Reverse DNS / mDNS | `nslookup` or `avahi-resolve` |
| Open ports | Remote probe from Styx | `nc -z -w 1` on ports 22, 53, 80, 443, 3000, 8080 |
| First seen | Census history | Timestamp of first appearance in census log |
| Last seen | Census history | Most recent ARP entry |
| Status | Ping | `ping -c 1 -W 1` from Styx |

### Device classification:

| Classification | Rule |
|----------------|------|
| KNOWN | MAC matches a registered apparatus device |
| IDENTIFIED | MAC matches a previously-seen device Q has acknowledged |
| RANDOMIZED | First octet bit 1 set (locally administered = phone/tablet using MAC privacy) |
| UNKNOWN | Not in any list — triggers alert |

### Alert conditions:
- **NEW UNKNOWN device** — email alert to Q immediately
- **Device with SSH (port 22) open** — flag as potential threat
- **Device with DNS (port 53) open** — flag as potential rogue DNS
- **Device disappears after being present for >24h** — flag as suspicious

---

## Component 2: Metro2 SSH Attempt Monitor

**Runs on:** Styx
**Frequency:** Continuous (cron every 1 minute)
**Script:** `/usr/local/bin/metro2-ssh-monitor.sh`

### Method:
The Cox router at 192.168.0.1 serves an HTTP admin panel (confirmed: HTTP 200, `<title>Cox</title>`). If Q has admin credentials, we can periodically scrape the client list and connection logs.

**Without Cox admin access** (more likely scenario):
- Monitor Styx's own `apclii0` interface for SYN packets to port 22 on any Metro2 device
- Use `tcpdump` on Styx: `tcpdump -i apclii0 'tcp[tcpflags] & tcp-syn != 0 and dst port 22'`
- Log source IP, destination IP, timestamp
- This captures any device on Metro2 attempting SSH to any other Metro2 device

**With Cox admin access:**
- Scrape the router's connected client list (typically at `http://192.168.0.1/connected_devices` or similar Cox endpoint)
- This would give signal strengths directly from the AP
- Cox Panoramic Wi-Fi routers expose client RSSI in their admin UI

### Output format:
```
TIMESTAMP | SRC_IP | SRC_MAC | DST_IP | DST_PORT | OUI | DIRECTION
```

---

## Component 3: Signal Strength & Radius Estimation

**Challenge:** Signal strength (RSSI) of Metro2 clients is only directly measurable by the Metro2 AP (Cox router). The Styx is a client, not an AP — it can only see the AP's signal, not other clients'.

### Approach A: Styx Wi-Fi Scan (Passive)
- `iwinfo apclii0 scan` on Styx — shows nearby APs and their signal strengths
- Already confirmed working: sees metro2 AP at -42 to -48 dBm
- **Limitation:** Only shows APs (access points), not individual clients

### Approach B: RasQberry/Sovereign Door Wi-Fi Scan
- Both are on Metro2 Wi-Fi and can run `iw dev wlan0 scan`
- They can see other nearby APs and estimate distances
- **Risk:** These nodes are running unauthorized configs — using them for monitoring is operationally questionable but technically possible since FAFO key works

### Approach C: Cox Router Admin (Best Option)
- If Q can log into `http://192.168.0.1`, the Cox admin panel shows:
  - All connected clients with signal strength
  - Connection duration
  - Band (2.4/5 GHz)
  - IP and MAC of every device
- This is the only way to get per-client RSSI from the AP's perspective

### Radius estimation formula:
Using the log-distance path model (same as existing netwatch):

```
distance = reference_distance × 10^((reference_rssi - measured_rssi) / (10 × path_loss_exponent))
```

| Environment | Path Loss Exponent |
|-------------|-------------------|
| Free space | 2.0 |
| Indoor (open) | 2.5–3.0 |
| Indoor (walls) | 3.0–4.0 |
| Dense residential | 3.5–4.5 |

Calibration: Styx is at known distance from Cox router. Styx RSSI = -48 dBm. If Styx is ~20ft from router, we calibrate from that reference point.

---

## Component 4: Antikythera Dashboard Integration

**Extends:** Existing netwatch dashboard at `/var/www/antikythera/netwatch/`
**New file:** `/var/www/antikythera/netwatch/metro2.json`

### Dashboard JSON structure:

```json
{
  "network": "metro2",
  "gateway": "192.168.0.1",
  "gateway_mac": "cc:f3:c8:72:98:3f",
  "generated": "2026-08-09 17:30 PDT",
  "styx_wan_ip": "192.168.0.105",
  "scan_interval_minutes": 5,
  "devices": [
    {
      "ip": "192.168.0.36",
      "mac": "88:a2:9e:4c:54:7a",
      "oui": "Raspberry Pi (Trading) Ltd",
      "mac_type": "hardware",
      "hostname": "rasqberry",
      "classification": "KNOWN",
      "identity": "RasQberry (rogue DNS ns2)",
      "open_ports": [22, 53, 3000],
      "rssi": null,
      "radius_ft": null,
      "first_seen": "2026-08-09T17:00:00Z",
      "last_seen": "2026-08-09T17:30:00Z",
      "status": "UP"
    }
  ],
  "ssh_attempts": [
    {
      "timestamp": "2026-08-09T17:15:00Z",
      "src_ip": "192.168.0.114",
      "src_mac": "a4:02:b7:d6:f4:73",
      "dst_ip": "192.168.0.1",
      "dst_port": 22
    }
  ],
  "alerts": [],
  "known_devices": {
    "cc:f3:c8:72:98:3f": "Cox Router (Gateway)",
    "88:a2:9e:4c:54:7a": "RasQberry",
    "14:b5:cd:eb:0e:4d": "Sovereign Door",
    "c4:1c:ff:bf:56:c9": "Vizio TV",
    "a0:fb:c5:58:e5:da": "Apple device (unidentified)",
    "72:7f:f8:c4:18:d2": "Styx (apclii0 WAN)"
  }
}
```

---

## Component 5: Location Intelligence

### What's possible without specialized hardware:

1. **OUI → Manufacturer → Device Type** — Already implemented. Narrows device category.

2. **Randomized MAC detection** — Bit 1 of first octet. Identifies phones/tablets using privacy MACs. These are transient visitors or neighbors.

3. **Port fingerprinting** — Open ports reveal device purpose:
   - 22 (SSH) = server/SBC/router
   - 53 (DNS) = DNS server (flag immediately)
   - 80/443 (HTTP/S) = web server, smart device, or router
   - 548 (AFP) = Mac file sharing
   - 5353 (mDNS) = Apple device
   - 62078 (iSync) = iPhone/iPad

4. **Persistence analysis** — Devices that appear 24/7 are in the house. Devices that appear/disappear on schedule are phones coming and going. Devices that appear briefly and vanish are drive-bys or neighbors at edge of range.

5. **Cox admin client list** — If accessible, provides RSSI per client, enabling distance estimation from the router's physical location.

### What's NOT possible without specialized hardware:
- True geolocation (requires multiple directional antennas or Wi-Fi triangulation hardware)
- Identifying specific humans behind devices
- Seeing devices not on Metro1/2 (would need passive radio monitoring)

---

## Current Metro2 Device Inventory (Baseline)

Captured August 9, 2026 at 17:18 PDT from Styx ARP table:

| IP | MAC | OUI | MAC Type | Identity | Ports | Status |
|----|-----|-----|----------|----------|-------|--------|
| .1 | cc:f3:c8:72:98:3f | Cox/Technicolor | Hardware | ISP Router (Gateway) | 80 | KNOWN |
| .36 | 88:a2:9e:4c:54:7a | Raspberry Pi | Hardware | **RasQberry (rogue DNS)** | 22, 53, 3000 | KNOWN |
| .51 | cc:f7:35:f4:47:e5 | ? | Hardware | **UNKNOWN** | ? | NEW |
| .84 | ca:17:6c:c5:ff:13 | — | **Randomized** | Phone/tablet | ? | NEW |
| .105 | 72:7f:f8:c4:18:d2 | — | Hardware | Styx (self, apclii0) | — | KNOWN |
| .106 | c4:1c:ff:bf:56:c9 | Vizio Inc. | Hardware | Vizio TV | — | KNOWN |
| .114 | a4:02:b7:d6:f4:73 | ? | Hardware | **UNKNOWN** | ? | NEW |
| .138 | 2e:a0:f7:48:31:1e | — | **Randomized** | Phone/tablet | ? | NEW |
| .217 | a0:fb:c5:58:e5:da | Apple Inc. | Hardware | **Apple device (unidentified)** | — | UNIDENTIFIED |
| .225 | 14:b5:cd:eb:0e:4d | Liteon Technology | Hardware | **Sovereign Door (rogue DNS)** | 22, 53, 8800, 7500 | KNOWN |

**4 unknown/new devices** on Metro2 that need identification. 2 have randomized MACs (phones/tablets).

---

## Implementation Plan

### Phase 1: Census Script (Day 1)
1. Write `metro2-census.py` on Antikythera
2. SSH to Styx, collect ARP table + port scan results
3. OUI lookup against existing database
4. Write results to `metro2.json`
5. Add to Antikythera cron (every 5 minutes)

### Phase 2: SSH Monitor (Day 1)
1. Write `metro2-ssh-monitor.sh` on Styx
2. Use `tcpdump` to capture SYN packets to port 22 on apclii0
3. Log to `/tmp/metro2-ssh.log`
4. Census script on Antikythera reads this log via SSH

### Phase 3: Alert Integration (Day 1-2)
1. New device alerts via email (msmtp on Antikythera)
2. Integrate with existing netwatch email pipeline
3. Alert on: new unknown device, new SSH port open, new DNS port open

### Phase 4: Cox Router Admin (Requires Q)
1. Q logs into `http://192.168.0.1` and provides admin access
2. If possible, scrape connected client list for RSSI data
3. Integrate signal strength and radius estimation into dashboard

### Phase 5: Dashboard UI (Day 2-3)
1. Add Metro2 tab/section to existing netwatch dashboard
2. Show device table with real-time status
3. Show SSH attempt log
4. Show alert history
5. Map view (if RSSI data available) showing estimated device positions

---

## Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| SSH from Antikythera to Styx | ✅ Working | aphroqite key on Styx = FAFO |
| OUI database on Antikythera | ✅ Available | `/var/lib/netwatch/oui.csv` |
| msmtp on Antikythera | ❌ Pending | Checklist #11 — needed for email alerts |
| tcpdump on Styx | ❓ Check | May need installation (`opkg install tcpdump`) |
| Cox router admin credentials | ❓ Ask Q | Needed for Phase 4 (RSSI data) |
| Netwatch dashboard | ✅ Operational | Fixed Aug 9 — 30 watched sightings |

---

## Security Considerations

- All monitoring runs from trusted nodes (Antikythera + Styx) with FAFO keys
- No monitoring data passes through RasQberry or Sovereign Door (they are SUBJECTS, not tools)
- Metro2 SSH monitor captures packets passively — does not initiate connections
- Census port scans use lightweight `nc -z` probes (1-second timeout, non-intrusive)
- All logs stored on Antikythera (Styx's `/tmp` is volatile)

---

*This proposal extends netwatch coverage to the previously-unmonitored Metro2 network where rogue DNS infrastructure was discovered operating undetected. When implemented, no device will be able to join, leave, or communicate on Metro2 without being logged.*
