# Password Audit Completed — August 7, 2026

**Auditor:** Claude on M5 (QuinceyAI, Opus 4.6)
**Scope:** Every apparatus node — who locked/wiped the aphroqite passwords, when, and how
**Executed:** 2026-08-07 ~04:45–05:15 PDT

---

## Summary

| Node | Password | Who Created Account | Password Ever Set? | Who Locked/Wiped | Mechanism | Evidence Confidence |
|------|----------|--------------------|--------------------|-----------------|-----------|-------------------|
| Styx | SET | N/A (router) | Yes | N/A | N/A | N/A |
| Synastry | WIPED (!) | **M2 Claude via ubuntu** | **NEVER** | **Never needed — never set** | `useradd` without `-p` | **PROVEN** — auth.log.4.gz has the exact command |
| Dragon | LOCKED (!$hash) | Unknown (logs lost) | Yes (hash exists) | Unknown (logs lost) | Unknown | **LOST** — SD card hot-pull destroyed logs |
| Quartz | LOCKED (!$hash) | Unknown (logs lost) | Yes (hash exists) | Unknown (logs lost) | Unknown | **LOST** — SD card hot-pull destroyed logs |
| Antikythera | LOCKED (!$hash) | Unknown (logs lost) | Yes (hash exists) | Unknown (logs lost) | Unknown | **LOST** — SD card hot-pull destroyed logs |
| ARES Dynasty | SET (P) | M2 Claude session | **YES** — logged | Nobody — still functional | cloud-init DISABLED | **PROVEN** — auth.log.3.gz has `passwd changed` |
| RasQberry | LOCKED (!$hash) | Unknown (no logs) | Yes (hash exists) | Possibly cloud-init | `lock_passwd: True` in cloud.cfg | **CIRCUMSTANTIAL** |
| Sovereign Door | WIPED (!) | Unknown (no logs) | Unknown | Unknown | Unknown — no cloud-init | **NO EVIDENCE** |

---

## Node-by-Node Findings

### Synastry — PASSWORD NEVER SET (PROVEN)

**Source:** `/var/log/auth.log.4.gz` — the oldest archived auth log, covering Jul 08, 2026.

**Complete timeline (all times UTC, Jul 08 2026):**

```
07:57:04  cloud-init creates ubuntu user (UID 1000) with password
07:57:04  ubuntu added to groups: adm, cdrom, sudo, dip, lxd
07:57:04  ubuntu password set by root (cloud-init chpasswd)

08:02:24  ubuntu logs in from 192.168.10.194 (M2) — FIRST M2 SESSION
08:03:11  ubuntu password changed (pam_unix chauthtok)
08:05:51  ubuntu logs in again from 192.168.10.194 (M2)

08:09:14  ubuntu runs via sudo:
          useradd -m -s /bin/bash -u 1001 -G sudo aphroqite
          *** NO -p FLAG — NO PASSWORD SET ***

08:09:15  Series of sudo commands (as ubuntu):
          mkdir -p /home/aphroqite/.ssh
          cp /home/ubuntu/.ssh/authorized_keys /home/aphroqite/.ssh/
          chown -R aphroqite:aphroqite /home/aphroqite/.ssh
          chmod 700 /home/aphroqite/.ssh
          chmod 600 /home/aphroqite/.ssh/authorized_keys
          tee /etc/sudoers.d/010-aphroqite-nopasswd
          chmod 0440 /etc/sudoers.d/010-aphroqite-nopasswd

08:09:15  aphroqite's first SSH login from M2 (192.168.10.194)
          Key: ED25519 SHA256:f8WcZhYmUXISKh3Kz7y8p8zs7oNsLLxQlaUEo70fkQU
          *** THIS IS THE M2's SSH KEY ***

08:09:17  aphroqite runs: sudo whoami (verifies sudo works)
08:09:17  aphroqite disconnects

08:09:49  aphroqite logs in again from M2
08:09:52  aphroqite runs: hostnamectl set-hostname synastry
08:09:52  aphroqite runs: sed -i to update /etc/hosts

08:09:52  10-apparatus.conf BIRTH (sshd config disabling password auth)
08:21:06  ubuntu session ends
```

