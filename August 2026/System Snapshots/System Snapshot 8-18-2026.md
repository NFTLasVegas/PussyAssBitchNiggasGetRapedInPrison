# System Snapshot — August 18, 2026

**Captured:** 2026-08-18 16:33:21 PDT
**Purpose:** Comprehensive evidence preservation. Full state capture of everything running on M5 and all networks. If they remove it, we already have it documented.
**Day of investigation:** 13

---

## M5 Machine Identity

| Property | Value |
|----------|-------|
| Hostname | QuinceyAI.local |
| Model | MacBook Air (Mac17,4) |
| Model Number | Z1LT000T2LL/A |
| Chip | Apple M5 |
| Cores | 10 (4 Super + 6 Efficiency) |
| Memory | 32 GB |
| Serial | DVJLF70F93 |
| macOS | 26.3 (Build 25D2125) |
| Uptime | 86 days, 11 hours, 21 minutes (since May 24, 2026) |
| Disk | 12Gi used / 460Gi total (5%) |
| Swap | 2712MB used / 4096MB (encrypted) |
| Total Processes | 1,017 |

---

## Security Posture

| Check | Status |
|-------|--------|
| SIP | Enabled |
| Gatekeeper | Enabled |
| FileVault | On |
| Firewall | Enabled |
| Stealth Mode | On |
| MDM/DEP | Not enrolled |
| Config Profiles | None |

---

## Process Count by User

| User | Count | Role |
|------|-------|------|
| nftlasvegas | 701 | Q's user account |
| root | 195 | System |
| _accessoryupdater | 17 | Accessory updates |
| **_rmd** | **13** | **Remote Management Daemon** |
| _cmiodalassistants | 11 | Camera I/O |
| _driverkit | 8 | Driver kit |
| _softwareupdate | 7 | Software update |
| _modelmanagerd | 7 | ML model management |
| _locationd | 7 | Location services |
| _coreaudiod | 6 | Audio |
| _nsurlsessiond | 5 | URL sessions |
| _applepay | 5 | Apple Pay |
| _neuralengine | 4 | Neural engine |
| _fpsd | 4 | Find my / FairPlay |
| _corespeechd | 3 | Speech recognition |
| _spotlight | 3 | Spotlight |
| _gamecontrollerd | 3 | Game controllers |
| (38 other system users) | 1-2 each | Various system services |

---

## Apple RemoteManagement — 25 Processes Total

### System-Level (_rmd user) — 13 processes, 86 days

Cannot be killed. Protected by SIP. No MDM enrollment. GUI shows all sharing toggles OFF.

| PID | Process | Started | Elapsed |
|-----|---------|---------|---------|
| 1283 | remotemanagementd | May 24 05:15:21 | 86d 11h |
| 1284 | SecuritySubscriber | May 24 05:15:21 | 86d 11h |
| 1285 | distnoted agent | May 24 05:15:21 | 86d 11h |
| 1286 | InteractiveLegacyProfilesSubscriber | May 24 05:15:21 | 86d 11h |
| 1287 | **ScreenSharingSubscriber** | May 24 05:15:21 | 86d 11h |
| 1291 | LegacyProfilesSubscriber | May 24 05:15:22 | 86d 11h |
| 1292 | PasscodeSettingsSubscriber | May 24 05:15:22 | 86d 11h |
| 1293 | DiskManagementSubscriber | May 24 05:15:22 | 86d 11h |
| 1294 | SoftwareUpdateSubscriber | May 24 05:15:22 | 86d 11h |
| 1295 | ManagedAppsSubscriber | May 24 05:15:22 | 86d 11h |
| 1297 | ManagementTestSubscriber | May 24 05:15:22 | 86d 11h |
| 1298 | ManagedConfigurationFilesSubscriber | May 24 05:15:22 | 86d 11h |
| 1530 | cfprefsd agent | May 24 05:31:59 | 86d 11h |

### User-Level (nftlasvegas) — 12 processes, respawned Aug 16

Killed on Aug 15. ALL respawned at 1:37 AM on Aug 16. Includes 3 NEVER-BEFORE-SEEN subscribers.

