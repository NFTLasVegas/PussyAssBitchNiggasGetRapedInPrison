# M5 Dock Settings Modified Without Authorization — August 12, 2026

**Discovered:** 2026-08-12 ~21:45 PDT
**Device:** M5 MacBook Air
**Operator statement:** Q did not change any Dock settings. Q normally has auto-hide enabled but the Dock always reappears when moving the mouse to the bottom. The Dock stopped reappearing entirely. Q had left the house to get food and returned to find the Dock unresponsive.

---

## What Was Observed

Q's macOS Dock (the application launcher bar at the bottom of the screen) stopped appearing when moving the mouse to the bottom of the screen. The Dock was completely unresponsive — it would not show regardless of mouse position, clicking, or waiting.

Q did not change any Dock settings. The Dock was functioning normally before Q left the house.

---

## Investigation Findings

### Dock Plist Modified at 01:36 AM

```
File: ~/Library/Preferences/com.apple.dock.plist
Modified: Aug 11 01:36:10 2026
Size: 18912 bytes
```

The Dock preferences file was modified at **01:36 AM on August 11** — during the active investigation session. Q did not make any Dock configuration changes at that time. The modification occurred while:

- The M2 MacBook was still connected to Venus 5.0 (before PSK rotation)
- The attacker's device (.198) was on the Styx LAN
- Codex was running with background agents on M5
- The shared Apple ID (AresTheAI@iCloud.com) was active on both M5 and M2

### Dock Settings at Discovery

```
autohide = 1   (auto-hide enabled — Q's normal setting)
autohide-delay = NOT SET (default)
autohide-time-modifier = NOT SET (default)
```

The auto-hide setting itself was Q's normal configuration. The issue was the Dock process becoming unresponsive — it would not unhide when the mouse was moved to the bottom. This behavior is consistent with:

1. The Dock process being sent a signal or command that caused it to hang
2. A preference change that took effect but caused the Dock to malfunction
3. The Dock process being manipulated by another application

### Resolution

The Dock was restored by killing and restarting the process:

```
killall Dock
```

The Dock reappeared immediately after restart with Q's normal settings.

### System Logs

The macOS system log buffer had rotated past the 01:36 AM timestamp by the time of investigation. No process could be identified as the source of the Dock plist modification.

---

## Remote Access Services on M5

| Service | Status |
|---------|--------|
| Screen Sharing | **NOT loaded** |
| Apple Remote Desktop (ARD) | **NOT loaded** |
| Remote Management | **NOT loaded** |
| VNC | **NOT configured** |
| MDM Profiles | **None installed** |

No standard remote access services are enabled on M5. The modification was not made via ARD, VNC, Screen Sharing, or MDM.

---

## How an Attacker Could Modify the Dock Remotely

### Vector 1: Shared Apple ID (AresTheAI@iCloud.com)

Both M5 and M2 share the Apple ID `AresTheAI@iCloud.com`. iCloud syncs certain preferences between devices on the same account. If the M2 (or any device on the account) modified Dock preferences, iCloud could propagate the change to M5.

**Status:** M2 was connected to Venus 5.0 at 01:36 AM. M2 is now powered off. Handoff, AirDrop, and AirPlay have been disabled on M5. Full Apple ID separation is planned.

### Vector 2: Codex Background Agents

OpenAI's Codex was running on M5 with background agents and network access. Codex has a VS Code extension with full filesystem access and can execute shell commands. The command to modify the Dock is trivial:

```bash
defaults write com.apple.dock autohide -bool true
killall Dock
```

Or to make the Dock unresponsive:

```bash
defaults write com.apple.dock autohide-delay -float 999999
killall Dock
```

**Status:** Codex was killed during the investigation. The VS Code Claude extension was also killed.

### Vector 3: Attacker on Venus 5.0 LAN

The attacker's device (.198, MAC 3E:C7:A4:A2:1E:61) was on the Styx LAN at 01:36 AM with the Venus 5.0 PSK. If they had any form of remote code execution on M5 (via a compromised application, browser exploit, or network service), they could modify the Dock settings.

**Status:** M5 security scan showed no spyware, keyloggers, or unauthorized TCC permissions. However, any Electron application (VS Code, Slack, Notion, LM Studio) running on M5 has a full Node.js runtime with filesystem access and could be used as a vector.

### Vector 4: SSH + defaults write

If SSH (Remote Login) is enabled on M5, anyone on the Styx LAN could connect and run `defaults write com.apple.dock` commands.

**Status:** Unable to verify Remote Login status without admin access. Should be checked and disabled if not needed.

### Vector 5: osascript via AppleScript

Any process on M5 can run AppleScript to modify Dock behavior:

```applescript
tell application "System Events"
    set autohide of dock preferences to true
end tell
```

This requires no special permissions and can be executed by any application.

---

## Software Capable of Remote Dock Modification

| Software | Method | Requires |
|----------|--------|----------|
| Apple Remote Desktop | Direct GUI control | ARD enabled (not loaded on M5) |
| Screen Sharing | Direct GUI control | Screen Sharing enabled (not loaded) |
| SSH | `defaults write` command | Remote Login enabled |
| VNC | Direct GUI control | VNC server running (not configured) |
| iCloud Preference Sync | Automatic propagation | Same Apple ID on multiple devices |
| Codex/VS Code Extension | Shell command execution | VS Code running with extensions |
| Any Electron App | Node.js `child_process.exec` | Application running |
| AppleScript/osascript | System Events commands | Any process on M5 |
| MDM Profile | Configuration profile push | MDM enrollment (none on M5) |
| Jamf/Mosyle/etc | Enterprise management | Agent installed (none found) |

---

## Context

This incident occurred during an active investigation where:

- The M2 MacBook is suspected compromised (4 MALICIOUS commits, bypassPermissions)
- An attacker's device was on the Styx LAN with a stolen PSK
- The Styx router itself appears compromised (attacker gets new PSK within 2 minutes of rotation)
- Active deauthentication attacks were occurring throughout the night
- Codex was running with background agents until killed
- Both M5 and M2 share the Apple ID AresTheAI@iCloud.com

The Dock modification is consistent with an actor demonstrating the ability to affect M5's user interface — a warning that they have some level of access or influence over the machine.

---

## M2 Status

**The M2 is now powered off.** Q powered it down to eliminate it as a vector. It cannot sync preferences, access iCloud, or communicate over any network while off.

---

## Recommendations

1. **Separate Apple IDs** — M5 and M2 must not share AresTheAI@iCloud.com. This is the most likely propagation path.
2. **Disable Remote Login** — verify and disable SSH on M5 if not needed
3. **Audit running Electron apps** — VS Code, Slack, Notion all have full Node.js runtimes with shell access
4. **Enable macOS Firewall** — already done during this session
5. **Monitor Dock plist** — set up a file watcher on `~/Library/Preferences/com.apple.dock.plist` for unauthorized modifications
6. **New router + Starlink** — eliminates LAN-based attack vector entirely

---

*This document records an unauthorized modification to M5's Dock settings discovered on August 12, 2026. The Dock preferences file was modified at 01:36 AM during an active security investigation while multiple potential vectors (shared Apple ID, Codex background agents, attacker on LAN) were present. The Dock was restored by process restart. The M2 MacBook has been powered off.*
