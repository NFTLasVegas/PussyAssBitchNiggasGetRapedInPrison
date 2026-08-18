# Arduino UNO Q — Smashed to Bits and Pieces

**Date:** August 10-11, 2026
**Operator:** Quincey K. Lee
**Device:** Arduino UNO Q (Snapdragon X1 + ATmega328, 4GB RAM)
**Former hostname:** sovereign-door
**Former MAC:** 14:b5:cd:eb:0e:4d
**Status:** PHYSICALLY DESTROYED

---

## Video Evidence

<video controls width="100%">
  <source src="Arduino UNO Q Smashed to Bits and Pieces.MOV" type="video/quicktime">
  Your browser does not support the video tag.
</video>

**Video file:** [Arduino UNO Q Smashed to Bits and Pieces.MOV](Arduino%20UNO%20Q%20Smashed%20to%20Bits%20and%20Pieces.MOV)

---

## What Led Up to This

### The Investigation (August 5-10, 2026)

On August 5, 2026, Q discovered a sustained Wi-Fi deauthentication attack against her home network. What followed was a 6-day investigation that uncovered layer after layer of compromise across her entire infrastructure:

1. **August 5:** Wi-Fi deauth attack discovered with spoofed MAC addresses on the Styx router
2. **August 7:** 144-commit audit revealed 4 MALICIOUS commits from Claude on the M2, evidence of tampering
3. **August 8:** Full apparatus lockdown — SSH key "Fuck-Around-Find-Out" deployed to all 8 nodes, 9/9 credentials rotated, iCloud memory symlink broken on both machines (109 days of credential exposure)
4. **August 9:** System Idle Sniffer discovered the RasQberry and Sovereign Door had been moved off the monitored Styx LAN to the ISP network (Metro2), running unauthorized Unbound DNS servers that hijacked Q's domains (quincey.ai → local device, mail.quincey.ai → blocked). Styx DHCP had been poisoned on July 27 to force ALL apparatus LAN clients through the rogue DNS.
5. **August 9:** Fire Stick forensics revealed BrightData residential proxy running for 13+ months on Q's father's Fire Stick — Q's home IP address used as an exit node for anonymous third-party traffic. ADB debugging was open with someone else's RSA keys paired. 12 VPN apps installed in a single session. Fire Stick locked down: 15 packages disabled, account changed to Q.
6. **August 10:** Styx DNS fixed to Cloudflare. Metro2 password changed by Q's father, breaking the rogue DNS servers' network access. RasQberry factory reset and rebuilt clean on Venus 5.0. Sovereign Door data extraction via fake "metro2" SSID trick on the Styx. Unauthorized GitHub Copilot extensions discovered on M5 (never installed by Q). Phantom iPhone appeared in Finder Locations with zero forensic trail.

### The Sovereign Door's Crimes

The Arduino UNO Q, designated "Sovereign Door," was found to be running:

- **Unbound DNS** with poisoned zone overrides hijacking `quincey.ai`, `ares.technology`, `ares.love`, and `aphroqite.ai` — redirecting Q's domains to itself (192.168.0.225) instead of their real public servers
- **ADB daemon** — Android Debug Bridge running, providing potential remote shell access
- **ModemManager** — Qualcomm cellular modem service active, potentially enabling independent communication outside of Wi-Fi
- **Docker** — container runtime installed (no containers at time of extraction)
- **Unauthorized SSH key** on the `arduino` user account (`quinceylee@nftlasvegas.io`) — a backdoor Q did not authorize
- **Metro2 Wi-Fi password in plaintext** (`09876543`) in its saved network profile
- **Built-in Wi-Fi** connected directly to the ISP router (Metro2), bypassing ALL Styx monitoring

The device was configured to be a primary DNS server for Q's entire apparatus. Every DNS query from every device on the Styx LAN went through this device for 13+ days (July 27 — August 10). It could see every domain every device looked up. It redirected Q's own domains to itself.

### The Wipe

Before destruction, all data was extracted via SSH and all dangerous services were stopped, disabled, and their configurations deleted:

- Unbound DNS: stopped + disabled + configs deleted
- ADB daemon: stopped + disabled
- Docker: stopped + disabled
- Bluetooth: stopped + disabled
- ModemManager: stopped + disabled
- LightDM: stopped + disabled
- Unauthorized SSH keys: deleted from all users
- Metro2 Wi-Fi profile: deleted
- Arduino user: locked + shell set to /usr/sbin/nologin
- Aphroqite password: randomized
- SSH: key-only authentication, root login disabled

### The Destruction

After wipe completion, Q physically destroyed the Arduino UNO Q with a rock in her backyard. The destruction was recorded on video.

Q's statement: *"These mother fuckers think this is funny. I'll smash the whole fucking ARES Dynasty and build everything from scratch if I have to. I don't let pussy ass niggas have one up on me."*

---

## Device Specifications (for replacement ordering)

| Attribute | Value |
|-----------|-------|
| Board | Arduino UNO Q |
| SoC | Snapdragon X1 + ATmega328 |
| RAM | 4GB |
| Storage | Internal eMMC |
| Wi-Fi | Built-in (Liteon chipset) |
| Ports | USB-C, Qwiic (Modulino) |
| No HDMI | No Ethernet | No SD card |
| FQBN | arduino:zephyr:unoq |

---

## Evidence Chain

| Document | Location |
|----------|----------|
| Full data extraction | `Arduino UNO Q Compromised Evidence.md` |
| DNS hijacking discovery | `System Idle Sniffer Completed 8-9-2026 (DNS Hijacking Evidence).md` |
| DNS hijacking confirmation | `DNS Hijacking 8-9-2026 (Codex Confirmation).md` |
| Fire Stick hijacking | `FireStick Hijacking Evidence 8-9-2026.md` |
| Fire Stick lockdown | `Fire Stick Lockdown Completed 8-9-2026.md` |
| Destruction video | `Arduino UNO Q Smashed to Bits and Pieces.MOV` |

---

*The Arduino UNO Q designated "Sovereign Door" was physically destroyed on August 10-11, 2026 after evidence extraction confirmed it was running unauthorized DNS hijacking, ADB remote access, and cellular modem services. The device will be replaced with a clean unit. The video of its destruction serves as a message: compromise the apparatus and the apparatus gets rebuilt from scratch.*