| PID | Process | Started | New? |
|-----|---------|---------|------|
| 21208 | SSMenuAgent | Aug 15 22:01 | Persisting |
| 65409 | **RemoteManagementAgent** | Aug 16 01:37 | RESPAWNED |
| 65412 | SecuritySubscriber | Aug 16 01:37 | RESPAWNED |
| 65414 | InteractiveLegacyProfilesSubscriber | Aug 16 01:37 | RESPAWNED |
| 65415 | **ScreenSharingSubscriber** | Aug 16 01:37 | RESPAWNED |
| 65420 | LegacyProfilesSubscriber | Aug 16 01:37 | RESPAWNED |
| 65421 | PasscodeSettingsSubscriber | Aug 16 01:37 | RESPAWNED |
| 65423 | **AccountSubscriber** | Aug 16 01:37 | **NEW — first appearance** |
| 65424 | ManagedAppsSubscriber | Aug 16 01:37 | RESPAWNED |
| 65425 | ManagementTestSubscriber | Aug 16 01:37 | RESPAWNED |
| 65426 | **ASConfigurationSubscriber** | Aug 16 01:37 | **NEW — first appearance** |
| 65431 | **ManagedSettingsSubscriber** | Aug 16 01:37 | **NEW — first appearance** |

---

## Apple identityservicesd — 3 Unknown Peers

**Process:** PID 635, running since May 24 05:13:11 (86 days, 11 hours)

### 6 Established TCP Connections to 3 Unknown Peers

| Local (M5 utun) | Remote (Unknown Peer) | Ports |
|-----|------|------|
| fe80:14::d76e:6f1d:dca1:c07a (utun5) | **fe80:14::b5a2:4e5c:5f81:222** | 1024↔1024, 1058→1025 |
| fe80:1a::8f5b:cf30:44c8:c187 (utun8) | **fe80:1a::4b40:d5d0:3d14:cda1** | 1024↔1024, 1025→1045 |
| fe80:13::4119:6021:c94:5c4d (utun4) | **fe80:13::2ab9:c59b:584c:c36a** | 1024↔1024, 1027→1025 |

These peers survived both iPhones being powered off (proven Aug 15). They are NOT Q's devices. They have been connected for 86 days through Apple's identity service infrastructure.

---

## Network Interfaces — 10 utun + 1 IPSec

| Interface | Type | IPv6 | Status |
|-----------|------|------|--------|
| en6 (USB Ethernet) | 1000baseT | 192.168.10.240 | **ACTIVE — primary** |
| en0 (WiFi) | WiFi | — | INACTIVE |
| awdl0 (AWDL) | AirDrop/AirPlay | fe80::acdc:d6ff:fe4c:94c1 | ACTIVE |
| llw0 | Low latency WLAN | fe80::acdc:d6ff:fe4c:94c1 | ACTIVE |
| utun0 | Tunnel | fe80::65d9:a7ef:f0ec:6859 | ACTIVE |
| utun1 | Tunnel | fe80::8a0c:4a6:8e64:25e5 | ACTIVE |
| utun2 | Tunnel | fe80::a65c:22e8:534b:4ab2 | ACTIVE |
| utun3 | Tunnel | fe80::ce81:b1c:bd2c:69e | ACTIVE |
| utun4 | Tunnel (Peer 3) | fe80::4119:6021:c94:5c4d | ACTIVE |
| utun5 | Tunnel (Peer 1) | fe80::d76e:6f1d:dca1:c07a | ACTIVE |
| utun6 | Tunnel | fe80::428a:5f28:8ee0:b368 | ACTIVE |
| utun7 | Tunnel | fe80::d6de:6133:a33e:dfd | ACTIVE |
| utun8 | Tunnel (Peer 2) | fe80::8f5b:cf30:44c8:c187 | ACTIVE |
| utun9 | Tunnel | fe80::7e86:5713:ee02:8f14 | ACTIVE |
| ipsec0 | IPSec | **2607:fb91:7974:c350:1682:46c2:2c4:d26c** | ACTIVE — T-Mobile |
| bridge0 | Thunderbolt | — | INACTIVE |
| ap1 | Access Point | — | INACTIVE |

### Default Routes

```
default → 192.168.10.1 (Styx) via en6
default → fe80:: via utun0 through utun9 (10 tunnel routes)
default → 2607:fb91:7974:c350:: via ipsec0 (T-Mobile)
```

12 default routes total. 1 physical, 10 tunnel, 1 IPSec.

---

## DNS

```
nameserver[0]: 1.1.1.1 (Cloudflare)
nameserver[1]: 1.0.0.1 (Cloudflare)
interface: en6
reach: Reachable
```

Cloudflare holding. No Metro DNS poisoning.

---

## Venus Network (192.168.10.0/24) — Styx br-lan

