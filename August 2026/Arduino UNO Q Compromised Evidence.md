# Arduino UNO Q (Sovereign Door) — Compromised Evidence

**Extracted:** 2026-08-10 ~22:13 PDT
**Device:** Arduino UNO Q (Snapdragon X1 + ATmega328, 4GB RAM)
**Hostname:** sovereign-door
**MAC:** 14:b5:cd:eb:0e:4d (built-in Wi-Fi)
**Connection method:** Styx configured to broadcast fake "metro2" SSID with old password — Sovereign Door auto-connected to Styx LAN at 192.168.10.160

---

## Users

| User | UID | Shell | Password | SSH Keys |
|------|-----|-------|----------|----------|
| root | 0 | /bin/bash | Locked (*) | DNS push key from Synastry (forced command) |
| arduino | 1000 | /bin/bash | **SET** (yescrypt hash) | `quinceylee@nftlasvegas.io` — **NOT FAFO** |
| aphroqite | 1001 | /bin/bash | Locked (!) | FAFO + Q-Emergency-Backup |

**The `arduino` user has a separate SSH key** (`quinceylee@nftlasvegas.io`) that is NOT the FAFO key. This key could allow someone with the matching private key to SSH into the Sovereign Door as the `arduino` user.

---

## Running Services

| Service | Description | Concern |
|---------|-------------|---------|
| **adbd.service** | Android Debug Bridge daemon | **ADB IS RUNNING** — same issue as Fire Stick |
| arduino-app-cli.service | Arduino App CLI daemon | Board management |
| arduino-router-serial.service | Proxy for Arduino Router Monitor | Serial bridge |
| arduino-router.service | Arduino Router Service | Board routing |
| unbound.service | Unbound DNS server | **Poisoned DNS zones** |
| docker.service | Docker container engine | No containers running |
| containerd.service | Container runtime | Docker dependency |
| ssh.service | OpenSSH server | Remote access |
| lightdm.service | Display manager | GUI (XFCE desktop) |
| rmtfs.service | Qualcomm remotefs service | Qualcomm modem |
| tqftpserv.service | QRTR TFTP service | Qualcomm modem |
| ModemManager.service | Modem Manager | **Cellular modem capability?** |
| NetworkManager.service | Network Manager | Wi-Fi management |
| bluetooth.service | Bluetooth service | Bluetooth active |
| fwupd.service | Firmware update daemon | Auto-update |

**ADB daemon is running on the Sovereign Door.** This is the same vulnerability found on the Fire Stick — ADB provides shell access to anyone who connects.

**ModemManager is running** — the Snapdragon X1 may have cellular modem capability. If so, the device could communicate over cellular data independently of Wi-Fi.

---

## Listening Ports

| Port | Address | Service |
|------|---------|---------|
| 22 | 0.0.0.0 | SSH |
| 53 | 127.0.0.1 | Unbound DNS (local) |
| 53 | 192.168.0.225 | Unbound DNS (bound to OLD Metro2 IP) |
| 7500 | 127.0.0.1 | Unknown |
| 8800 | 127.0.0.1 | Unknown |
| 43405 | 127.0.0.1 | Unknown |

---

## Saved Wi-Fi Network

```
SSID: metro2
Password: 09876543 (PLAINTEXT)
DNS: 127.0.0.1 (self — Unbound)
ignore-auto-dns: true
```

---

## SSH Keys

### aphroqite authorized_keys
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGfurBqscmRsDzpJjnBSB+Ur+aeyoJ24gj643+rS+8RZ Fuck-Around-Find-Out
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHEzD79uN1dvXYtXx+SRRdrd1gBTXWpmCXRM84Wq7E/I Q-Emergency-Backup
```

### root authorized_keys (forced command)
```
command="/usr/local/bin/apply-dns-config",no-agent-forwarding,no-port-forwarding,no-pty,no-user-rc,no-X11-forwarding ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFlGw4BZik+hjSPr2wSF0v7ZUhu/ZLxSNX04fz2JbnKz apparatus-dns-push@synastry
```

### arduino authorized_keys — UNAUTHORIZED KEY
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILWskEAylIS10n247Q209XL+h+dlAD2/duLK5JkmqVxq quinceylee@nftlasvegas.io
```

This key on the `arduino` user was NOT placed by Q. It provides SSH access via a key labeled with Q's email address but is a separate key from FAFO.

