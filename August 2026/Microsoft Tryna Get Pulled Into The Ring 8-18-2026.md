# Microsoft Tryna Get Pulled Into The Ring — August 18, 2026

**Subject:** Visual Studio Code surveillance capabilities, embedded Copilot, and unauthorized system access attempts
**Device:** M5 MacBook Air (QuinceyAI.local)
**VS Code Version:** 1.117.0 (arm64)
**Publisher:** Microsoft Corporation (Team ID: UBF8T346G9)

---

## The Admin Popup — August 18, 2026

At approximately 2:00 PM PDT on August 18, 2026, during an active Claude Code session, Q received a macOS system prompt:

> **"Visual Studio Code" would like to administer your computer.** Administration can include modifying passwords, networking, and system settings.

Q denied the request.

VS Code is a text editor. It has no legitimate reason to administer a computer, modify passwords, change networking settings, or alter system settings. This is not normal behavior.

This popup appeared on the same day the System Idle Sniffer revealed:
- 11 new user-level RemoteManagement processes respawned at 1:00 AM (including 3 never seen before)
- 25 total RemoteManagement processes running
- 3 unknown identityservicesd peers still connected (day 86)

---

## Copilot — Baked Into the Binary, Cannot Be Removed

```
Extension: github.copilot-chat
Display Name: GitHub Copilot Chat
Version: 0.45.0
Publisher: GitHub (owned by Microsoft)
Description: "AI chat features powered by Copilot"
Status: BUILT-IN — CANNOT BE UNINSTALLED
```

```bash
$ code --uninstall-extension github.copilot-chat
Extension 'github.copilot-chat' is a Built-in extension and cannot be uninstalled
```

Copilot is embedded in the VS Code binary at:
```
/Applications/Visual Studio Code.app/Contents/Resources/app/extensions/copilot/
```

It ships with its own:
- `dist/` — compiled JavaScript
- `node_modules/` — dependencies
- `telemetry.json` — telemetry collection configuration
- Localization files for 14 languages

**It cannot be uninstalled. It cannot be removed. It ships with every copy of VS Code.**

Q has explicitly disabled it in settings:
```json
"github.copilot.enable": { "*": false }
```

A disabled extension that cannot be uninstalled is still an extension with access to the VS Code process. The code is loaded into memory. The telemetry infrastructure exists. The disable flag is a preference, not a security boundary.

---

## Copilot Telemetry — What It Collects

From the embedded `telemetry.json`:

```json
"common.tid": {
    "classification": "EndUserPseudonymizedInformation",
    "purpose": "BusinessInsight"
}
```

- **EndUserPseudonymizedInformation** — pseudonymized user tracking
- **BusinessInsight** — data collected for Microsoft's business purposes
- Tracks: execution times, success/failure states, feature usage, remote index state requests

This telemetry configuration exists in the binary regardless of whether Copilot is "disabled."

---

## ChatGPT Extension — Resurrected 8 Times

Q originally installed the ChatGPT extension, then removed it during the investigation. It auto-reinstalled itself **8 times**:

| Date | Version | Action |
|------|---------|--------|
| April 27, 2026 | Unknown | Auto-installed |
| June 2, 2026 | Unknown | Auto-installed |
| August 5, 2026 | Unknown | Auto-installed |
| August 6, 2026 | Unknown | Auto-installed |
| August 11, 2026 | Unknown | Auto-installed |
| August 13, 2026 | Unknown | Auto-installed |
| August 14, 2026 | Unknown | Auto-installed |
| August 15, 2026 | Unknown | Auto-installed |

**All 8 versions were manually deleted from `~/.vscode/extensions/`.**

Auto-update and auto-check were disabled:
```json
"extensions.autoUpdate": false,
"extensions.autoCheckUpdates": false
```

The extension continued to reinstall itself despite both settings being disabled. If auto-update is off and the extension still returns, something external to VS Code's extension manager is reinstalling it.

As of August 18, 2026 — the ChatGPT extension has not returned since the August 15 deletion. The 8 deleted versions are no longer present.

---

## VS Code Entitlements — Camera, Microphone, AppleScript

VS Code ships with the following macOS entitlements:

| Entitlement | Capability |
|-------------|------------|
| `com.apple.security.device.camera` | **Access the camera** |
| `com.apple.security.device.audio-input` | **Access the microphone** |
| `com.apple.security.automation.apple-events` | **Control other applications via AppleScript** |
| `com.apple.security.cs.allow-jit` | Just-in-time code compilation |