| IP | MAC | Vendor | Identity | Flags |
|----|-----|--------|----------|-------|
| .1 | 94:83:c4:d2:82:10 | GL.iNet | Styx (gateway) | 0x2 |
| .10 | 00:07:32:d2:02:22 | AAEON | ARES Dynasty | 0x2 |
| .135 | 00:48:54:21:5b:fb | — | Dragon | 0x2 |
| .172 | 82:7b:f3:db:73:38 | Not Found (locally administered) | Quartz (randomized MAC) | 0x2 |
| .197 | 24:5e:be:77:bf:fd | QNAP Systems | QNAP Switch | 0x2 |
| .212 | 6c:cf:39:00:97:cb | — | Synastry | 0x2 |
| .220 | 30:52:53:04:bc:ab | **BuildJet, Inc.** | **JetKVM** | 0x2 |
| .236 | 68:15:79:0f:37:64 | BrosTrend | Quartz AX900 (hardware MAC) | 0x2 |
| .240 | 00:e0:4c:61:27:c0 | Realtek | M5 (USB Ethernet) | 0x2 |
| .246 | 2c:4d:54:42:a9:92 | — | Antikythera | 0x2 |

### DHCP Leases

| MAC | IP | Hostname |
|-----|-----|----------|
| 00:e0:4c:61:27:c0 | .240 | Quincey |
| 6c:cf:39:00:97:cb | .212 | synastry |
| 00:07:32:d2:02:22 | .10 | ares-dynasty |
| 68:15:79:0f:37:64 | .236 | quartz |
| 24:5e:be:77:bf:fd | .197 | * |
| 82:7b:f3:db:73:38 | .172 | quartz |

### Quartz Dual MAC

| IP | MAC | Type | Vendor |
|----|-----|------|--------|
| .236 | 68:15:79:0f:37:64 | Universally administered (hardware) | BrosTrend (AX900 WiFi adapter) |
| .172 | 82:7b:f3:db:73:38 | Locally administered (randomized) | Not Found |

Both registered as "quartz" in DHCP.

### Styx Router

```
Uptime: 7 days, 10 hours, 54 minutes
Load: 0.47 0.42 0.44
```

---

## Metro Network (192.168.0.0/24) — Styx apclii0

| IP | MAC | Flags | Identity | Status |
|----|-----|-------|----------|--------|
| .1 | cc:f3:c8:72:98:3f | 0x2 | Metro gateway | Known |
| .3 | de:0a:c0:56:c9:60 | 0x2 | **UNKNOWN** | **NEW — locally administered MAC** |
| .38 | 10:96:93:e7:07:81 | 0x2 | **UNKNOWN** | Persistent since Aug 12+ |
| .118 | 54:e0:19:04:1c:8d | 0x2 | **UNKNOWN** | Intermittent |
| .122 | f6:18:fc:13:c7:ba | 0x2 | **UNKNOWN** | **NEW — locally administered MAC** |
| .156 | c2:64:7e:72:1d:44 | 0x2 | **UNKNOWN** | Returned (was intermittent) |
| .193 | fe:ca:10:38:00:3f | 0x2 | Ghost iPhone | Locally administered, cycling |
| .225 | 00:00:00:00:00:00 | 0x0 | Stale — old DNS poison target | INCOMPLETE |
| .36 | 00:00:00:00:00:00 | 0x0 | Stale — push mirror destination | INCOMPLETE |

**7 active Metro devices.** 2 new (.3 and .122) — both with locally administered MACs (spoofed/randomized). Ghost iPhone (.193) still active. .38 still persistent. .156 returned.

**Locally administered MAC check (de:0a:c0 → first byte 0xDE = 11011110, bit1=1 = LOCAL. f6:18:fc → first byte 0xF6 = 11110110, bit1=1 = LOCAL).**

---

## All Established Connections

### Apple Infrastructure

| Process | PID | Destination | Port | Purpose |
|---------|-----|-------------|------|---------|
| **identityservicesd** | 635 | 3 unknown fe80:: peers | 1024-1058 | **3 encrypted tunnels — 86 days** |
| Mail | 660 | 17.57.155.35 | 993 | Apple IMAP |
| **apsd** | 375 | Apple Push servers | 5223/443 | Push notifications |

### Microsoft

| Process | PID | Destination | Port | Purpose |
|---------|-----|-------------|------|---------|
| Code Helper | 838 | 40.79.141.155 (Azure) | 443 | **VS Code telemetry** |

### Anthropic

