# Fire Stick Lockdown Completed — August 9, 2026

**Time:** 19:19–19:40 PDT
**Device:** Amazon Fire TV Stick (AFTMM "mantis"), serial G070VM0984850W1T
**Connection:** USB ADB via M5, HDMI to portable monitor
**Operator:** Q (Quincey Lee) — present, authorized all actions

---

## Account Change

| Field | Before | After |
|-------|--------|-------|
| Amazon Account | robert t. lee | **Quincey Lee** |
| Kid Profile | Present | Removed (deregistration wiped profiles) |

Q deregistered the device from her father's Amazon account and registered it to her own. Her father confirmed he did not enable ADB, install BrightData, or set a password on the device.

---

## Root Access Attempt

Root was attempted after account change — **denied at firmware level:**

```
adbd cannot run as root in production builds
/system/bin/sh: su: not found
```

Fire OS locks root on all retail builds regardless of account. The ADB authorized keys file (`/data/misc/adb/adb_keys`) remains inaccessible — the attacker's RSA fingerprint cannot be extracted without a bootloader exploit or custom firmware.

---

## Processes Killed

All malicious processes were force-stopped:

- `com.amazon.brightdata.jellyfish` — BrightData residential proxy (was PID 7293)
- `com.free.vpn`
- `ch.protonvpn.android`
- `com.backysoft.security.vpn.router.freefire`
- `com.arcapps.freevpn.unblockproxy.hotspot.fastvpnsecure`
- `free.androidtv.vpn.proxy.purevpn`
- `com.vpn.freevpntv`
- `com.fastvpnword.firevpn`
- `com.securefast.firevpn`
- `com.palaxo.master.vpn.unlimited.proxy.server`
- `com.digiapp.vpn.tv.amazon`
- `com.azonejp.vpn`
- `org.hola.amazon` — Hola VPN (second proxy network)
- `com.esaba.downloader` — Downloader sideloading app
- `org.xbmc.kodi` — Kodi media player

Post-kill verification confirmed BrightData dead, all VPNs dead, Kodi dead, Downloader dead. No suspicious user processes remaining.

---

## Packages Disabled (15)

All 15 malicious packages were disabled via `pm disable-user --user 0`. Disabled packages **cannot restart on boot** and are invisible to the launcher. They remain installed (preserving evidence) but are inert.

```
package:com.amazon.brightdata.jellyfish          disabled-user
package:com.free.vpn                              disabled-user
package:ch.protonvpn.android                      disabled-user
package:com.backysoft.security.vpn.router.freefire disabled-user
package:com.arcapps.freevpn.unblockproxy.hotspot.fastvpnsecure disabled-user
package:free.androidtv.vpn.proxy.purevpn          disabled-user
package:com.vpn.freevpntv                         disabled-user
package:com.fastvpnword.firevpn                   disabled-user
package:com.securefast.firevpn                    disabled-user
package:com.palaxo.master.vpn.unlimited.proxy.server disabled-user
package:com.digiapp.vpn.tv.amazon                 disabled-user
package:com.azonejp.vpn                           disabled-user
package:org.hola.amazon                           disabled-user
package:com.esaba.downloader                      disabled-user
package:org.xbmc.kodi                             disabled-user
```

---

## App Data Cleared

| App | Action |
|-----|--------|
| com.amazon.brightdata.jellyfish | `pm clear` — **Success** (local data wiped) |
| org.hola.amazon | `pm clear` — **Success** (local data wiped) |

Data was cleared AFTER the encrypted log file and sdcard data were extracted for evidence.

---

## Evidence Extracted

| Artifact | Size | Location on M5 |
|----------|------|----------------|
| Screenshot | 580 KB | `/tmp/firestick-screenshot.png` |
| BrightData encrypted log | 2.1 MB | `/tmp/brightdata-jellyfish.log` |
| BrightData sdcard data | — | `/tmp/firestick-evidence/com.amazon.brightdata.jellyfish/` |
| Kodi sdcard data (577 files) | 6.2 MB | `/tmp/firestick-evidence/org.xbmc.kodi/` |
| Free VPN sdcard data (74 files) | 7.5 MB | `/tmp/firestick-evidence/com.arcapps.freevpn.unblockproxy.hotspot.fastvpnsecure/` |
| Hola sdcard data | — | `/tmp/firestick-evidence/org.hola.amazon/` |
| Downloader folder | empty | `/tmp/firestick-evidence/Downloader/` (APKs were deleted after sideloading) |
| **Total evidence pulled** | **~16 MB** | `/tmp/firestick-evidence/` |

---

## Remaining Issues (Require Root)

| Issue | Status | Why |
|-------|--------|-----|
| Port 1080 (SOCKS proxy) | **STILL LISTENING** | Held by system UID 1010 (wifi service) |
| Port 8888 (HTTP proxy) | **STILL LISTENING** | Held by system UID 1010 (wifi service) |
| Port 5555 (ADB network) | **STILL LISTENING** | Cannot restart adbd without root |
| ADB authorized keys | **CANNOT READ** | `/data/misc/adb/adb_keys` permission denied |
| Wi-Fi disable | **CANNOT FORCE** | `svc wifi disable` requires system permission |
| Airplane mode | **CANNOT FORCE** | Broadcast requires system permission |

The proxy ports (1080/8888) and ADB network port (5555) cannot be closed without root access. Fire OS production builds prevent `adb root`, `su`, airplane mode broadcast, and wifi service control from the shell user.

**Mitigation:** Q will unplug the Fire Stick from power before leaving. The 15 disabled packages persist across reboots — when the device is plugged back in, none of the malicious apps will start.

---

## Monitors Running on M5

| Monitor | PID | Frequency | Log File |
|---------|-----|-----------|----------|
| Apparatus Watchdog | 17451 | Every 60s | `/tmp/apparatus-watchdog.log` + `/tmp/apparatus-alerts.log` |
| DNS Hijack Ping | 3940 | Every 17s | `/tmp/dns-hijack-ping.log` |

The watchdog scans: Metro2 device census (new MACs), all 6 Styx LAN apparatus nodes, DNS hijack state, Fire Stick return detection, non-M5 SSH to Styx, and non-M5 auth on Dragon.

---

## M5 ADB Pairing

M5 is now permanently paired with the Fire Stick via RSA key exchange. When the Fire Stick is plugged back into M5 via USB, ADB will reconnect without requiring the screen authorization dialog again. The pairing survives device reboots and account changes.

---

*Lockdown complete. 15 malicious packages disabled. BrightData proxy killed and data cleared. All extractable evidence preserved on M5. Device registered to Q. Fire Stick to be unplugged before Q leaves.*
