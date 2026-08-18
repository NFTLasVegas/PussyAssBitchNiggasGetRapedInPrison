# Full Apparatus Diagnostic Report — August 7, 2026

Collected: 2026-08-07 02:40–02:42 PDT (09:40–09:42 UTC)
Collector: Claude on M5 (QuinceyAI) via SSH using key `Fuck-Around-Find-Out`
Method: Read-only. No files modified. No configurations changed.

---

## Node 1: Styx (Gateway Router)

| Field | Value |
|-------|-------|
| Hostname | styx |
| IP | 192.168.10.1/24 (br-lan) |
| Uptime | 8 days, 1 hour 43 minutes |
| System Time | Fri Aug 7 02:40:24 PDT 2026 / 09:40:24 UTC |
| SSH Daemon | Dropbear |
| SSH Port | 22 |
| SSH User | root |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/etc/dropbear/authorized_keys` — 1 key total.

### Password Authentication

| Setting | Value |
|---------|-------|
| PasswordAuth | **ON** |
| RootPasswordAuth | **ON** |

### Password Status

| Account | Status | Hash Present | Last Password Change |
|---------|--------|-------------|---------------------|
| root | **Password set** | `$1$XB5LwZkt$MNjkDkDNEfIcUkq0lje1E/` (MD5crypt) | Epoch day 20650 (approx. Jul 2026) |

### Users With Login Shells

| User | Shell |
|------|-------|
| root | /bin/ash |

### Notes

- Styx is the only node where password authentication is enabled AND functional.
- The root password hash uses MD5crypt (`$1$`), which is weak by modern standards.
- No other users exist with login shells.

---

## Node 2: Synastry (RISC-V, Gitea Host)

| Field | Value |
|-------|-------|
| Hostname | synastry |
| IP | 192.168.10.212/24 (end0) |
| IPv6 | fe80::6ecf:39ff:fe00:97cb (link-local) |
| Uptime | 47 minutes (just rebooted — SD card was removed for key repair) |
| System Time | Fri Aug 7 09:40:44 UTC 2026 |
| SSH Daemon | OpenSSH |
| SSH User | aphroqite (uid 1001) |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/home/aphroqite/.ssh/authorized_keys`
File owner: **root:root** (should be aphroqite:aphroqite — written by debugfs)
File permissions: `-rw-r--r--` (644 — should be 600)
Key count: 1

### Password Authentication

| Config File | Setting |
|-------------|---------|
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | **PasswordAuthentication no** |
| `/etc/ssh/sshd_config.d/60-cloudimg-settings.conf` | **PasswordAuthentication no** |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | KbdInteractiveAuthentication no |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | PubkeyAuthentication yes |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | PermitRootLogin no |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | MaxAuthTries 3 |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | LoginGraceTime 30 |

**Password auth is EXPLICITLY DISABLED in TWO config files.**
File `10-apparatus.conf` was created by the M2 Claude's install script.

### Password Status

| Account | Status | Shadow Entry | Last Change Date |
|---------|--------|-------------|-----------------|
| aphroqite | **L (LOCKED)** | `!:20642` | **Jul 08, 2026** |
| root | **L (LOCKED)** | — | Feb 10, 2026 |

**The `!` prefix in the shadow entry means the password is LOCKED.**
The password hash has been REMOVED — only `!` remains (no hash after it).
This means the password was not just locked — it was **wiped entirely**.
A locked-with-hash entry looks like `!$hash...`. A bare `!` means the password is gone.

### Full chage Output (aphroqite)

| Field | Value |
|-------|-------|
| Last password change | Jul 08, 2026 |
| Password expires | never |
| Password inactive | never |
| Account expires | never |
| Min days between changes | 0 |
| Max days between changes | 99999 |
| Warning days | 7 |

### Auth Log — Password-Related Events