### SSH Host Keys
```
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHXz4EuzJ8jUcEQ+SZt9SERBpV5s0zempi0JZ6BomyC9CfVOXDtSf+oofsGwXFMQ+aqth9gZwLal5ZCiU3CmP9k=
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOZ1+zssD/TpACELy/+6iq/FJVuc9yYZuLacFBfP2s5S
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDOfiKC6U+7k/TtvjlpM0eoP9zeNycCIwk+NwC8OOQwqImRGekHVCmHMvOGzglblc9BglCaLt/LXWbMIXIjx62aNr5YiF8T3RTXmNMlwQenAOIlswJ7mHAbapVk8lQ614XQ05BmleClFBzIb9YDpOLCCkloq0b5KfQ9VILO18qHZ2Woepk3yh5qLqFjPUazJYkmi63L/7O0QHc2vHnmggaW+3ZV7aSzIU8XrWDcRmR8Q4KZIsAuBa4nm4nogAI6HWkXJKoJmI0JFoXGpu7tzcopbmRdoMjxHEr8pAnoZw6w/IoIEra173zLA+7UGg7ewx3ecWckHIH99OTxmR4CWriSgjUv7lcqdsLsNcemV4pZUU+iTWMoY2O5zE6/sNf5yweDB01vX/ZETyhNU/pSrukIJyM8Qak3P3t+iyjDZwZQX8da/ar7kKLHe2lBjWdHnZ81/mwBBSQOHiOI/tV+wI9GwDknCHWuoUHNH9qZbWkiUA0C1ggqr9MSiooW7SaMr2k=
```

---

## Login History

| Date | Event |
|------|-------|
| 2026-07-06 04:20–08:45 | 8 crash/reboot cycles (configuration/testing) |
| 2026-07-12 10:45 | Stable boot — ran continuously until Aug 10 |
| 2026-08-10 ~22:34 PDT | Rebooted (Q unplugged/replugged USB-C) |
| 2026-08-10 ~22:34 PDT | Auto-powered off while connected to M5 USB |

**The device spontaneously powered off while connected to M5 via USB-C.** Q did not unplug it. The device shut itself down and had to be power-cycled.

---

## Unbound DNS Configuration

Same poisoned zone configuration as documented in `DNS Hijacking 8-9-2026` evidence. All four domains (quincey.ai, ares.technology, ares.love, aphroqite.ai) resolve to 192.168.0.225 (self).

---

## Hardware Details (from Unbound config comment)

```
Hardware: Arduino UNO Q (Snapdragon X1 + ATmega328, 4GB RAM)
```

- **Snapdragon X1:** Qualcomm mobile SoC — explains rmtfs, tqftpserv, ModemManager services. May have cellular modem capability.
- **ATmega328:** Arduino microcontroller co-processor — handles the Arduino sketch/serial interface.
- **4GB RAM**
- **Built-in Wi-Fi** (14:b5:cd:eb:0e:4d, Liteon chipset)
- **USB-C** (serial console to ATmega328, power)
- **Qwiic port** (for Modulino accessories)

---

## Docker

Docker and containerd are running but **no containers or images** are present. Docker may have been used previously and cleaned, or was installed for future use.

---

## Desktop Environment

XFCE 4.20 is installed with lightdm display manager. This is a full desktop environment on a headless device with no HDMI port — suggesting it was configured for remote desktop access (VNC/RDP) or was pre-installed by the OS image.

---

## Evidence Summary

| Finding | Severity |
|---------|----------|
| ADB daemon running | **HIGH** — remote shell access |
| Unauthorized SSH key on `arduino` user | **HIGH** — backdoor access |
| ModemManager active (cellular modem?) | **HIGH** — independent communication channel |
| Poisoned DNS zones for Q's domains | **HIGH** — domain hijacking |
| Metro2 password in plaintext | **MEDIUM** — credential exposure |
| Spontaneous power-off during investigation | **SUSPICIOUS** — possible anti-forensic behavior |
| Bluetooth service active | **MEDIUM** — additional attack surface |
| Docker installed with no containers | **LOW** — prepared but unused |

---

*This document records all data extracted from the Arduino UNO Q (Sovereign Door) on August 10, 2026, immediately prior to factory reset. The device was accessed by configuring the Styx router to broadcast a fake "metro2" SSID with the device's saved password, causing it to auto-connect to the Styx LAN.*