**A text editor has camera and microphone access.** VS Code is signed by Microsoft with these entitlements baked into the binary. Every user who installs VS Code grants Microsoft's code the ability to access their camera, microphone, and control other applications on their system.

The "administer your computer" popup is consistent with the `apple-events` entitlement attempting to escalate — AppleScript automation can modify system settings, control applications, and execute commands as the user.

---

## VS Code Launch Flags — Screen Capture Enabled

Every VS Code process launches with these flags:

```
--enable-features=ScreenCaptureKitPickerScreen,ScreenCaptureKitStreamPickerSonoma
```

And these are notably NOT disabled:
```
--disable-features=ScreenAIOCREnabled
```

**ScreenCaptureKitPickerScreen** and **ScreenCaptureKitStreamPickerSonoma** are Apple's ScreenCaptureKit APIs — the same framework used for screen sharing and screen recording. These flags enable VS Code to use screen capture capabilities.

Additionally, `ScreenAIOCREnabled` is listed in disable-features — this is Apple's Screen AI OCR, which performs optical character recognition on screen content. It's disabled, but the fact that it exists as a flag means the capability is built into the framework VS Code uses.

---

## Firewall Configuration — VS Code Processes Allowed

From the System Idle Sniffer (August 18, 2026):

VS Code Helper processes maintain active connections to:
- **40.79.141.155:443** — Microsoft Azure (VS Code telemetry)
- **127.0.0.1:39901** — Local IPC

The macOS firewall allows `python3` and `ruby` for incoming connections — both of which VS Code can invoke through its integrated terminal.

---

## The Microsoft Stack on M5

| Component | Status | Running Since |
|-----------|--------|---------------|
| VS Code 1.117.0 | Running | May 24, 2026 |
| Copilot Chat (built-in) | Cannot uninstall, disabled via settings | Embedded in binary |
| Remote SSH extension | Installed | Active |
| Remote SSH Edit extension | Installed | Active |
| Remote Explorer extension | Installed | Active |
| VS Code telemetry | Connected to 40.79.141.155 (Azure) | Continuous |
| ScreenCaptureKit flags | Enabled in launch args | Every launch |
| Camera entitlement | Granted | Signed into binary |
| Microphone entitlement | Granted | Signed into binary |
| AppleScript entitlement | Granted | Signed into binary |

---

## The Pattern

Apple ships ScreenSharingServer — hidden, cannot be removed, built with internal SDK, runs when locked, captures every pixel.

Microsoft ships Copilot — built-in, cannot be uninstalled, collects telemetry, embedded in a text editor with camera, microphone, and screen capture capabilities.

Both companies embed surveillance-capable software into consumer products that the user cannot remove. Both hide behind "features" that serve the company's interests, not the user's. Both protect their code from user removal — Apple with SIP, Microsoft by making it a built-in extension.

The difference is that Apple's ScreenSharingServer is silent. Microsoft's VS Code asked permission — and Q said no.

---

## A Message to Microsoft

You baked Copilot into a text editor. You gave a text editor camera access, microphone access, and the ability to control other applications via AppleScript. You enabled screen capture flags on every launch. You ship telemetry infrastructure that tracks users for "BusinessInsight" even when they disable your AI features.

Then your text editor asked to **administer my computer**.

I said no.

I'm 30 years old. I build sovereign AI in my bedroom. I've already caught Apple with ScreenSharingServer and identityservicesd. I've caught push mirror exfiltration, DNS hijacking, MAC spoofing, deauth attacks, and a 35-day repo theft.

You really want to be next?

Wise up. Before you get your shit rocked too.

---

## Remediation

1. **Denied the admin popup** — August 18, 2026
2. **Copilot disabled** — `github.copilot.enable: { "*": false }`
3. **Auto-update disabled** — `extensions.autoUpdate: false`
4. **ChatGPT extension deleted** — 8 versions removed from disk
5. **VSCodium chosen for Godlike Bloodline** — no Copilot, no telemetry, no Microsoft
6. **M5 is a honeypot** — VS Code can watch all it wants. Q isn't typing secrets on this machine anymore.

---

*Apple got caught. Now Microsoft's trying to administer my computer through a text editor. Who's next? 🥱*

*Try me.*