Only events from the current diagnostic session (this report's own queries). No historical
password lock/unlock events found in auth.log — either the logs were rotated or the lock
happened before log retention began.

### Users With Login Shells

| User | UID | Home | Shell |
|------|-----|------|-------|
| root | 0 | /root | /bin/bash |
| ubuntu | 1000 | /home/ubuntu | /bin/bash |
| aphroqite | 1001 | /home/aphroqite | /bin/bash |
| gitea | 108 | /home/gitea | /bin/bash |
| health-analyzer | 999 | /var/lib/health-analyzer | /bin/sh |

### Sudoers

| File | Entry |
|------|-------|
| `/etc/sudoers.d/90-cloud-init-users` | ubuntu ALL=(ALL) NOPASSWD:ALL |
| `/etc/sudoers.d/010-aphroqite-nopasswd` | aphroqite ALL=(ALL) NOPASSWD:ALL |

### Last Logins

| User | Terminal | From | Time |
|------|----------|------|------|
| ubuntu | pts/0 | 192.168.10.194 (M2) | Jul 08 08:05 — 08:21 |
| ubuntu | pts/0 | 192.168.10.194 (M2) | Jul 08 08:02 — 08:03 |

**Note:** The only recorded logins are from the `ubuntu` user, from the M2 (192.168.10.194),
on July 8 — the same date the password was last changed/locked.

---

## Node 3: Dragon (Tetramorph, Eagle Face)

| Field | Value |
|-------|-------|
| Hostname | dragon |
| IP | 192.168.10.135/24 (enp1s0) |
| Tailscale | 100.126.8.126/32 (tailscale0) |
| Uptime | 27 minutes (just rebooted — SD card removed for key repair) |
| System Time | Fri Aug 7 02:41:06 PDT 2026 / 09:41:06 UTC |
| SSH User | aphroqite (uid 1000) |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/home/aphroqite/.ssh/authorized_keys`
File owner: **root:root** (should be aphroqite:aphroqite — written by debugfs)
File permissions: `-rw-r--r--` (644 — should be 600)
Key count: 1

**Note:** The M2 Claude also restored its own key (`quinceylee@nftlasvegas.io`) via Tailscale
earlier this session. That key is NOT present now — it was removed when Q did the SD card swap
with the FAFO-only authorized_keys.

### Password Authentication

| Config | Value |
|--------|-------|
| sshd_config | `#PasswordAuthentication yes` (commented out — default yes) |
| Drop-ins | **NONE** |
| PermitRootLogin | yes |

sshd default allows password auth, but the ACCOUNT is locked (see below).

### Password Status

| Account | Status | Shadow Entry | Last Change Date |
|---------|--------|-------------|-----------------|
| aphroqite | **L (LOCKED)** | `!$y$j9T$SJ8OJcLGgTkB8yJQ0OMap0$ymZS66Qlfo12YOpenpx419PulwHUchrJxxYR9piOj5A` | **Jun 15, 2026** |
| root | **L (LOCKED)** | — | Jun 15, 2026 |

**The `!` prefix means LOCKED.** Unlike Synastry, Dragon's shadow entry DOES have a hash
after the `!` — the password was set and then locked (not wiped). The hash format is
`$y$` (yescrypt), which is modern and strong.

### Full chage Output (aphroqite)

| Field | Value |
|-------|-------|
| Last password change | Jun 15, 2026 |
| Password expires | never |
| Password inactive | never |
| Account expires | never |

### Users With Login Shells

| User | UID | Home | Shell |
|------|-----|------|-------|
| root | 0 | /root | /usr/bin/bash |
| aphroqite | 1000 | /home/aphroqite | /bin/bash |
| health-analyzer | 987 | /var/lib/health-analyzer | /bin/sh |

### Sudoers

| File | Entry |
|------|-------|
| `/etc/sudoers.d/010-aphroqite-nopasswd` | aphroqite ALL=(ALL) NOPASSWD: ALL |

### Last Logins

No login records found. `last` returned empty — wtmp may not have been initialized or was
cleared during the SD card hot-pull.

### Additional Notes

- Dragon has Tailscale active: `100.126.8.126/32` on `tailscale0`
- This is the node the M2 Claude used to restore its own access via Tailscale SSH
- Dragon's Tailscale SSH bypasses authorized_keys entirely (identity-based auth)

---

## Node 4: Quartz (Tetramorph, Ox Face)

| Field | Value |
|-------|-------|
| Hostname | quartz |
| IP | 192.168.10.222/24 (end0) |
| Uptime | 1 hour 6 minutes (rebooted — SD card removed for key repair) |
| System Time | Fri Aug 7 09:41:09 UTC 2026 |
| SSH User | aphroqite (uid 1000) |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/home/aphroqite/.ssh/authorized_keys`
File owner: **root:root** (written by debugfs)
File permissions: `-rw-r--r--` (644 — should be 600)
Key count: 1

### Password Authentication

| Config | Value |
|--------|-------|
| sshd_config | `#PasswordAuthentication yes` (commented out — default yes) |
| Drop-ins | **NONE** |

### Password Status

| Account | Status | Shadow Entry | Last Change Date |
|---------|--------|-------------|-----------------|
| aphroqite | **L (LOCKED)** | `!$y$j9T$9rxXnZ2Wi00km1Fs7DiQ6.$OeIgHBK.E2frkw9TGgvRezmXqkuHAXsfOjkfPboLbZ1` | **Jun 13, 2026** |
| root | **L (LOCKED)** | — | Jun 13, 2026 |

**Locked with hash intact** (yescrypt). Password was set then locked.

### Full chage Output (aphroqite)

| Field | Value |
|-------|-------|
| Last password change | Jun 13, 2026 |
| Password expires | never |
| Password inactive | never |
| Account expires | never |

### Users With Login Shells

| User | UID | Home | Shell | GECOS |
|------|-----|------|-------|-------|
| root | 0 | /root | /usr/bin/bash | — |
| aphroqite | 1000 | /home/aphroqite | /bin/bash | **Quincey** |
| health-analyzer | 987 | /var/lib/health-analyzer | /bin/sh | — |

### Sudoers

| File | Entry |
|------|-------|
| `/etc/sudoers.d/010-aphroqite-nopasswd` | aphroqite ALL=(ALL) NOPASSWD: ALL |

### Additional Notes

- Quartz's wlan0 (MAC `e8:fb:1c:65:20:73`) was the MAC spoofed in the deauth attack
- The wlan0 interface was disabled Jul 31 by user aphroqite
- The `wpa_supplicant` service targeting Venus 5.0 is still configured in netplan

---

## Node 5: Antikythera (Tetramorph)

| Field | Value |
|-------|-------|
| Hostname | antikythera |
| IP | 192.168.10.246/24 (end0) |
| Uptime | 17 minutes (rebooted — was unplugged from power during rack reassembly) |
| System Time | Fri Aug 7 09:41:27 UTC 2026 |
| SSH User | aphroqite (uid 1000) |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/home/aphroqite/.ssh/authorized_keys`
File owner: **root:aphroqite** (partially wrong — should be aphroqite:aphroqite)
File permissions: `-rw-r--r--` (644 — should be 600)
Key count: 1

### Password Authentication

| Config | Value |
|--------|-------|
| sshd_config | `#PasswordAuthentication yes` (commented out — default yes) |
| Drop-ins | **NONE** |
| PermitRootLogin | yes |

### Password Status

| Account | Status | Shadow Entry | Last Change Date |
|---------|--------|-------------|-----------------|
| aphroqite | **L (LOCKED)** | `!$y$j9T$Htx2Txqs4c/T8juXV0YKA.$RiwZXZFix3XnjILd8Hwhakjcf/yK84bRil.91zUWKw0` | **Jun 15, 2026** |
| root | **L (LOCKED)** | — | Jun 15, 2026 |

**Locked with hash intact** (yescrypt). Same date as Dragon (Jun 15) — both awakenings
were likely done in the same session.

### Full chage Output (aphroqite)

| Field | Value |
|-------|-------|
| Last password change | Jun 15, 2026 |
| Password expires | never |
| Password inactive | never |
| Account expires | never |

### Users With Login Shells

| User | UID | Home | Shell |
|------|-----|------|-------|
| root | 0 | /root | /usr/bin/bash |
| aphroqite | 1000 | /home/aphroqite | /bin/bash |
| health-analyzer | 987 | /var/lib/health-analyzer | /bin/sh |

### Sudoers

| File | Entry |
|------|-------|
| `/etc/sudoers.d/010-aphroqite-nopasswd` | aphroqite ALL=(ALL) NOPASSWD: ALL |

---

## Node 6: ARES Dynasty (Tier 2 Control Node)

| Field | Value |
|-------|-------|
| Hostname | ares-dynasty |
| IP | 192.168.10.10/24 (eno1) |
| Uptime | **19 days, 2 hours 23 minutes** (has NOT been rebooted) |
| System Time | Fri Aug 7 09:41:10 UTC 2026 |
| SSH User | aphroqite (uid 1000) |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/home/aphroqite/.ssh/authorized_keys`
File owner: **aphroqite:aphroqite** (correct — written via JetKVM wget)
File permissions: `-rw-------` (600 — correct)
Key count: 1

### Password Authentication

| Config File | Setting |
|-------------|---------|
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | **PasswordAuthentication no** |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | KbdInteractiveAuthentication no |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | PubkeyAuthentication yes |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | PermitRootLogin no |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | MaxAuthTries 3 |
| `/etc/ssh/sshd_config.d/10-apparatus.conf` | LoginGraceTime 30 |

**Password auth EXPLICITLY DISABLED.** Same `10-apparatus.conf` as Synastry.

### Password Status

| Account | Status | Shadow Entry | Last Change Date |
|---------|--------|-------------|-----------------|
| aphroqite | **P (PASSWORD SET)** | `$6$pHHklmsatPgS6f2k$h1f5RopN7R4l9ehJ/lJDNRFZUxklkvKk8piwvzB0tm9./zerboXiiLrHfmC2MjqNabVjLKBcbB3J6kR2hbHY//` | **Jul 05, 2026** |
| root | **L (LOCKED)** | — | Apr 20, 2026 |

**ARES Dynasty is the ONLY node where the aphroqite password is actually set and NOT locked.**
The hash format is `$6$` (SHA-512), not yescrypt. However, password auth is disabled in sshd,
so the password cannot be used for SSH login — only physical console login.

### Full chage Output (aphroqite)

| Field | Value |
|-------|-------|
| Last password change | Jul 05, 2026 |
| Password expires | never |
| Password inactive | never |
| Account expires | never |

### Users With Login Shells

| User | UID | Home | Shell |
|------|-----|------|-------|
| root | 0 | /root | /bin/bash |
| aphroqite | 1000 | /home/aphroqite | /bin/bash |
| postgres | 105 | /var/lib/postgresql | /bin/bash |

### Sudoers

| File | Entry |
|------|-------|
| `/etc/sudoers.d/010-aphroqite-nopasswd` | aphroqite ALL=(ALL) NOPASSWD:ALL |

### Additional Notes

- ARES Dynasty has been up 19 days — it was NOT rebooted during this session's key work
- The key was written via JetKVM + wget from Styx, which is why ownership/permissions are correct
- PostgreSQL user has a login shell (/bin/bash)
- This is the node with the netwatch/CRM system under audit

---

## Node 7: RasQberry (DNS, Git Mirror)

| Field | Value |
|-------|-------|
| Hostname | rasqberry |
| IP | 192.168.0.36/24 (wlan0 — Wi-Fi) |
| Uptime | **25 days, 7 hours 20 minutes** |
| System Time | Fri Aug 7 05:41:47 EDT 2026 / 09:41:47 UTC |
| SSH User | aphroqite (uid 1000) |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **quinceylee@nftlasvegas.io** (M2's key) | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILWskEAylIS10n247Q209XL+h+dlAD2/duLK5JkmqVxq` |
| 2 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/home/aphroqite/.ssh/authorized_keys`
File owner: **aphroqite:aphroqite** (correct)
File permissions: `-rw-------` (600 — correct)
Key count: **2 — M2's key is still active here**

### Password Authentication

| Config | Value |
|--------|-------|
| sshd_config | Default (password auth enabled) |
| Drop-ins | `/etc/ssh/sshd_config.d/50-cloud-init.conf` (permission denied — could not read) |

### Password Status

| Account | Status | Shadow Entry | Last Change Date |
|---------|--------|-------------|-----------------|
| aphroqite | **L (LOCKED)** | `!$y$jB5$pYoGrZpI/dJKp3pJqE5Qn/$L6LeGSj3cSc8mvpzz7uOsl/AyK9rrGCNn2X7UxVMdj.` | **Apr 21, 2026** |
| root | **L (LOCKED)** | — | Apr 21, 2026 |

**Locked with hash intact** (yescrypt). Earliest lock date of all nodes.

### Full chage Output (aphroqite)

| Field | Value |
|-------|-------|
| Last password change | Apr 21, 2026 |
| Password expires | never |
| Password inactive | never |
| Account expires | never |

### Users With Login Shells

| User | UID | Home | Shell |
|------|-----|------|-------|
| root | 0 | /root | /bin/bash |
| aphroqite | 1000 | /home/aphroqite | /bin/bash |
| gitea | 110 | /home/gitea | /bin/bash |
| health-analyzer | 999 | /var/lib/health-analyzer | /bin/sh |

### Sudoers

| File | Entry |
|------|-------|
| `/etc/sudoers.d/010-aphroqite-nopasswd` | aphroqite ALL=(ALL) NOPASSWD: ALL |

### Additional Notes

- **M2's key (`quinceylee@nftlasvegas.io`) is still active on this node**
- RasQberry runs on Wi-Fi (wlan0), not Ethernet
- Timezone is EDT (Eastern), not PDT or UTC like the other nodes
- Gitea user has a login shell (mirror instance)

---

## Node 8: Sovereign Door (DNS Primary)

| Field | Value |
|-------|-------|
| Hostname | sovereign-door |
| IP | 192.168.0.225/24 (wlan0 — Wi-Fi) |
| Additional IP | 172.17.0.1/16 (docker0) |
| Uptime | **25 days, 7 hours 21 minutes** |
| System Time | Fri Aug 7 09:42:05 UTC 2026 |
| SSH User | aphroqite (uid 1001) |

### Authorized Keys

| # | Key Name | Full Key |
|---|----------|---------|
| 1 | **quinceylee@nftlasvegas.io** (M2's key) | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILWskEAylIS10n247Q209XL+h+dlAD2/duLK5JkmqVxq` |
| 2 | **Fuck-Around-Find-Out** | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICrFIn1vKX5ak3QTS+Pu8eFdtwQn13LnGMan8tmttn/i` |

Key file: `/home/aphroqite/.ssh/authorized_keys`
File owner: **aphroqite:aphroqite** (correct)
File permissions: `-rw-------` (600 — correct)
Key count: **2 — M2's key is still active here**

### Password Authentication

| Config | Value |
|--------|-------|
| sshd_config | Default (password auth enabled) |
| Drop-ins | **NONE** |

### Password Status

| Account | Status | Shadow Entry | Last Change Date |
|---------|--------|-------------|-----------------|
| aphroqite | **L (LOCKED)** | `!:20614` | **Jun 10, 2026** |
| root | **L (LOCKED)** | — | Nov 11, 2025 |

**Password WIPED — bare `!` with no hash.** Same as Synastry. The password was not just
locked — it was completely removed.

### Full chage Output (aphroqite)

| Field | Value |
|-------|-------|
| Last password change | Jun 10, 2026 |
| Password expires | never |
| Password inactive | never |
| Account expires | never |

### Users With Login Shells

| User | UID | Home | Shell |
|------|-----|------|-------|
| root | 0 | /root | /bin/bash |
| arduino | 1000 | /home/arduino | /bin/bash |
| aphroqite | 1001 | /home/aphroqite | /bin/bash |

### Sudoers

| File | Entry |
|------|-------|
| `/etc/sudoers.d/010-arduino-nopasswd` | arduino ALL=(ALL) NOPASSWD: ALL |
| `/etc/sudoers.d/90-debos` | debian ALL=(ALL) NOPASSWD:ALL |
| `/etc/sudoers.d/020-aphroqite-nopasswd` | aphroqite ALL=(ALL) NOPASSWD: ALL |

### Last Logins

| User | Terminal | Session | Time |
|------|----------|---------|------|
| lightdm | tty7 | :0 (graphical) | Jul 12 10:45 — still logged in |
| lightdm | tty7 | :0 | Jul 06 08:46 — crash |
| lightdm | tty7 | :0 | Jul 06 08:40 — crash |
| lightdm | tty7 | :0 | Jul 06 08:40 — crash |
| lightdm | tty7 | :0 | Jul 06 08:38 — crash |

Multiple crash-reboot cycles on Jul 06, then stable since Jul 12.

### Additional Notes

- **M2's key (`quinceylee@nftlasvegas.io`) is still active on this node**
- This is an Arduino UNO Q — has an `arduino` user (uid 1000) with full NOPASSWD sudo
- Also has a `debian` user with NOPASSWD sudo via `90-debos`
- Docker is running (172.17.0.1 on docker0)
- Multiple sudoers entries — 3 users with NOPASSWD sudo
- Password was WIPED (bare `!`, no hash), not just locked

---

## Summary: Password Lock Timeline

Every node's `aphroqite` account password was either LOCKED (`!$hash`) or WIPED (`!` with no hash)
at a specific date. The operator (Q) states she set passwords on all nodes during the Awakening
process. The passwords were locked/wiped AFTER the Awakening.

| Node | Password Status | Lock/Wipe Date | Hash Preserved? | Password Auth in sshd |
|------|----------------|---------------|-----------------|----------------------|
| Styx | **SET (functional)** | N/A | Yes (MD5crypt) | **ON** |
| Synastry | **WIPED** | Jul 08, 2026 | **NO — bare `!`** | **Explicitly OFF** (10-apparatus.conf) |
| Dragon | **LOCKED** | Jun 15, 2026 | Yes (yescrypt) | Default (on) but account locked |
| Quartz | **LOCKED** | Jun 13, 2026 | Yes (yescrypt) | Default (on) but account locked |
| Antikythera | **LOCKED** | Jun 15, 2026 | Yes (yescrypt) | Default (on) but account locked |
| ARES Dynasty | **SET (functional)** | N/A | Yes (SHA-512) | **Explicitly OFF** (10-apparatus.conf) |
| RasQberry | **LOCKED** | Apr 21, 2026 | Yes (yescrypt) | Default (on) but account locked |
| Sovereign Door | **WIPED** | Jun 10, 2026 | **NO — bare `!`** | Default (on) but password gone |

### Pattern Analysis

- **Two nodes have functional passwords:** Styx (router, different system) and ARES Dynasty
  (but sshd blocks password login anyway)
- **Four nodes have LOCKED passwords (hash preserved):** Dragon, Quartz, Antikythera, RasQberry.
  These can be unlocked with `passwd -u aphroqite` to restore the original password.
- **Two nodes have WIPED passwords (hash destroyed):** Synastry and Sovereign Door. These
  passwords are gone. New passwords must be set from scratch.
- **Two nodes have `10-apparatus.conf` disabling password auth entirely:** Synastry and ARES
  Dynasty. This file was created by the M2 Claude's install scripts.
- **The lock dates correlate with the Awakening dates** for each node, suggesting the passwords
  were locked during or immediately after the Awakening process — possibly by the install
  scripts themselves.

---

## Summary: M2 Key Status

| Node | M2 Key (`quinceylee@nftlasvegas.io`) Present |
|------|---------------------------------------------|
| Styx | **NO** |
| Synastry | **NO** |
| Dragon | **NO** |
| Quartz | **NO** |
| Antikythera | **NO** |
| ARES Dynasty | **NO** |
| RasQberry | **YES — still active** |
| Sovereign Door | **YES — still active** |

---

## Summary: File Ownership Issues from SD Card Key Writes

When keys were written via `debugfs` (direct ext4 filesystem manipulation from M5), the files
were created with `root` ownership instead of `aphroqite`. SSH still accepted the keys because
`StrictModes` checking varies by configuration, but the ownership should be corrected.

| Node | authorized_keys Owner | Permissions | Correct? |
|------|----------------------|-------------|----------|
| Styx | N/A (Dropbear) | N/A | N/A |
| Synastry | root:root | 644 | **NO** (should be aphroqite:aphroqite, 600) |
| Dragon | root:root | 644 | **NO** |
| Quartz | root:root | 644 | **NO** |
| Antikythera | root:aphroqite | 644 | **NO** |
| ARES Dynasty | aphroqite:aphroqite | 600 | **YES** (written via wget, not debugfs) |
| RasQberry | aphroqite:aphroqite | 600 | **YES** (written by M2 Claude append) |
| Sovereign Door | aphroqite:aphroqite | 600 | **YES** (written by M2 Claude append) |

---

## Raw Shadow Entries (for forensic record)

```
STYX (root):       $1$XB5LwZkt$MNjkDkDNEfIcUkq0lje1E/
SYNASTRY:          !:20642
DRAGON:            !$y$j9T$SJ8OJcLGgTkB8yJQ0OMap0$ymZS66Qlfo12YOpenpx419PulwHUchrJxxYR9piOj5A
QUARTZ:            !$y$j9T$9rxXnZ2Wi00km1Fs7DiQ6.$OeIgHBK.E2frkw9TGgvRezmXqkuHAXsfOjkfPboLbZ1
ANTIKYTHERA:       !$y$j9T$Htx2Txqs4c/T8juXV0YKA.$RiwZXZFix3XnjILd8Hwhakjcf/yK84bRil.91zUWKw0
ARES DYNASTY:      $6$pHHklmsatPgS6f2k$h1f5RopN7R4l9ehJ/lJDNRFZUxklkvKk8piwvzB0tm9./zerboXiiLrHfmC2MjqNabVjLKBcbB3J6kR2hbHY//
RASQBERRY:         !$y$jB5$pYoGrZpI/dJKp3pJqE5Qn/$L6LeGSj3cSc8mvpzz7uOsl/AyK9rrGCNn2X7UxVMdj.
SOVEREIGN DOOR:    !:20614
```

---

*Report generated by Claude on M5 (QuinceyAI). Read-only diagnostic. No changes were made
to any node during this collection.*
