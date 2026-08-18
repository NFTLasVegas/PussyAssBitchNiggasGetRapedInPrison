# DHCP Hardening Playbook

**Purpose:** Harden the Styx router's DHCP configuration to prevent DHCP-based attacks including starvation, rogue servers, lease hijacking, and stealth devices.

**Why:** The apparatus investigation uncovered an unidentified QNAP NAS device (192.168.10.197, MAC 24:5E:BE:77:BF:FD) on the Styx LAN that the operator did not recognize. The device has no DHCP hostname set (stealth — harder to identify in the lease table), actively renews its lease every 5-6 hours, and was never provisioned by the operator. Additionally, the Styx's DHCP server (dnsmasq) currently assigns addresses to ANY device with the Wi-Fi password, with no MAC filtering, no lease limits, and no alerting on unknown devices.

**Evidence cited:**
- QNAP device at .197: `Pussy Ass Bitch Niggas Get Raped In Prison/August 2026/System Idle Sniffer Completed 8-7-2026.md`
- Deauth attack (attacker trying to get on the network): `research/M5_network_forensics_2026-08-05.md`
- DHCP leases showing unknown devices: Styx `/tmp/dhcp.leases`

---

## Current State

The Styx router (GL.iNet GL-MT3600BE, OpenWrt) runs dnsmasq for DHCP on 192.168.10.0/24. Current configuration:
- Assigns any IP in the pool to any device with the PSK
- No MAC allowlist
- No unknown-device alerting
- Static leases for apparatus nodes (Dragon, Quartz, Antikythera, Synastry, ARES Dynasty)
- Dynamic leases for operator devices (M5, M2, iPhone)
- Unknown device at .197 (QNAP) obtained a lease without question

---

## Hardening Measures

### 1. Static Lease Binding for All Known Devices

Bind every known apparatus MAC to a fixed IP. Only these MACs get leases.

```bash
ssh root@192.168.10.1

# Add static leases for every known device via uci
# Format: uci add dhcp host; uci set dhcp.@host[-1].mac=XX:XX:XX:XX:XX:XX; etc.

# Apparatus nodes (already have static leases — verify)
uci show dhcp | grep host

# Operator devices (add static bindings)
# M5 MacBook
uci add dhcp host
uci set dhcp.@host[-1].mac='26:4A:71:F8:58:7F'
uci set dhcp.@host[-1].ip='192.168.10.202'
uci set dhcp.@host[-1].name='M5'

# M2 MacBook
uci add dhcp host
uci set dhcp.@host[-1].mac='DE:6F:C6:1A:27:9A'
uci set dhcp.@host[-1].ip='192.168.10.194'
uci set dhcp.@host[-1].name='M2'

# iPhone
uci add dhcp host
uci set dhcp.@host[-1].mac='52:9D:DD:95:B8:1E'
uci set dhcp.@host[-1].ip='192.168.10.241'
uci set dhcp.@host[-1].name='iPhone'

uci commit dhcp
/etc/init.d/dnsmasq restart
```

### 2. Limit the DHCP Pool Size

Reduce the available address pool so unknown devices can't easily get an IP.

```bash
# Check current pool
uci show dhcp | grep start
uci show dhcp | grep limit

# Set a tight pool — only enough for known devices + small buffer
uci set dhcp.lan.start='190'    # Start at .190
uci set dhcp.lan.limit='20'     # Only 20 addresses (.190-.209)
# Known static leases are outside this pool and always work

uci commit dhcp
/etc/init.d/dnsmasq restart
```

### 3. Enable DHCP Logging

dnsmasq can log every DHCP event — every request, offer, and acknowledgment.

```bash
# Enable query and DHCP logging
uci set dhcp.@dnsmasq[0].logdhcp='1'
uci commit dhcp
/etc/init.d/dnsmasq restart

# Now every DHCP event appears in logread:
# DHCPDISCOVER, DHCPOFFER, DHCPREQUEST, DHCPACK, DHCPNAK
```

### 4. Unknown Device Alerting

Create a script that checks the DHCP lease table for unknown MACs and alerts.

```bash
# Create the script on the Styx
cat > /usr/local/bin/dhcp-watchdog.sh << 'DHCPWATCH'
#!/bin/sh
# DHCP Watchdog — alerts on unknown MAC addresses getting DHCP leases

KNOWN_MACS="
26:4a:71:f8:58:7f
de:6f:c6:1a:27:9a
52:9d:dd:95:b8:1e
4a:21:74:3b:11:b2
00:48:54:21:5b:fb
6c:cf:39:00:97:cb
02:71:75:61:72:7a
2c:4d:54:42:a9:92
00:07:32:d2:02:22
30:52:53:04:bc:ab
"

LOG=/tmp/dhcp-watchdog.log

while read line; do
    MAC=$(echo "$line" | awk '{print $2}')
    IP=$(echo "$line" | awk '{print $3}')
    NAME=$(echo "$line" | awk '{print $4}')

    if ! echo "$KNOWN_MACS" | grep -qi "$MAC"; then
        TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
        echo "$TIMESTAMP UNKNOWN DEVICE: MAC=$MAC IP=$IP NAME=$NAME" >> $LOG
    fi
done < /tmp/dhcp.leases
DHCPWATCH

chmod +x /usr/local/bin/dhcp-watchdog.sh

# Run every 5 minutes
echo "*/5 * * * * /usr/local/bin/dhcp-watchdog.sh" >> /etc/crontabs/root
/etc/init.d/cron restart
```

### 5. Consider MAC Address Filtering (Optional — Aggressive)

OpenWrt supports MAC allowlisting where ONLY known MACs can connect to Wi-Fi.
This is the nuclear option — any new device you bring home won't connect until
you add its MAC to the allowlist.

```bash
# This is configured in the wireless config, not DHCP
# Only do this if you want maximum security and accept the inconvenience

uci set wireless.ra0.macfilter='allow'
uci set wireless.ra0.maclist='26:4A:71:F8:58:7F DE:6F:C6:1A:27:9A 52:9D:DD:95:B8:1E'
# Add all known MACs...
uci commit wireless
wifi reload
```

**WARNING:** This will kick off any device not in the allowlist immediately. Make sure
ALL your device MACs are listed before enabling.

---

## Immediate Action: Investigate the QNAP Device

Before applying the hardening, the QNAP NAS at 192.168.10.197 needs to be identified
or removed from the network. If the operator does not own a QNAP NAS, this device is
unauthorized and should be blocked.

```bash
# Block the QNAP MAC on the Styx
uci add dhcp mac
uci set dhcp.@mac[-1].mac='24:5E:BE:77:BF:FD'
uci set dhcp.@mac[-1].action='deny'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

---

## Verification Checklist

- [ ] All known device MACs have static lease bindings
- [ ] DHCP pool size reduced to minimum required
- [ ] DHCP logging enabled (logdhcp=1)
- [ ] DHCP watchdog script installed and running via cron
- [ ] QNAP device identified or blocked
- [ ] Unknown device alerting tested (connect a new device, verify alert fires)

---

## Future: Protocol Update Required

When the following devices are onboarded, their MACs must be added to:
1. The DHCP static lease table
2. The DHCP watchdog known-MAC list
3. The MAC allowlist (if enabled)

Devices pending:
- AphroQite Dynasty
- Godlike Bloodline
- DGX Spark Q1
- DGX Spark Q2
