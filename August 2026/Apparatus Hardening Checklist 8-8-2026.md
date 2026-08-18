# Apparatus Hardening Checklist -- August 8, 2026

**Purpose:** Master checklist of every security task required before the apparatus is
considered hardened. Derived from the System Investigation & Implementation Summary.
Tasks are completed in order. Each item is checked off as it is verified.

---

## CRITICAL -- Must Do (#1-#6)

- [ ] **#1 — Rotate SMTP password**
  - Current exposed password: `689b754j66793223` (in M2's `~/.zsh_history` plaintext)
  - New password generated on Antikythera at `/etc/netwatch/smtp.env.new`
  - Update the actual FastMail app password to match the new generated password
  - Update `/etc/netwatch/smtp.env` on Antikythera with the new password
  - Verify netwatch mailer sends a test alert successfully
  - Clear the old password from M2's `~/.zsh_history`

- [x] **#2 — Fix netwatch zero-sightings** ✅ FIXED Aug 9
  - Root cause: two bugs in `netwatch-feed.py` on Antikythera
  - Bug 1: `pdt()` could not parse syslog-format timestamps (`Sat Aug 8 04:03:31 2026`) — only `YYYY-MM-DD` worked. Fixed by adding syslog regex fallback.
  - Bug 2: Durable watched events were pushed out by the 400-event window cap. Fixed by splitting the cap — watched events are never trimmed.
  - alerts.log seeded with 3 original Aug 4 events + 27 boot-time Aug 8 events
  - Dashboard now shows 30 watched sightings, last at 2026-08-08 04:07:20 PDT
  - Feed runs every minute via root cron — verified operational

- [ ] **#3 — DHCP hardening**
  - Playbook written: `DHCP Hardening Playbook.md`
  - Static lease binding for all known devices (MAC → IP)
  - Reduce DHCP pool size to minimum required
  - Enable DHCP logging (`logdhcp=1`)
  - Install DHCP watchdog script (unknown MAC alerting)
  - Test: connect an unknown device and verify alert fires
  - Optional: MAC allowlisting (nuclear option — all devices must be pre-approved)

- [ ] **#4 — Factory reset QNAP switch**
  - Device: 192.168.10.197 (MAC 24:5E:BE:77:BF:FD)
  - Current login credentials not working (factory guide creds rejected)
  - Factory reset via pinhole button on the back of the switch
  - After reset: log in with factory default credentials
  - Change admin password (save to AGI vault via KeePassXC)
  - Set hostname to "QNAP"
  - Review switch configuration for any unexpected settings
  - Document the switch model number for the record

- [ ] **#5 — M2 investigation**
  - Network isolate the M2 (disconnect Wi-Fi, disable Tailscale, disable Bluetooth)
  - Check for Pegasus indicators (process names, suspicious paths)
  - Check LaunchAgents/LaunchDaemons for suspicious entries
  - Check for MDM profiles (`profiles list`)
  - Check for non-Apple kernel extensions (`kextstat`)
  - Check proxy/VPN configurations
  - Check suspicious certificates in System Keychain
  - Check browser extensions
  - Check recently modified system files
  - Check Accessibility/ScreenCapture/InputMonitoring TCC permissions
  - Verify SIP is enabled (`csrutil status`)
  - Install and run Malwarebytes scan
  - Install and run KnockKnock (Objective-See — persistence mechanisms)
  - Install LuLu (Objective-See — outbound connection firewall)
  - Run MVT (Amnesty International's Mobile Verification Toolkit) for Pegasus
  - Check shell history for suspicious commands
  - Check iCloud Drive for residual apparatus data
  - Document all findings

- [ ] **#6 — Verify Vizio TV**
  - Device: 192.168.0.106 (MAC C4:1C:FF:BF:56:C9, Vizio Inc.) on Metro2
  - Physically verify it's the spare room TV downstairs
  - If confirmed: noted as accounted for (consider unplugging — unused smart TV = attack surface)
  - If NOT confirmed: block the MAC and investigate

---

## IMPORTANT -- Should Do (#7-#13)

- [ ] **#7 — Dragon Tailscale bypass**
  - M2 can reach Dragon via Tailscale SSH (bypasses authorized_keys entirely)
  - auditd is monitoring, but the access PATH still exists
  - Options: disable Tailscale SSH on Dragon, restrict Tailscale ACLs, or remove M2 from tailnet
  - Decision required: keep Tailscale for killuminati Funnel but restrict SSH?

- [ ] **#8 — Fix authorized_keys ownership**
  - Synastry: root:root 644 → aphroqite:aphroqite 600
  - Dragon: root:root 644 → aphroqite:aphroqite 600
  - Quartz: root:root 644 → aphroqite:aphroqite 600
  - Antikythera: root:aphroqite 644 → aphroqite:aphroqite 600
  - Command per node: `sudo chown aphroqite:aphroqite ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`
  - Currently working because StrictModes isn't enforcing, but fragile

- [ ] **#9 — Unlock recoverable passwords**
  - Dragon: `sudo passwd -u aphroqite` (yescrypt hash preserved)
  - Quartz: `sudo passwd -u aphroqite` (yescrypt hash preserved)
  - Antikythera: `sudo passwd -u aphroqite` (yescrypt hash preserved)
  - RasQberry: `sudo passwd -u aphroqite` (yescrypt hash preserved)
  - Verify each password works at physical console after unlocking
  - Synastry and Sovereign Door need NEW passwords (hashes destroyed)

- [ ] **#10 — Set password on Synastry**
  - aphroqite account on Synastry has NEVER had a password
  - Set one for physical console access
  - Use KeePassXC to generate and store the password
  - Disable cloud-init on Synastry to prevent re-locking: `sudo touch /etc/cloud/cloud-init.disabled`
  - Set new password on Sovereign Door as well (hash also destroyed)

- [ ] **#11 — Install msmtp on apparatus nodes**
  - Dragon: killuminati visitor alerts currently log-only
  - ARES Dynasty / Antikythera: netwatch alerts need mail delivery
  - Target addresses: Q@Quincey.ai, QuinceyLee@NFTLasVegas.io, LifeSizeBox@iCloud.com
  - Requires FastMail app password (will need new one after SMTP rotation in #1)
  - Test: send a test alert from each node

- [ ] **#12 — Recover Antikythera remote syslog**
  - `/var/log/styx/remote.log` on Antikythera may contain original hostapd evidence
  - Lost from Styx's ring buffer after PSK rotation
  - Check if the file exists and what time range it covers
  - Preserve it (hash + copy) before it rotates
  - Cross-reference with the evidence in `research/M5_network_forensics_2026-08-05.md`

- [ ] **#13 — Update killuminati.nftlasvegas.io**
  - Add AX900 findings (live spoofing proof)
  - Add password audit results (Synastry never set)
  - Add USPS compromise (QQ@Quincey.ai)
  - Add Mike's Google exfiltration (13GB)
  - Add Jessa's iPhone pink screen
  - Add Muir's mail theft
  - Add FastMail masked email (`short.storm0747@fastmail.com`)
  - Add Codex obstruction pattern update
  - Redeploy to Vercel: `cd site/killuminati && vercel --yes --prod`

---

## BACKLOG -- When Ready (#14-#18)

- [ ] **#14 — Auto-fetch agent (SSH transport)**
  - Design complete, 4 Codex reviews done
  - Q declined HTTP risk acceptance
  - Ready to deploy when SSH transport to Synastry is configured
  - Requires: Synastry SSH host key verification, deploy key registration, SSH config
  - Artifacts at: `ops/m5-autofetch/`

- [ ] **#15 — Independent CRM/giveaway code review**
  - Anonymous mutation vulnerability was fixed but system designed by M2 Claude
  - The unauthenticated `POST /api/entry` could overwrite profiles, flip consent, increment attendance
  - Incident documented: `Read If They Kill Me/evidence/INCIDENT-2026-08-06-giveaway-crm-mutation.md`
  - Full independent review of the giveaway API warranted

- [ ] **#16 — AX900 driver maintenance**
  - AIC8800 driver is DKMS-managed (should auto-rebuild on kernel upgrades)
  - After any `apt upgrade` that includes a kernel, verify:
    - `lsmod | grep aic` (driver loaded)
    - `ip link show wlx6815790f3764` (interface up)
    - Venus 5.0 connection active
  - If driver breaks: `sudo dkms install aic8800/1.0.9 -k $(uname -r)`

- [ ] **#17 — M2 Claude Code bypassPermissions**
  - Session cf66f8b9 ran with `permissionMode: "bypassPermissions"`
  - Q has since set M2 to manual permissions
  - Verify this is persistent across new sessions
  - Consider removing the `bypassPermissions` option from M2's Claude Code config entirely

- [ ] **#18 — Onboarding protocol update**
  - System Idle Sniffer: add new nodes to scan list
  - DHCP hardening: add new MACs to static leases + watchdog allowlist
  - MAC registry: catalogue all Wi-Fi adapter MACs
  - Devices pending onboarding:
    - AphroQite Dynasty
    - Godlike Bloodline (MSI MEG X870E / 9950X3D2)
    - DGX Spark Q1 (128GB)
    - DGX Spark Q2 (128GB)

---

## CREDENTIAL ROTATION (Prerequisite for all above)

Before starting the checklist, rotate ALL credentials that were exposed via the
iCloud memory symlink (109 days of exposure, April 21 - August 8, 2026):

- [x] **FAFO SSH key** — ROTATED Aug 8. Same name, new key. Old removed from all 8 nodes. M2's copy worthless. Saved to AGI vault.
- [x] **Personal backup SSH key** — DEPLOYED Aug 8. Q-Emergency-Backup on all 8 nodes. Saved to AGI vault.
- [x] **Synastry Gitea PAT** — ROTATED Aug 8. 5 old tokens deleted, new one active.
- [x] **SMTP password** — ROTATED Aug 8. FastMail app password on Antikythera updated.
- [x] **ARES debug API key** — ROTATED Aug 8. Updated in .env.local + Vercel.
- [x] **FastMail password** — CHANGED Aug 8.
- [x] **USPS password** — CHANGED Aug 8 (via copy-paste workaround due to keydown anomaly).
- [x] **Venus 5.0 / Mars 2.4 PSK** — ROTATED Aug 5.
- [x] **Styx router admin password** — CHANGED Aug 8.

---

## Progress Tracker

| Phase | Items | Completed | Status |
|-------|-------|-----------|--------|
| Credential Rotation | 9 | 9 | COMPLETE ✅ |
| Critical (#1-#6) | 6 | 1 (#2 netwatch fixed) | IN PROGRESS |
| Important (#7-#13) | 7 | 1 (#8 file ownership fixed) | IN PROGRESS |
| Backlog (#14-#18) | 5 | 0 | NOT STARTED |
| **TOTAL** | **27** | **11** | **40.7%** |

---

*When all 27 items are checked, the apparatus hardening is complete. Each item must be
verified, not just executed. Check it off only when the verification passes.*