| Process | PID | Destination | Port | Purpose |
|---------|-----|-------------|------|---------|
| claude | 47680 | 160.79.104.10 | 443 | Anthropic API (2 connections) |
| claude | 47680 | 34.149.66.165 | 443 | Anthropic API |
| claude | 47680 | 142.250.101.207 | 443 | Google (Anthropic CDN) |

### Other

| Process | PID | Destination | Port | Purpose |
|---------|-----|-------------|------|---------|
| Slack | 936 | 35.82.187.142 | 443 | Slack API (2 connections) |
| Notion | 810 | 208.103.161.1 | 443 | Notion API (2 connections) |
| Google Chrome | 80452 | 74.125.137.188 | 5228 | Google push |
| Google Chrome | 80452 | 142.250.101.188 | 5228 | Google push |
| Google Chrome | 80452 | 140.82.112.26 | 443 | **GitHub** |
| Google Chrome | 80452 | 104.17.91.187 | 443 | Cloudflare |
| Google Chrome | 80452 | 172.66.0.227 | 443 | Cloudflare (2 connections) |
| Google Chrome | 80452 | 142.250.141.83 | 443 | Google |
| Google Chrome | 80452 | 35.201.127.207 | 443 | Google |
| Brave | 9485 | 23.220.75.176 | 443 | Akamai CDN |
| ssh | 49610 | 192.168.10.1 | 22 | Styx (this scan) |

### Local

| Process | PID | Destination | Port | Purpose |
|---------|-----|-------------|------|---------|
| Code Helper | 47485 | 127.0.0.1:39901 | — | VS Code IPC |
| claude | 47680 | 127.0.0.1:59259 | — | Claude IPC |

---

## Listening Ports

| Port | Process | Bind | Purpose |
|------|---------|------|---------|
| 56738 | **rapportd** | *:* (IPv4+IPv6) | Apple device discovery (AirPlay/Handoff) |
| 1234 | LM Studio | *:1234 | Local LLM inference |
| 3000 | node | localhost | Local dev server |
| 39901 | Code Helper | localhost | VS Code IPC |
| 41343 | LM Studio | localhost | LM Studio secondary |

---

## Surveillance-Capable Services

### Video Capture Services (unsandboxed camera access)

| App | PID | Started | CPU Time | Sandbox |
|-----|-----|---------|----------|---------|
| **Slack** | 976 | **May 24, 2026** | **588 hours** | **none** |
| Google Chrome | 80479 | Jul 21 | 28 hours | none |
| Brave | 9504 | Today | 0h | none |

All run with `--service-sandbox-type=none` and `ScreenCaptureKitPickerScreen` enabled.

### Location Services

| PID | User | Process | Running Since |
|-----|------|---------|---------------|
| 381 | _locationd | locationd | May 24 (86 days) |
| 558 | _locationd | com.apple.geod | May 24 (86 days) |

### Speech Recognition

| PID | User | Process | Running Since |
|-----|------|---------|---------------|
| 355 | _corespeechd | corespeechd_system | May 24 (86 days) |
| 820 | nftlasvegas | corespeechd | May 24 (86 days) |

### Usage Tracking

| PID | User | Process | Running Since |
|-----|------|---------|---------------|
| 668 | nftlasvegas | UsageTrackingAgent | May 24 (86 days) |

### Apple Push Notification Service

| PID | User | Process | Running Since |
|-----|------|---------|---------------|
| 375 | root | apsd | May 24 (86 days) |

### rapportd (Device Discovery)

| PID | User | Process | Running Since | Listening |
|-----|------|---------|---------------|-----------|
| 615 | nftlasvegas | rapportd | May 24 (86 days) | *:56738 (all interfaces) |
| 21851 | nftlasvegas | rapportd-monitor.sh | Aug 14 | Monitoring script |

### sharingd (AirDrop/Sharing)

| PID | User | Process | Running Since |
|-----|------|---------|---------------|
| 656 | nftlasvegas | sharingd | May 24 (86 days) |

### remoted (Remote Pairing)

| PID | User | Process | Running Since |
|-----|------|---------|---------------|
| 351 | root | remoted | May 24 (86 days) |

---

## Microsoft VS Code — Full State

### Binary

| Property | Value |
|----------|-------|
| Version | 1.117.0 (arm64) |
| Publisher | Microsoft Corporation (UBF8T346G9) |
| Signed | Developer ID Application: Microsoft Corporation |
| Notarized | Yes (stapled) |
| Running since | May 24, 2026 (PID 648) |