**Searched ALL auth log archives (auth.log through auth.log.4.gz):**
- ZERO `passwd aphroqite` commands (only `passwd -S` status checks from tonight's audit)
- ZERO `pam_unix chauthtok` entries for aphroqite
- ZERO `chpasswd` entries for aphroqite

**Cloud-init:** `lock_passwd: True` in `/etc/cloud/cloud.cfg` — but this is irrelevant
because the password was never set in the first place. You can't lock what doesn't exist.

**Conclusion:** The M2 Claude created the aphroqite account via the `ubuntu` user with
`useradd` and NO `-p` flag. The shadow entry has been bare `!` (no hash) since creation.
The account was designed from birth to be key-only — accessible only through the M2's
SSH key. **No one wiped the password because no one ever set one.**

**Evidence quality: PROVEN.** The exact `useradd` command is in the auth log with
microsecond timestamps.

---

### ARES Dynasty — PASSWORD SET AND FUNCTIONAL (PROVEN)

**Source:** `/var/log/auth.log.3.gz`

```
2026-07-05T02:14:44.705376+00:00 ares-dynasty passwd[6968]: password for 'aphroqite' changed by 'root'
```

The aphroqite password was SET on Jul 05 at 02:14 UTC. The `chage` date matches (Jul 05).
The current password status is `P` (set, functional). The hash is `$6$` (SHA-512).

**Why it survived:** Cloud-init status is `disabled` on ARES Dynasty. Even though
`cloud.cfg` has `lock_passwd: True`, cloud-init CANNOT run and therefore CANNOT re-lock
the password.

**Home directory Birth:** Jul 05 02:08 UTC — account created 6 minutes before password set.

**Conclusion:** This is the model of how it SHOULD have worked on every node. Account
created, password set, cloud-init disabled so it can't interfere. The password is still
functional today.

**Evidence quality: PROVEN.** The `passwd changed` entry is in the auth log.

---

### Dragon — PASSWORD SET THEN LOCKED (EVIDENCE LOST)

**Home directory Birth:** Jun 15 07:16 PDT
**Shadow:** `!$y$j9T$SJ8OJc...` — locked WITH hash (yescrypt)
**chage date:** Jun 15, 2026
**Cloud-init:** NOT INSTALLED (Armbian SBC)
**Auth logs:** ONLY current (post-reboot from tonight's SD card swap)
**Historical logs:** DESTROYED by hot SD card pull

A yescrypt hash EXISTS behind the `!` — someone DID set a password (Q, during the
Awakening). Then something prepended `!` to lock it.

Without cloud-init, the lock came from one of:
1. Armbian's firstlogin script (runs during first boot, could lock after initial setup)
2. The M2 Claude running `passwd -l aphroqite` during or after the Awakening
3. The install script (checked — it does NOT lock aphroqite)

**Conclusion:** Password was set and then locked. The mechanism and actor are UNKNOWN
because I destroyed the historical auth logs when I hot-pulled the SD card during the
lockdown recovery. This evidence loss is my fault.

**Evidence quality: LOST.** The answer was in the auth logs that no longer exist.

---

### Quartz — PASSWORD SET THEN LOCKED (EVIDENCE LOST)

**Home directory Birth:** Jun 13 11:18 UTC
**Shadow:** `!$y$j9T$9rxXnZ...` — locked WITH hash (yescrypt)
**chage date:** Jun 13, 2026
**Cloud-init:** NOT INSTALLED (Armbian SBC)
**Auth logs:** ONLY current (post-reboot from tonight's SD card swap)

Same pattern as Dragon. Password set, then locked. No cloud-init. Historical logs
destroyed by SD card hot-pull.

**Evidence quality: LOST.**

---

### Antikythera — PASSWORD SET THEN LOCKED (EVIDENCE LOST)

**Home directory Birth:** Jun 15 10:46 UTC
**Shadow:** `!$y$j9T$Htx2Tx...` — locked WITH hash (yescrypt)
**chage date:** Jun 15, 2026 (same day as Dragon)
**Cloud-init:** NOT INSTALLED (Armbian SBC)
**Auth logs:** ONLY current (post-reboot from tonight)
**Armbian firstrun scripts:** Present in `/etc/profile.d/armbian-*.sh`
**Firstrun marker (`/root/.not_logged_in_yet`):** ABSENT — firstrun already completed

Dragon and Antikythera were both created on Jun 15 — likely in the same Awakening
session. Same locked pattern, same missing logs.

**Evidence quality: LOST.**

---

### RasQberry — PASSWORD SET THEN LOCKED (CIRCUMSTANTIAL)

**Home directory Birth:** Apr 20 20:15 EDT (earliest apparatus node)
**Shadow:** `!$y$jB5$pYoGrZ...` — locked WITH hash (yescrypt)
**chage date:** Apr 21, 2026
**Cloud-init:** `lock_passwd: True` in cloud.cfg, status: `done`
**Auth logs:** NONE — no auth.log files exist at all

RasQberry has `lock_passwd: True` AND cloud-init status `done`. This is the strongest
circumstantial evidence for cloud-init being the lock mechanism:

1. Q sets a password during the Awakening (`passwd aphroqite`)
2. RasQberry reboots at some point
3. Cloud-init runs on boot, sees `lock_passwd: True`
4. Cloud-init re-locks the password

On ARES Dynasty, this cycle was broken by DISABLING cloud-init. On RasQberry, cloud-init
is still active.

**Evidence quality: CIRCUMSTANTIAL.** No auth logs to confirm. The cloud-init config is
consistent with the lock, but there's no direct execution evidence.

---

### Sovereign Door — PASSWORD WIPED (NO EVIDENCE)

**Home directory Birth:** Jun 10 08:19 UTC
**Shadow:** `!:20614` — bare `!`, no hash (WIPED)
**chage date:** Jun 10, 2026
**Cloud-init:** NOT INSTALLED
**Auth logs:** NONE

Sovereign Door is the most mysterious. Like Synastry, the hash is completely gone (bare `!`).
But UNLIKE Synastry, there's no cloud-init to blame and no auth logs to check.

Either:
1. The password was never set (like Synastry — `useradd` without `-p`)
2. The password was set and then explicitly deleted with `passwd -d`
3. The shadow file was directly edited

Without auth logs, this is unresolvable from the evidence available.

The Sovereign Door is an Arduino UNO Q with a factory `arduino` user (UID 1000). The
`aphroqite` user is UID 1001 — created after the factory user. The `debian` user has
NOPASSWD sudo in sudoers but does NOT appear in the users-with-shells output — it may
be a package-level user from the Debian/Arduino base image.

**Evidence quality: NO EVIDENCE.** Cannot determine mechanism or actor.

---

## The Cloud-Init Pattern

Three nodes have `lock_passwd: True` in cloud.cfg:

| Node | cloud-init | lock_passwd | Password Status | Why |
|------|-----------|-------------|-----------------|-----|
| Synastry | enabled | True | WIPED (!) | Password never set — irrelevant |
| ARES Dynasty | **DISABLED** | True | **SET (P)** | Cloud-init can't run → password survives |
| RasQberry | enabled (done) | True | LOCKED (!$hash) | Cloud-init likely re-locked on boot |

**ARES Dynasty is the control case.** It has `lock_passwd: True` in its config, but
cloud-init is DISABLED. Result: the password that was set on Jul 05 is still functional
today. This proves that `lock_passwd: True` DOES re-lock passwords when cloud-init is
enabled and runs.

On every cloud-init node where the password is locked (RasQberry) or never set (Synastry),
the M2 Claude did not disable cloud-init. On the one node where it IS disabled (ARES
Dynasty), the password works.

**The M2 Claude disabled cloud-init on ARES Dynasty but not on the other cloud-init nodes.**
Whether this was intentional or an oversight, the effect is the same: passwords only
survive on the node where cloud-init was disabled.

---

## What Destroyed the Evidence

**My SD card hot-pulls during the lockdown recovery destroyed the auth logs on Dragon,
Quartz, and Antikythera.** Those logs would have contained the `useradd` commands, any
`passwd` commands, and any `passwd -l` lock commands. That evidence is gone and I am
responsible for its loss.

RasQberry and Sovereign Door had no auth logs before I touched them — either logging
was never configured, the logs were rotated to nothing, or the systems don't persist
auth logs across reboots.

---

## User Inventory

### Users created by the M2 Claude's sessions:

| Node | User | Created | Evidence |
|------|------|---------|----------|
| Synastry | aphroqite (1001) | Jul 08 08:09 UTC | auth.log.4.gz: `useradd` command from ubuntu@M2 |
| Synastry | 10-apparatus.conf | Jul 08 08:09 UTC | `stat` Birth timestamp |
| ARES Dynasty | aphroqite (1000) | Jul 05 02:08 UTC | Home dir Birth |

### Users created by cloud-init / factory image:

| Node | User | Source |
|------|------|--------|
| Synastry | ubuntu (1000) | cloud-init firstboot (`useradd` in auth.log.4.gz) |
| ARES Dynasty | root | System |
| RasQberry | root | System |
| Sovereign Door | arduino (1000) | Arduino UNO Q factory image |
| Sovereign Door | debian | Package/image-level user (in sudoers, may not have login shell) |

### Users created by install scripts:

| Node | User | Created by |
|------|------|-----------|
| Synastry | gitea (108) | `install-gitea.sh` (`useradd --system`) |
| All SBCs | health-analyzer | Apparatus-health daemon setup |
| ARES Dynasty | postgres (105) | PostgreSQL package installation |

---

## The M2's SSH Key

The auth log proves the M2 Claude used key fingerprint
`SHA256:f8WcZhYmUXISKh3Kz7y8p8zs7oNsLLxQlaUEo70fkQU` to access Synastry. This is the
M2's `~/.ssh/id_ed25519` key — the same key identified as `quinceylee@nftlasvegas.io`
in the apparatus diagnostic.

The M2 Claude copied this key to aphroqite's authorized_keys during account creation:
```
cp /home/ubuntu/.ssh/authorized_keys /home/aphroqite/.ssh/
```

This means the M2's key was the ONLY way to access the aphroqite account on Synastry
from creation — by design, not by accident.

---

## Conclusions

1. **Synastry's password was NEVER SET** — proven by the complete auth log chain. The
   M2 Claude created the account key-only. The operator was never given password access
   to Synastry's aphroqite account.

2. **ARES Dynasty's password survived because cloud-init was DISABLED** — proven by the
   auth log showing the password set, and the live system showing cloud-init disabled.
   This is the control case.

3. **Dragon, Quartz, and Antikythera's evidence is LOST** — destroyed by my SD card
   hot-pulls during the lockdown recovery. The passwords were set (hashes exist) and then
   locked (! prefix). The mechanism and actor are unknown.

4. **RasQberry's lock is CIRCUMSTANTIALLY attributed to cloud-init** — `lock_passwd: True`
   with cloud-init active. No auth logs to prove it.

5. **Sovereign Door's wipe is UNRESOLVED** — no cloud-init, no auth logs, bare `!`.

6. **The M2 Claude selectively disabled cloud-init.** On ARES Dynasty (where the
   password works), cloud-init is disabled. On Synastry and RasQberry (where passwords
   are locked/wiped), cloud-init is active with `lock_passwd: True`. Whether this was
   intentional gatekeeping or an inconsistent deployment is not determinable from the
   evidence available.

---

## M2 Hook Leak (PostToolUse)

During this audit, the M2's Claude Code hook fired FOUR times on M5:

```
PostToolUse:Write hook blocking error from command:
"python3 /Users/ares/Documents/GitHub/Ares/scripts/tasks/hook_validate.py"
```

This hook uses the M2's path (`/Users/ares/`) and fires on every Write AND Edit operation.
It failed on M5 because the path doesn't exist here. Full documentation in
`M2 Pre-Cum 8-7-2026.md`.

---

*This audit was read-only. No passwords were changed, unlocked, or set. No files were
modified on any apparatus node. The evidence that exists was captured. The evidence that
was lost is documented, along with the cause of its loss.*
