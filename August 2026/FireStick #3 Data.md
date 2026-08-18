# FireStick #3 Data — August 11, 2026

**Extracted:** 2026-08-11 ~03:45 PDT
**Device:** Amazon Fire TV Stick, model AFTSSS "sheldonp"
**Serial:** G4N1EL13330609K9
**Fire OS:** 7.0 (Android 9), build PS7715.5586N
**Registered to:** robert t. lee (Q's father), with Kid profile
**Location:** Parents' room — used by Q's parents for TV before bed
**Status:** WIPED AND LOCKED DOWN

---

## Compromise Summary

Fire Stick #3 was running proxy services on ports 1080 (SOCKS), 8888 (HTTP), and 9080 via the Fire VPN app (`com.azone.firevpn`). ADB over network was enabled on port 5555. Downloader (sideloading tool) was updated as recently as August 3, 2026 — 8 days before discovery.

This is the second compromised Fire Stick in Q's household (Fire Stick #2 had BrightData + 12 VPNs). Same pattern: VPN apps with proxy functionality, Downloader for sideloading, ADB enabled.

---

## Installed Apps (Non-System)

| App | Install Date | Last Updated |
|-----|-------------|--------------|
| com.netflix.ninja (Netflix) | 2025-08-22 | 2026-08-07 |
| com.esaba.downloader (Downloader) | 2025-08-23 | **2026-08-03** |
| tv.pluto.android (Pluto TV) | 2025-08-23 | 2026-05-25 |
| com.xumo.xumo.tv (Xumo) | 2025-08-23 | 2026-07-22 |
| com.roku.web.trc (Roku Channel) | 2025-08-23 | 2026-02-20 |
| com.future.moviesByFawesomeAndroidTV (Fawesome) | 2025-08-23 | 2026-07-04 |
| free.androidtv.vpn.proxy.purevpn (PureVPN) | 2025-09-08 | 2026-06-22 |
| digital.wmt.mobile.androidtv.themw (Walmart) | 2025-09-13 | 2025-12-18 |
| com.azone.firevpn (Fire VPN) | 2025-09-13 | 2025-09-13 |
| com.tubitv.ott (Tubi) | 2025-09-17 | 2026-07-24 |
| com.fox5vegas.ott (Fox 5 Vegas) | 2025-09-20 | 2026-04-28 |
| com.peacock.peacockfiretv (Peacock) | 2025-12-22 | 2026-07-24 |
| com.fox.foxone (Fox) | 2026-06-13 | 2026-08-07 |

---

## Proxy Ports (When Connected to Metro)

| Port | Protocol | Source App |
|------|----------|-----------|
| 1080 | SOCKS proxy | com.azone.firevpn (Fire VPN) |
| 8888 | HTTP proxy | com.azone.firevpn (Fire VPN) |
| 9080 | Additional proxy | com.azone.firevpn (Fire VPN) |
| 8009 | Google Cast | System |
| 55442-55445 | Amazon protocol | System |

---

## ADB Configuration

```
adb_enabled: 1
adb_tcp_port: 5555
```

ADB over network was enabled. Unlike Fire Stick #2, this device accepted our ADB connection directly after Q enabled Developer Options — no pre-existing unauthorized pairing was detected (though without root, the ADB keys file cannot be verified).

---

## VPN Usage Statistics

| App | Total Days Active | Screen Time | Last Used |
|-----|------------------|-------------|-----------|
| PureVPN | 68 days | 10d 21h | Active |
| Fire VPN | 67 days | 10d 21h | Active |

Both VPN apps were ACTIVELY used for over 10 days of cumulative screen time across 67-68 days. This is consistent with Q's father using VPNs for free TV streaming.

---

## Fire VPN Proxy Service

```
com.azone.firevpn/com.azone.mizuvpn.proxy.ProxyService
  Permission: android.permission.BIND_VPN_SERVICE
  Version: 1.5.16
  Installed via: Amazon App Store (com.amazon.venezia)
```

The Fire VPN app contains a `ProxyService` class (`com.azone.mizuvpn.proxy.ProxyService`) that runs a local SOCKS and HTTP proxy as part of its VPN routing. This is what opened ports 1080, 8888, and 9080 on the network.

**Fire VPN APK preserved at:** `/tmp/firestick3-firevpn.apk` (15.6 MB) for offline analysis.

---

## Account Details

- **Primary:** robert t. lee (registered 2025-07-16)
- **Kid profile:** created 2025-08-22
- **Account IDs redacted** by Amazon's DHCP privacy masking in dumpsys output

---

## Network Configuration

- **Saved Wi-Fi:** metro1 only (2.4 GHz, WPA3 SAE)
- **Default DNS:** 8.8.4.4, 8.8.8.8 (Google — not using rogue DNS)
- **Bluetooth:** Paired with `robert's FireTVStick` (20:FE:00:6E:A9:7A)
- **Bonded BLE device:** One unknown LE device (MAC redacted)

---

## Nearby Networks Scanned by Device

The device's Wi-Fi logs show it scanned and catalogued 50+ nearby BSSIDs including:
- Cox router APs (ce:f3:c8:73:98:41/42/45/46/47)
- Multiple neighbor APs
- Several hidden SSIDs

---

## Downloader — Updated August 3, 2026

The Downloader app (com.esaba.downloader) was updated to the latest version on August 3, 2026 — 8 days before discovery. The Download folder was empty at extraction time (downloaded APKs are deleted after sideloading).

This means the sideloading tool was actively maintained and potentially used recently.

---

## Lockdown Actions Taken

| Action | Result |
|--------|--------|
| Fire VPN killed | Force-stopped |
| PureVPN killed | Force-stopped |
| Downloader killed | Force-stopped |
| Fire VPN disabled | disabled-user |
| PureVPN disabled | disabled-user |
| Downloader disabled | disabled-user |
| Fire VPN data cleared | Success |
| PureVPN data cleared | Success |
| Downloader data cleared | Success |
| ADB Wi-Fi disabled | Setting cleared |
| Proxy ports verified closed | 1080/8888/9080 ALL CLOSED |

---

## Comparison: Fire Stick #2 vs Fire Stick #3

| | Fire Stick #2 (purse) | Fire Stick #3 (parents room) |
|---|---|---|
| Model | AFTMM "mantis" | AFTSSS "sheldonp" |
| BrightData | **YES** (13 months) | NO |
| VPN apps | 12 (mass install session) | 2 (PureVPN + Fire VPN) |
| Proxy ports | 1080, 8888, 9080 (BrightData) | 1080, 8888, 9080 (Fire VPN) |
| ADB pre-paired | **YES** (unauthorized keys) | Not detected |
| Downloader | YES (updated Jul 31) | YES (updated **Aug 3**) |
| Origin | **Mom got from a "friend"** | Unknown — needs verification |
| Account | robert t. lee | robert t. lee |

---

*This document records all data extracted from Fire Stick #3 (AFTSSS, serial G4N1EL13330609K9) on August 11, 2026. The device was running Fire VPN proxy services on ports 1080/8888/9080 and had ADB network debugging enabled. All VPN apps, Downloader, and proxy services have been killed, disabled, and their data cleared.*