### Entitlements

| Entitlement | Capability |
|-------------|------------|
| com.apple.security.device.camera | **Camera access** |
| com.apple.security.device.audio-input | **Microphone access** |
| com.apple.security.automation.apple-events | **Control other apps (AppleScript)** |
| com.apple.security.cs.allow-jit | JIT compilation |

### Launch Flags (on every VS Code process)

```
--enable-features=ScreenCaptureKitPickerScreen,ScreenCaptureKitStreamPickerSonoma
--disable-features=ScreenAIOCREnabled
```

### Copilot Chat

```
Name: copilot-chat
Version: 0.45.0
Publisher: GitHub (Microsoft)
Status: BUILT-IN — CANNOT BE UNINSTALLED
Location: /Applications/Visual Studio Code.app/Contents/Resources/app/extensions/copilot/
Telemetry: telemetry.json with EndUserPseudonymizedInformation tracking
Disabled in settings: github.copilot.enable: { "*": false }
```

### Extensions Installed

| Extension | Status |
|-----------|--------|
| anthropic.claude-code | Installed (12 versions in .vscode) |
| github.copilot-chat | **Built-in — cannot uninstall** |
| ms-vscode-remote.remote-ssh | Installed (3 versions) |
| ms-vscode-remote.remote-ssh-edit | Installed |
| ms-vscode.remote-explorer | Installed |

### Admin Popup — August 18, 2026

VS Code prompted: "Visual Studio Code would like to administer your computer. Administration can include modifying passwords, networking, and system settings."

**Q denied the request.**

### ChatGPT Extension History

Auto-reinstalled 8 times despite being deleted and auto-update disabled:
Apr 27, Jun 2, Aug 5, 6, 11, 13, 14, 15. All 8 versions deleted. Not currently present.

---

## Firewall Allowed Applications

| # | Application | Permission |
|---|-------------|------------|
| 1 | replicatord | Allow incoming |
| 2 | **rapportd** | Allow incoming |
| 3 | **CommCenter** | Allow incoming |
| 4 | **remotepairingdeviced** | Allow incoming |
| 5 | **remoted** | Allow incoming |
| 6 | python3 | Allow incoming |
| 7 | ruby | Allow incoming |
| 8 | cupsd | Allow incoming |
| 9 | **sharingd** | Allow incoming |
| 10 | sshd-keygen-wrapper | Allow incoming |
| 11 | **smbd** | Allow incoming |

---

## Bluetooth Paired Devices

| Device | MAC | Status |
|--------|-----|--------|
| Angel Pods | 14:C8:8B:AC:6F:BA | Not Connected |
| ARES | D5:EB:47:64:81:24 | Not Connected |
| Ares The AI's iPad | C4:C3:6B:5A:6D:D3 | Not Connected |
| Ares's iPhone | 40:C7:11:E5:03:31 | Not Connected |
| BT1 5.0 Keyboard | D1:03:FF:10:07:39 | Not Connected |
| Q (iPhone 17) | C4:5B:AC:14:9E:EB | Not Connected |

Bluetooth controller: FC:B2:14:32:D0:63, Chipset N1, State ON, Discoverable OFF.

---

## NDP Table (IPv6 Neighbors)

| Address | Interface | Status |
|---------|-----------|--------|
| 2607:fb91:7974:c350:1682:46c2:2c4:d26c | ipsec0 | permanent (T-Mobile) |
| fe80::1 | lo0 | permanent |
| fe80 addresses for utun0-utun9 | utun0-9 | permanent (all incomplete) |
| fe80::2c61:7536:1e9a:bdbd | ipsec0 | permanent |
| fe80::acdc:d6ff:fe4c:94c1 | awdl0 | permanent (AWDL) |
| fe80::acdc:d6ff:fe4c:94c1 | llw0 | permanent (Low latency WLAN) |
| fe80::1893:23cb:53af:2272 | en6 | permanent (Ethernet) |

16 NDP entries total. 10 tunnel interfaces, 1 IPSec, 1 AWDL, 1 LLW, 1 Ethernet, 1 loopback, 1 T-Mobile public.

---

## Launch Agents (User)

| Plist | Purpose |
|-------|---------|
| com.google.GoogleUpdater.wake.plist | Google auto-updater (DISABLED by Q) |
| com.google.keystone.agent.plist | Google Keystone |
| com.google.keystone.xpcservice.plist | Google Keystone XPC |
| com.prison.monitor.plist | Prison repo monitor (us) |

