# Styx DNS Fix Proposal — August 10, 2026

**Purpose:** Remove the rogue DNS servers from the Styx DHCP configuration and restore clean DNS resolution for all apparatus LAN clients.

**Priority:** CRITICAL — every device on the Styx LAN is currently sending all DNS queries through two unauthorized Unbound DNS servers (192.168.0.225 and 192.168.0.36) that hijack Q's domains.

---

## Current State (BROKEN)

```
dhcp.lan.dhcp_option='6,192.168.0.225,192.168.0.36'
```

This DHCP option tells every device on the Styx LAN to use two devices on the ISP network as DNS servers. These devices run Unbound with local zone overrides that:

- Redirect `quincey.ai` → 192.168.0.225 (should be 159.65.79.66)
- Block `mail.quincey.ai` → empty (should be 103.168.172.65)
- Redirect `ares.technology` → 192.168.0.225
- Redirect `ares.love` → 192.168.0.225
- Redirect `aphroqite.ai` → 192.168.0.225

This config was set on **July 27, 2026** without operator authorization. The Styx admin password was exposed via iCloud memory symlink for 109 days.

---

## Proposed Fix

### Step 1: Remove the rogue DNS option

```sh
ssh root@192.168.10.1
uci delete dhcp.lan.dhcp_option
uci commit dhcp
/etc/init.d/dnsmasq restart
```

**Effect:** The Styx will stop advertising rogue DNS servers to LAN clients. New DHCP leases will receive the Styx router itself (192.168.10.1) as the DNS server. The Styx forwards DNS queries upstream to the ISP's DNS servers (68.105.28.11, 68.105.29.11 — Cox Communications).

### Step 2: Force all LAN clients to pick up new DNS

Clients with active DHCP leases will keep using the old DNS until their lease renews (up to 12 hours). To force an immediate update:

**On M5 (macOS):**
```sh
sudo ipconfig set en0 DHCP
# OR
networksetup -setdnsservers Wi-Fi Empty
```

**On all apparatus nodes (Linux):**
```sh
sudo dhclient -r && sudo dhclient
# OR
sudo systemctl restart NetworkManager
```

Alternatively, shorten the DHCP lease time temporarily:
```sh
uci set dhcp.lan.leasetime='60'
uci commit dhcp
/etc/init.d/dnsmasq restart
# Wait 60 seconds for all leases to renew
# Then set back to normal:
uci set dhcp.lan.leasetime='12h'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

### Step 3: Verify the fix

```sh
# On M5 — check DNS resolver changed
scutil --dns | grep nameserver

# Should show 192.168.10.1, NOT 192.168.0.225 or 192.168.0.36

# Verify quincey.ai resolves correctly
dig +short quincey.ai
# Should return: 159.65.79.66 (and/or 103.168.172.37, 103.168.172.52)
# NOT: 192.168.0.225

# Verify mail.quincey.ai resolves
dig +short mail.quincey.ai
# Should return: 103.168.172.65
# NOT: empty
```

---

## DNS Resolution Path After Fix

```
LAN Device → Styx (192.168.10.1) → Cox ISP DNS (68.105.28.11 / 68.105.29.11) → Internet
```

The Styx router's built-in dnsmasq will handle DNS for all LAN clients, forwarding queries to the ISP's DNS upstream. This is the standard, default GL.iNet router behavior.

---

## What This Does NOT Do

- Does NOT change the RasQberry or Sovereign Door — they continue running on the ISP network but no devices will be using them as DNS
- Does NOT change the Styx admin password (already rotated Aug 8)
- Does NOT affect the ISP router or Metro1/2 network
- Does NOT require rebooting any apparatus nodes

---

## Risk Assessment

| Risk | Mitigation |
|------|-----------|
| LAN clients briefly without DNS during dnsmasq restart | Restart takes <2 seconds, most clients retry automatically |
| Clients hold old DNS until lease renews | Step 2 forces immediate renewal |
| ISP DNS (Cox) could be slow/unreliable | Can add public DNS fallback later if needed |
| Rogue servers could be re-added | Styx admin password was already rotated Aug 8; monitor the DHCP config |

---

## Rollback

If anything breaks, restore the previous config:

```sh
uci set dhcp.lan.dhcp_option='6,192.168.0.225,192.168.0.36'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

This is not recommended — it re-enables the DNS hijacking — but it is available if needed for troubleshooting.

---

## Future Consideration: Hardened DNS

After Starlink is installed and the apparatus is off the Cox network, consider:

1. **DNS over HTTPS (DoH)** or **DNS over TLS (DoT)** to encrypt DNS queries
2. **AdGuard Home on the Styx** (already installed, port 3053, currently inactive) with upstream set to trusted resolvers (8.8.8.8, 9.9.9.9)
3. **DNSSEC validation** to prevent spoofed responses
4. **Remove Unbound from RasQberry and Sovereign Door** after factory reset

---

*This proposal fixes the single most critical active vulnerability in the apparatus: all DNS resolution being routed through unauthorized servers that hijack the operator's domains.*