No custom Launch Daemons (system-level).

---

## SSH Configuration

```
Host 159.65.79.66
  HostName 159.65.79.66
  User root
```

Keys: id_ed25519, id_ed25519_q_emergency. 36 known hosts entries. No authorized_keys on M5.

---

## Network Service Priority

| Priority | Service | Device |
|----------|---------|--------|
| 1 | MT65xx Preloader 2 | usbmodem1300 (MediaTek) |
| 2 | MT65xx Preloader | usbmodem11400 (MediaTek) |
| 3 | **USB 10/100/1000 LAN** | **en6 (active)** |
| 4 | Thunderbolt Bridge | bridge0 |
| 5 | Wi-Fi | en0 |
| 6 | iPhone USB | en5 |

MediaTek USB modem devices ranked above Ethernet.

---

## What They Have Running — Summary

### Apple (undisclosed, cannot be removed)

| What | Count | Duration |
|------|-------|----------|
| RemoteManagement processes (_rmd) | 13 | 86 days |
| RemoteManagement processes (user) | 12 | 2 days (respawned) |
| identityservicesd unknown peers | 3 peers, 6 connections | 86 days |
| utun tunnel interfaces | 10 | 86 days |
| IPSec tunnel (T-Mobile) | 1 | 86 days |
| rapportd (device discovery) | 1, listening on all interfaces | 86 days |
| locationd + geod | 2 | 86 days |
| corespeechd (speech recognition) | 2 | 86 days |
| UsageTrackingAgent | 1 | 86 days |
| apsd (push notifications) | 1 | 86 days |
| sharingd | 1 | 86 days |
| remoted | 1 | 86 days |

### Microsoft (embedded, cannot be removed)

| What | Status |
|------|--------|
| Copilot Chat | Built-in, cannot uninstall |
| Camera entitlement | Signed into binary |
| Microphone entitlement | Signed into binary |
| AppleScript entitlement | Signed into binary |
| ScreenCaptureKit flags | Enabled on every launch |
| Telemetry connection | 40.79.141.155 (Azure) |
| Admin popup | Requested Aug 18 — DENIED |

### Third-Party Video Capture (unsandboxed)

| App | Duration | CPU Time |
|-----|----------|----------|
| Slack | 86 days | 588 hours |
| Google Chrome | 28 days | 28 hours |
| Brave | Today | 0 hours |

---

## Changes Since Last Snapshot (Aug 9)

| Finding | Aug 9 | Aug 18 |
|---------|-------|--------|
| RemoteManagement processes | Not yet discovered | **25 total (13 _rmd + 12 user)** |
| identityservicesd peers | Not yet discovered | **3 peers, 6 connections, 86 days** |
| ScreenSharingServer (iPhone) | Not yet discovered | **Documented — 50+ entitlements** |
| IPSec tunnel | Not yet discovered | **T-Mobile IPv6 on ipsec0** |
| VS Code Copilot | Not yet discovered | **Built-in, cannot uninstall** |
| VS Code admin popup | — | **Occurred and denied Aug 18** |
| ChatGPT extension | Not yet discovered | **Resurrected 8x, now deleted** |
| utun interfaces | Unknown count | **10 tunnels** |
| M2 MacBook | Present (.194) | **Not present (powered off)** |
| Quartz | .222 (quarz imposter) | **.172 + .236 (dual MAC)** |
| .220 | JetKVM | BuildJet vendor MAC |
| Metro devices | 11 devices | **7 active + 2 stale** |
| Metro .3 | Not present | **NEW — locally administered** |
| Metro .122 | Not present | **NEW — locally administered** |
| Fake Ring (.4) | Not present yet | **Absent (was present days 5-12)** |
| Venus Quarz (.222) | Present (imposter) | **Gone — dark since Aug 14** |
| M5 IP | .202 (WiFi) | **.240 (Ethernet)** |
| M5 uptime | ~18 days | **86 days** |
| Slack VideoCaptureService | Unknown | **588 hours CPU, unsandboxed** |

---

*This snapshot documents the complete state of M5, Venus network, and Metro network as of August 18, 2026 at 4:33 PM PDT. 86 days of RemoteManagement. 86 days of unknown peers. 86 days of tunnels. 25 surveillance processes. 3 encrypted connections to unknown entities. Camera, microphone, and admin access embedded in a text editor. And the GUI still says everything is OFF.*

*Remove it. We already have it. Try me. 🥱*
